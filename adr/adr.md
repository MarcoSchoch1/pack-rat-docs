# Pack-Rat — Architecture Decision Records

A log of the significant technical decisions made on this project, the context behind them, and the alternatives considered. Kept for future reference — and to make the reasoning behind the project easy to walk through, not just the outcome.

---

## ADR-001: Use PostgreSQL via Docker instead of H2

**Status:** Accepted

**Context:** Needed a database for local development. H2 is the default pairing with Spring Initializr and requires zero setup, but is primarily meant for testing rather than an app with real, persistent data.

**Decision:** Use PostgreSQL, run locally via Docker Compose, from day one — rather than starting on H2 and migrating later.

**Alternatives considered:**
- **H2 (in-memory):** Fastest to start with, but data doesn't survive a restart unless configured for file-based storage.
- **H2 (file-based):** Persists to disk, no Docker required, but still meant for prototyping — SQL dialect quirks can hide bugs that surface later on a real database.
- **SQLite:** Simple, single-file, no server process — a reasonable alternative, but less representative of a production deployment.

**Consequences:** A small amount of upfront setup (Docker Compose file, container management) in exchange for skipping a schema-migration step later and having a stack that matches how the app would actually be deployed.

---

## ADR-002: Use UUIDs instead of auto-incrementing integers for primary keys

**Status:** Accepted

**Context:** Default Spring Data JPA behavior uses auto-incrementing integer/long IDs. These are simple, but sequential and predictable.

**Decision:** Use UUIDs (Postgres `UUID` column type, Java `java.util.UUID`) for all primary keys.

**Alternatives considered:**
- **Auto-incrementing integers:** Simpler, marginally better index performance, but exposes information (e.g. total record count) and invites ID enumeration if the API is ever exposed beyond a trusted network.

**Consequences:** Slightly larger index size and less human-readable IDs in logs/debugging, in exchange for IDs that are safe to expose externally and don't leak record counts — a reasonable default for anything that might eventually be reachable outside localhost.

---

## ADR-003: Keep User → Collection as one-to-many, even though the MVP only uses one collection per user

**Status:** Accepted

**Context:** The MVP scope only requires a single collection per user. A strict one-to-one relationship would be simpler to reason about today.

**Decision:** Model the relationship as one-to-many (a user can own multiple collections) from the start, even though the UI only creates and displays one for now.

**Alternatives considered:**
- **One-to-one:** Simpler schema for the current scope, but would require a migration (and likely data backfill) if multiple collections per user — e.g. separate collections per TCG — became a real feature later.

**Consequences:** Zero added complexity now (a one-to-many FK costs nothing extra to build), and the schema doesn't block a natural future feature.

---

## ADR-004: Model Image as its own entity rather than fields on Item

**Status:** Accepted

**Context:** The MVP requires "a picture" per item. An uploaded image also carries metadata (original filename, content type, file size) beyond just a URL — and there was a real design choice between storing that as flat fields on `Item`, or as a related entity.

**Decision:** Create a separate `Image` entity, related to `Item` as one-to-many, even though the MVP only ever uploads one image per item.

**Alternatives considered:**
- **Flat fields on Item** (`imageUrl`, `originalFilename`, `contentType`, `fileSizeBytes`): Simpler, no extra join, matches the current "one picture" rule exactly — but would require a schema migration to support multiple photos per item later (e.g. front/back of a card).

**Consequences:** One extra table and join for what is currently a 1:1 relationship, in exchange for room to grow without a migration.

---

## ADR-005: Fixed enum for item condition instead of free text

**Status:** Accepted

**Context:** Condition needed to be captured per item. Free text is fastest to implement but produces inconsistent data ("NM" vs "Near Mint" vs "near mint").

**Decision:** Use a fixed set of values matching standard TCG marketplace grading: Mint, Near Mint, Excellent, Good, Light Played, Played, Poor.

**Alternatives considered:**
- **Free text:** No validation needed, but breaks any future filtering, sorting, or reporting by condition.

**Consequences:** Slightly more setup (backend enum, frontend dropdown) for meaningfully cleaner, queryable data.

---

## ADR-006: Store currency as a field per item rather than assuming a single fixed currency

**Status:** Accepted

**Context:** Price paid needed a currency. A single implicit currency (e.g. always CHF) is simpler; marketplace links (TCGPlayer in USD, Cardmarket in EUR) suggested items could realistically be priced in different currencies.

**Decision:** Add a `currency` field (ISO code, e.g. CHF/USD/EUR) to each item rather than assuming one fixed currency app-wide.

**Consequences:** Slightly more fields in the data model and forms, in exchange for accurate cost data regardless of where an item was purchased.

---

## ADR-007: JWT for authentication instead of server-side sessions

**Status:** Accepted

**Context:** MVP uses a dev/hardcoded login, but needs to be structured so real accounts (for friends) can be added later without a rework.

**Decision:** Use stateless JWTs, issued on login and sent as a Bearer token on subsequent requests.

**Alternatives considered:**
- **Session cookies:** Simpler in some respects (server manages state), but requires server-side session storage and doesn't scale as cleanly if the backend and frontend are ever deployed separately.

**Consequences:** No server-side session state to manage; the same auth mechanism will carry over unchanged when real multi-user login replaces the dev login — only credential validation changes.

---

## ADR-008: Image upload as its own endpoint, decoupled from item creation

**Status:** Accepted

**Context:** Items are created via a JSON POST request. Images are uploaded as multipart form data. Combining both into a single request is possible but awkward to implement cleanly in Spring Boot, and conflates two different concerns.

**Decision:** Create an item first via `POST /api/collections/{collectionId}/items` (JSON), then upload its image(s) separately via `POST /api/items/{itemId}/images` (multipart).

**Alternatives considered:**
- **Single combined multipart request:** Fewer round-trips, but mixes JSON fields and binary file data in one request, complicating both the request shape and error handling (e.g. what happens if the item fields are valid but the image upload fails?).

**Consequences:** Two requests instead of one for the full "add item with photo" flow, in exchange for cleaner separation of concerns and simpler validation per step.

---

## ADR-009: Split backend and frontend into separate repositories

**Status:** Accepted

**Context:** Spring Boot backend and Angular frontend are independently deployable units.

**Decision:** Use separate GitHub repositories (`pack-rat-backend`, `pack-rat-frontend`), plus a lightweight `pack-rat-docs` repository for cross-cutting documentation and general project issues that don't belong to either codebase.

**Alternatives considered:**
- **Monorepo:** Single repo, simpler issue tracking (no need to combine boards across repos), but couples the deploy lifecycle of two genuinely independent applications.

**Consequences:** Some added friction around cross-repo project tracking (GitHub's auto-add workflows are capped per-repo on the free tier, requiring a custom GitHub Action to work around it — see repo automation setup), in exchange for a repo structure that matches how the app is actually built and deployed.

---

## ADR-010: Separate containers per service; DB-only Docker during local development

**Status:** Accepted

**Context:** The app will eventually be hosted (backend, frontend, database) rather than only run locally. Needed to decide both the container structure for deployment, and how that structure should (or shouldn't) affect the day-to-day development workflow.

**Decision:**
- Use one container per service — `postgres`, `backend` (Spring Boot JAR in a slim JRE image), `frontend` (Angular build served via Nginx) — orchestrated with a single `docker-compose.yml`, rather than bundling backend and database into one container.
- During active development, run only `postgres` in Docker; run the backend from the IDE and the frontend via `ng serve` for fast hot-reload iteration. Periodically run the full `docker compose up --build` to confirm the fully containerized version still works before considering a feature done.

**Alternatives considered:**
- **Backend + DB in one container:** Simpler on the surface, but couples two independently-scaled concerns and risks losing persistent data when the backend container is rebuilt (which happens constantly during development).
- **Everything containerized even during development:** Exactly matches production at all times, but a rebuild-and-restart cycle on every code change makes day-to-day iteration noticeably slower.

**Consequences:** Fast local iteration without sacrificing confidence that the app actually works in its real, fully-containerized deployment shape — verified periodically rather than continuously. Hosting target (VPS vs. home server) deferred as a separate decision.
 
---
 
## ADR-011: Use Tailwind CSS instead of a component library (e.g. Angular Material)
 
**Status:** Accepted
 
**Context:** The Angular frontend needs a styling approach. Angular Material offers pre-built, accessible components (buttons, forms, dropdowns) with minimal styling effort. Tailwind CSS offers utility classes instead of components, requiring more manual styling work but no framework-specific component API to learn.
 
**Decision:** Use Tailwind CSS for styling.
 
**Alternatives considered:**
- **Angular Material:** Faster to build a consistent, accessible UI with less manual styling, but results in a recognizably "Material Design" look and ties UI knowledge specifically to Angular's ecosystem.
- **Plain CSS/SCSS:** Full control with no dependency, but no utility-class speed benefit and more time spent naming classes and managing stylesheets.
**Consequences:** More manual work to build common UI patterns (dropdowns, modals) that Material would provide out of the box, in exchange for a widely transferable frontend skill (Tailwind is framework-agnostic and broadly in-demand) and full control over the app's visual identity rather than a stock component look.
 