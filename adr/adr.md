# Pack-Rat — Architecture Decision Records

A log of the significant technical decisions made on this project, the context behind them, and the alternatives considered. Kept for future reference — and to make the reasoning behind the project easy to walk through, not just the outcome.

Entries are numbered chronologically, in the order decisions were made, and are never renumbered — this preserves the actual decision history, especially once later decisions revise or build on earlier ones. Use the index below to jump to a topic instead.

## Index by topic

**Database & data model**
[ADR-001](#adr-001-use-postgresql-via-docker-instead-of-h2) (Postgres/Docker) · [ADR-002](#adr-002-use-uuids-instead-of-auto-incrementing-integers-for-primary-keys) (UUID keys) · [ADR-003](#adr-003-keep-user--collection-as-one-to-many-even-though-the-mvp-only-uses-one-collection-per-user) (User→Collection) · [ADR-004](#adr-004-model-image-as-its-own-entity-rather-than-fields-on-item) (Image entity) · [ADR-005](#adr-005-fixed-enum-for-item-condition-instead-of-free-text) (condition enum) · [ADR-006](#adr-006-store-currency-as-a-field-per-item-rather-than-assuming-a-single-fixed-currency) (currency field) · [ADR-020](#adr-020-mark-self-pulled-cards-with-a-selfpulled-boolean-price-paid-forced-to-0) (self-pulled flag) · [ADR-022](#adr-022-multiple-collections--totals-per-collection-in-the-list-response-overview-dashboard-on-the-client) (multiple collections)

**API design**
[ADR-007](#adr-007-jwt-for-authentication-instead-of-server-side-sessions) (JWT auth) · [ADR-008](#adr-008-image-upload-as-its-own-endpoint-decoupled-from-item-creation) (image upload endpoint) · [ADR-016](#adr-016-explicit-cors-configuration-via-spring-security-not-a-reverse-proxy-workaround) (CORS) · [ADR-019](#adr-019-send-the-jwt-in-an-httponly-cookie-instead-of-storing-it-in-localstorage) (token in httpOnly cookie) · [ADR-021](#adr-021-self-sign-up-through-single-use-invite-links) (self sign-up) · [ADR-022](#adr-022-multiple-collections--totals-per-collection-in-the-list-response-overview-dashboard-on-the-client) (multiple collections)

**Images (cross-cutting: data model + API + constraints)**
[ADR-004](#adr-004-model-image-as-its-own-entity-rather-than-fields-on-item) (entity) · [ADR-008](#adr-008-image-upload-as-its-own-endpoint-decoupled-from-item-creation) (endpoint) · [ADR-015](#adr-015-image-upload-limits--max-2mb-upload-resized-to-500px-longest-edge) (size limits) · [ADR-018](#adr-018-store-images-as-bytea-in-postgres-served-publicly-by-unguessable-uuid) (storage & serving)

**Repos & infrastructure**
[ADR-001](#adr-001-use-postgresql-via-docker-instead-of-h2) (Postgres/Docker) · [ADR-009](#adr-009-split-backend-and-frontend-into-separate-repositories) (repo split) · [ADR-010](#adr-010-separate-containers-per-service-db-only-docker-during-local-development) (container-per-service) · [ADR-012](#adr-012-docker-compose-for-local-postgres-lives-in-the-backend-repo-a-separate-deploy-repo-will-handle-full-multi-service-orchestration) (compose file placement, deploy repo)

**Hosting & deployment**
[ADR-023](#adr-023-host-on-an-old-computer-at-home-running-ubuntu-server-and-docker-engine) (home server) · [ADR-024](#adr-024-reach-the-app-only-over-tailscale-with-https-from-tailscale-serve) (Tailscale only) · [ADR-025](#adr-025-one-origin-in-deployment-the-frontends-nginx-serves-the-app-and-proxies-api-to-the-backend) (single origin) · [ADR-026](#adr-026-build-images-with-github-actions-publish-them-to-ghcr-and-run-them-from-pack-rat-deploy) (GHCR images, deploy repo) · [ADR-027](#adr-027-create-the-first-account-from-environment-variables-when-the-database-is-empty) (first account) · [ADR-028](#adr-028-nightly-pg_dump-backups-stored-on-a-different-disk) (backups)

**Frontend**
[ADR-011](#adr-011-use-tailwind-css-instead-of-a-component-library-eg-angular-material) (Tailwind CSS)

**Backend setup & operations**
[ADR-013](#adr-013-backend-project-setup--maven-java-21-lts-package-by-layer) (Maven/Java 21/package structure) · [ADR-014](#adr-014-secrets-management-via-gitignored-local-config-environment-variables-in-deployment) (secrets management)

**Pricing**
[ADR-006](#adr-006-store-currency-as-a-field-per-item-rather-than-assuming-a-single-fixed-currency) (currency field) · [ADR-017](#adr-017-current-price-is-a-manual-optional-field-for-now--no-automated-price-lookup-yet) (manual current price) · [ADR-020](#adr-020-mark-self-pulled-cards-with-a-selfpulled-boolean-price-paid-forced-to-0) (self-pulled flag)

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

**Related:** ADR-010 (container-per-service), ADR-012 (compose file placement)

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

**Related:** ADR-008 (upload endpoint), ADR-015 (size limits)

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

**Related:** ADR-004 (Image entity), ADR-015 (size limits)

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

**Related:** ADR-001 (Postgres/Docker), ADR-012 (compose file placement)

---

## ADR-011: Use Tailwind CSS instead of a component library (e.g. Angular Material)

**Status:** Accepted

**Context:** The Angular frontend needs a styling approach. Angular Material offers pre-built, accessible components (buttons, forms, dropdowns) with minimal styling effort. Tailwind CSS offers utility classes instead of components, requiring more manual styling work but no framework-specific component API to learn.

**Decision:** Use Tailwind CSS for styling.

**Alternatives considered:**
- **Angular Material:** Faster to build a consistent, accessible UI with less manual styling, but results in a recognizably "Material Design" look and ties UI knowledge specifically to Angular's ecosystem.
- **Plain CSS/SCSS:** Full control with no dependency, but no utility-class speed benefit and more time spent naming classes and managing stylesheets.

**Consequences:** More manual work to build common UI patterns (dropdowns, modals) that Material would provide out of the box, in exchange for a widely transferable frontend skill (Tailwind is framework-agnostic and broadly in-demand) and full control over the app's visual identity rather than a stock component look.

---

## ADR-012: Docker Compose for local Postgres lives in the backend repo; a separate deploy repo will handle full multi-service orchestration

**Status:** Accepted

**Context:** Per ADR-010, only the backend touches Postgres directly during local development (`docker compose up postgres`, backend and frontend run natively). Needed to decide where that Postgres `docker-compose.yml` should live, and separately, how the full multi-service deployment (Postgres + backend + frontend together, per ADR-010's three-container plan) will eventually be orchestrated.

**Decision:**
- Keep a single-service `docker-compose.yml` (Postgres only) inside `pack-rat-backend`, since it's a dependency of the backend during local dev and nothing else needs it at that stage. Anyone cloning the backend repo gets a working local setup with no cross-repo lookup.
- Create a separate `pack-rat-deploy` repository later to hold the full deployment compose file (all three services together) and CI/CD orchestration, since a compose file inside a single-service repo can't cleanly reference container images built from the other repos.

**Alternatives considered:**
- **One shared compose file across all repos from the start:** Would need to live somewhere not tied to any single service, adding structure before it's actually needed. Premature for the current stage where only Postgres is containerized during dev.
- **Duplicate the full compose file into each repo:** Avoids a new repo, but risks the files drifting out of sync as services change.

**Consequences:** Local dev setup stays simple and self-contained per repo now; deployment/CI-CD orchestration gets a clear, dedicated home once that stage of the project starts, without needing to relocate files out of `pack-rat-backend` later.

**Related:** ADR-001 (Postgres/Docker), ADR-010 (container-per-service)

---

## ADR-013: Backend project setup — Maven, Java 21 (LTS), package-by-layer

**Status:** Accepted

**Context:** Needed to pick the Spring Initializr configuration for `pack-rat-backend` before generating the project, to avoid regenerating it later.

**Decision:**
- Build tool: Maven
- Java version: 21 (LTS)
- Package structure: by layer (`controller/`, `service/`, `repository/`, `model/`)

**Alternatives considered:**
- **Gradle:** Faster builds, more concise config, but Maven's XML convention is more common in the enterprise Java shops matching the target job roles for this project.
- **Latest Java version (25):** More current, but LTS releases are what most companies actually run in production, to avoid frequent upgrade cycles — Java 21 is the more defensible, production-credible choice.
- **Package-by-feature:** Scales better for larger codebases, but package-by-layer is simpler to navigate for a project this size and more familiar as a starting convention.

**Consequences:** A conventional, enterprise-recognizable project setup — easy to explain and defend in an interview context, even if not the most "modern" choice on every axis.

---

## ADR-014: Secrets management via gitignored local config, environment variables in deployment

**Status:** Accepted

**Context:** DB credentials and the JWT signing secret must not be hardcoded or committed to the repository.

**Decision:**
- Local dev: secrets live in a gitignored `.env` (or `application-local.properties`) file; a committed `.env.example` / `application-local.properties.example` with placeholder values shows what's needed.
- Deployment: actual secrets are passed as environment variables into the container at runtime, handled in `pack-rat-deploy` (see ADR-012) — never baked into an image or committed anywhere.

**Consequences:** Standard, low-risk secrets handling with no special tooling required; anyone cloning the repo can see what configuration they need without exposing real values.

---

## ADR-015: Image upload limits — max 2MB upload, resized to 500px longest edge

**Status:** Accepted

**Context:** The MVP scope required images to be "small size," but no concrete numbers had been set, which blocks writing the actual resize/validation logic.

**Decision:** Accept uploads up to 2MB; resize/store images at a maximum of 500px on the longest edge.

**Consequences:** Keeps storage and page-load size small, appropriate for the personal/friends-scale use case; concrete numbers now exist for both backend validation (reject uploads over 2MB) and the resize step (`Image` entity's `fileSizeBytes` will reflect the post-resize size).

**Related:** ADR-004 (Image entity), ADR-008 (upload endpoint)

---

## ADR-016: Explicit CORS configuration via Spring Security, not a reverse-proxy workaround

**Status:** Accepted (still used in dev; in deployment, ADR-025 serves both under one origin)

**Context:** Frontend (Angular) and backend (Spring Boot) are separate origins both locally (`localhost:4200` vs `localhost:8080`) and in deployment (separate services per ADR-009/ADR-010) — browsers block cross-origin requests by default, so something has to explicitly allow them.

**Decision:** Configure CORS explicitly in Spring Security — a `CorsConfigurationSource` bean wired into the same filter chain as the JWT filter (ADR-007) — with allowed origins read from a config property (`app.cors.allowed-origins`) rather than hardcoded: `http://localhost:4200` for local dev, the real frontend domain once deployed.

**Alternatives considered:**
- **Reverse proxy (Nginx, or the Angular dev-server proxy) hiding the cross-origin call entirely:** Removes the problem with less code and no security-config surface, but couples the frontend and backend to always being deployed together behind the same proxy — less flexible if that ever changes, and moves the concern out of the backend entirely.

**Consequences:** A small, explainable piece of Spring Security config (one bean, one property) instead of none at all; origins are environment-driven, consistent with ADR-014's approach to environment-specific settings.

**Related:** ADR-007 (JWT auth, same filter chain), ADR-009/ADR-010 (separate frontend/backend services — why this is needed at all), ADR-014 (env-driven config pattern)

---

## ADR-017: Current price is a manual, optional field for now — no automated price-lookup yet

**Status:** Accepted

**Context:** The dashboard needs a "total price now" figure alongside "total price paid." Automatically fetching current value from marketplaces (TCGPlayer/Cardmarket) is a real feature planned for later, but per-marketplace scraping/API integration is real scope, not needed to unblock the MVP — `marketplaceLink` already lets a user look the price up manually.

**Decision:** Add `priceNow` as an optional, user-editable field on `Item` (same shape as `pricePaid`: decimal, same currency), filled in manually for now. `Collection.totalPriceNow` sums whatever items have a value set, and is `null` if none do. `marketplaceLink` stays the manual path to find the number; nothing fetches it automatically yet.

**Alternatives considered:**
- **Leave `totalPriceNow` unimplemented until automated lookup exists:** Avoids a manually-filled field today, but leaves the dashboard's second metric permanently blank and defers work (field + aggregation) that's needed either way.
- **Build the automated marketplace lookup now:** Solves it properly, but per-marketplace integration is scope creep for the MVP.

**Consequences:** Dashboard's "total price now" is real and functional immediately, user-maintained. The field and aggregation are already in place for when automated lookup arrives later — that becomes a new write path filling the same field, not a schema change.

**Related:** ADR-006 (currency field), ADR-004 (marketplaceLink stays the manual fallback)

---

## ADR-018: Store images as bytea in Postgres, served publicly by unguessable UUID

**Status:** Accepted

**Context:** Uploaded images (ADR-008) need to be stored somewhere. The app should stay fully self-hosted with no dependency on a cloud provider, and images are small: at most 500px on the longest edge after resizing, so around 50KB each (ADR-015). There is also a frontend constraint. A plain `<img src>` cannot send the JWT `Authorization` header (ADR-007), so serving images behind the normal auth would force the frontend to fetch each one as a blob.

**Decision:**
- Store the resized image bytes directly in Postgres as a `bytea` column on the `Image` table. `Image.url` is no longer stored. The image is served at `/api/images/{id}`.
- Map the column lazily (`@Basic(fetch = FetchType.LAZY)`) so loading items or image metadata does not pull the bytes.
- Deleting an item cascades to its images (`ON DELETE CASCADE`).
- `GET /api/images/{id}` is `permitAll` and returns the bytes with the stored `contentType` and a long `Cache-Control` max-age. The UUID id (ADR-002) is effectively unguessable, which is the same model as presigned or CDN URLs. Upload and delete stay JWT-protected.
- Frontend: upload with `FormData` (multipart) and display with a plain `<img [src]="'/api/images/' + id" loading="lazy">`.

**Alternatives considered:**
- **Filesystem on a Docker volume:** Keeps the DB small and is common practice, but adds a second thing to back up, can leave orphan files when deletes fail halfway, and needs care around file paths.
- **S3-compatible object storage in its own container (Garage, SeaweedFS, MinIO):** This is the industry-standard shape and offers an easy migration path to a cloud bucket, but it adds a container, credentials and an SDK. That is more moving parts than a friends-scale app needs.
- **JWT-protected image endpoint with blob fetching in Angular:** Keeps images behind auth, but adds frontend code for every image, and card photos are not sensitive enough to justify it.

**Consequences:** No new infrastructure. A single `pg_dump` backs up items and images together, and uploads and deletes are transactional with no orphan files. The costs: the DB grows (about 500MB per 10k images), every image request goes through the backend (offset by browser caching), and anyone with an image URL can view that image. If storage ever outgrows this, migrating means copying the bytes to files or object storage and rewriting only the image service. The frontend is unaffected because it only ever sees a URL.

**Related:** ADR-002 (UUID ids), ADR-004 (Image entity), ADR-007 (JWT auth), ADR-008 (upload endpoint), ADR-015 (size limits)

---

## ADR-019: Send the JWT in an httpOnly cookie instead of storing it in localStorage

**Status:** Accepted

**Context:** ADR-007 sends the JWT as a Bearer token, so the Angular frontend has to keep it somewhere JavaScript can read, which in practice means `localStorage`. That is the usual pairing, but any script injected through an XSS bug can read `localStorage` and send the token elsewhere. The token then works from any machine until it expires. Angular escapes template bindings by default, so XSS is unlikely, but one `bypassSecurityTrust*` call or one compromised npm dependency is enough.

**Decision:**
- Keep JWTs and stateless auth (ADR-007). Only the transport changes.
- On login, the backend sets the token as a cookie with `HttpOnly; Secure; SameSite=Lax; Path=/api`. It no longer returns the token in the response body. It returns only `{ "expiresAt": "…" }`, which the frontend keeps (it's not a secret) to know whether the user is still logged in, since it can't read the cookie. Any `401` sends the user back to login. `POST /api/auth/logout` clears the cookie.
- The JWT filter reads the token from the cookie instead of the `Authorization` header.
- CORS (ADR-016) sets `allowCredentials(true)`, which already requires explicit origins rather than `*`. The frontend sends requests with `withCredentials: true`.
- Deploy the frontend and backend on the same site so `SameSite=Lax` holds. In the home deployment they share one origin (ADR-025), served over HTTPS by Tailscale (ADR-024), which `Secure` needs. `localhost:4200` and `localhost:8080` already count as the same site, since ports are ignored.
- Because the browser now attaches credentials automatically, CSRF becomes a concern. `SameSite=Lax` blocks cross-site `POST`/`PUT`/`DELETE`, which covers it as long as no state-changing `GET` endpoints exist. Spring's `CookieCsrfTokenRepository` is the fallback if the deployment ever becomes cross-site.

**Alternatives considered:**
- **Keep the token in localStorage:** No backend change and no CSRF concern, but a single XSS bug leaks a usable token.
- **Keep the token only in memory (an Angular service):** XSS can't read it after the fact, but it is lost on every page reload, so the user logs in again or a refresh-token cookie is needed anyway, which brings back the cookie work.
- **Server-side sessions:** Rejected in ADR-007 and still not needed. An httpOnly cookie gets the XSS benefit without giving up statelessness.

**Consequences:** XSS can no longer steal the token. It can still make requests as the user while the page is open, so this limits the damage rather than removing it. The cost is a small backend change (set and clear the cookie, read it in the filter), CORS credentials, and a same-site deployment constraint. The frontend gets simpler: it stores no token, and the auth interceptor only sets `withCredentials`. As a side effect, `<img>` tags would now send the cookie, which removes the reason ADR-018 made the image endpoint public. That can stay as it is, but it no longer has to.

**Related:** ADR-007 (JWT auth, transport revised here), ADR-016 (CORS, now with credentials), ADR-018 (public image endpoint), ADR-024 (HTTPS), ADR-025 (single origin)

---

## ADR-020: Mark self-pulled cards with a `selfPulled` boolean, price paid forced to 0

**Status:** Accepted

**Context:** Not every card is bought. Many are pulled from packs the user opened themselves, so there is no meaningful price paid for that single card. `pricePaid` was required and had to be positive, so a pulled card could not be entered honestly.

**Decision:**
- Add `selfPulled` to `Item` (`boolean NOT NULL DEFAULT false`). Existing items count as bought, so no backfill is needed.
- Relax `pricePaid` validation from "positive" to "≥ 0".
- If `selfPulled` is true, the backend stores `pricePaid` as 0 regardless of what the client sends, so "pulled but paid 45 CHF" can't exist.
- The add item form gets a "Self-pulled" checkbox that sets price paid to 0 and disables the field.
- Totals need no change: a pulled card adds 0 to total price paid, and its value still counts through `priceNow` (ADR-017).

**Alternatives considered:**
- **Only allow `pricePaid = 0`, no flag:** No schema change, but 0 is ambiguous (pulled, gift, or just not entered) and pulled cards can't be filtered later.
- **`acquisitionType` enum (BOUGHT / PULLED / TRADED / GIFT):** Covers more cases, but trades and gifts aren't needed for the MVP. Can replace the boolean later as a feature (migration: `selfPulled = true` → `PULLED`, otherwise `BOUGHT`).

**Consequences:** One column, one validation change, one checkbox. The pack cost behind a pulled card isn't tracked anywhere, so total price paid understates what was actually spent on packs. Acceptable for the MVP.

**Related:** ADR-006 (currency), ADR-017 (manual current price)

---

## ADR-021: Self sign-up through single-use invite links

**Status:** Accepted

**Context:** Login already checks credentials against the `users` table with BCrypt (ADR-007). The only account is a `dev` user seeded under the `local` profile, so in production nobody can log in until a user is inserted by hand. Friends need to create their own accounts. Once deployed, the app is reachable from the internet, so a fully open sign-up lets anyone create accounts and fill the database with images (ADR-018).

**Decision:**
- Sign-up only works through an invite link, the same way a "confirm your email" link works: whoever has the link can register once, and nobody else can.
- New `Invite` entity: `id` (UUID, also the token in the link), `createdBy` (FK to User), `createdAt`, `expiresAt` (7 days after creation), `usedAt` and `usedBy` (FK to the new User, both null until the invite is used). Used invites are kept, not deleted, so the table records who invited whom and when. Like image URLs (ADR-018), the random UUID (ADR-002) is unguessable enough to act as the secret.
- `POST /api/invites` (logged in) creates an invite and returns `{ url, expiresAt }`, e.g. `https://<frontend>/register?invite=<uuid>`. Any logged-in user can invite. There are no roles, and the users are friends.
- `GET /api/invites/{token}` (public) returns `200` if the invite exists, is unused and has not expired, otherwise `404`. The register screen calls it first, so a dead link shows "This invite link is invalid or expired" instead of a form that can't succeed.
- `POST /api/auth/register` (public) with `{ inviteToken, username, password }` creates the user with a BCrypt hash (the existing `PasswordEncoder` bean) and claims the invite in the same transaction with `UPDATE invite SET used_at = now(), used_by = :user WHERE id = :token AND used_at IS NULL AND expires_at > now()`. If no row is updated, the transaction rolls back with `404`, so the link works once even when two people submit it at the same time. It returns the token exactly like `POST /api/auth/login`. ADR-019 changes how the token travels for both endpoints. A new user has no collection, so the existing first-login flow shows the create collection screen.
- Username: 3–32 characters, trimmed, compared case-insensitively. Add a unique constraint on `username`; a duplicate returns `409 Conflict`. Password: at least 8 characters, no composition rules (in line with NIST SP 800-63B).
- Frontend: an "Invite a friend" action creates an invite and copies the link. The login screen has no sign-up link. `/register` without a valid invite only shows the invalid-link message.

**Alternatives considered:**
- **Open sign-up:** Simplest, but anyone who finds the URL can create accounts and upload images.
- **One shared invite code from config:** One config value and no table, but a leaked code works for anyone until it is changed, and there's no way to tell who it was given to.
- **Admin creates accounts by hand (SQL or an admin endpoint):** No public endpoint, but every new friend needs manual work and the admin has to choose their password.
- **Email verification:** The real version of this flow, but needs mail sending and an email field. Sending the link over chat does the same job at this scale.

**Consequences:** One small table, three endpoints, one screen and one button. A leaked link creates at most one account and expires after 7 days. Any user can invite more users, so the user base grows by trust. If that ever becomes a problem, restrict `POST /api/invites` to an admin flag. Expired and used invites stay in the table; used ones are the sign-up audit trail. There's no password reset or rate limiting yet; both can be added later without changing this design.

**Related:** ADR-002 (UUID ids), ADR-007 (JWT auth), ADR-018 (unguessable-UUID URLs, image storage), ADR-019 (token transport)

---

## ADR-022: Multiple collections — totals per collection in the list response, overview dashboard on the client

**Status:** Accepted

**Context:** ADR-003 already allows a user to own several collections, but the UI only ever creates and shows one. Users want, for example, one collection per TCG, plus an overview that shows totals across all of them, just as the item dashboard does for one collection.

**Decision:**
- `GET /api/collections` returns each collection with its totals: `id`, `name`, `itemCount`, `totalPricePaid`, `totalPriceNow` (same rules as `/overview`, ADR-017). One aggregate query, one request.
- The overview dashboard sums these per-collection totals on the client and shows a tile per collection that links to its item dashboard.
- Routing after login: 0 collections → create collection screen; 1 → that collection's dashboard (unchanged); 2 or more → overview dashboard.
- The create collection screen is reused for a "New collection" action, reachable from the dashboard and the overview. Item dashboards get a link back to the overview once a second collection exists.

**Alternatives considered:**
- **Call `/api/collections/{id}/overview` per collection:** No backend change, but N+1 requests on every overview load.
- **Dedicated `GET /api/collections/overview` with server-side grand totals:** One more endpoint for a sum the client can do in one line from data it already loads.

**Consequences:** One query change and one new screen. Grand totals add up amounts without converting currencies, which is how per-collection totals already work (ADR-006). Mixed-currency totals are therefore approximate until currency conversion exists. Renaming and deleting collections already have endpoints (`PUT`/`DELETE /api/collections/{id}`) but no UI yet; that is out of scope here.

**Related:** ADR-003 (User → Collection one-to-many), ADR-006 (currency per item), ADR-017 (manual current price)

---

## ADR-023: Host on an old computer at home running Ubuntu Server and Docker Engine

**Status:** Accepted

**Context:** ADR-010 left the hosting target open (VPS or home server). The app is friends-scale, and an old computer is available at home. A VPS costs money every month and puts the app on the public internet by default, which ADR-021 only partly prepared for (no rate limiting, no password reset yet).

**Decision:**
- Self-host on the old computer. Wipe it and install Ubuntu Server 26.04 LTS (headless, no desktop) with hostname `packrat`.
- Install Docker Engine and the Compose plugin from Docker's apt repository, not Docker Desktop and not the snap.
- Every container uses `restart: unless-stopped` and Docker starts at boot, so the app comes back after a reboot or power cut without anyone logging in. In the BIOS, set "power on after AC loss". On a laptop, also make the OS ignore a closed lid.
- In the router, add a DHCP reservation so the computer's LAN IP never changes. No port forwarding.
- Administer it over SSH from the Mac, with key-only login (password login disabled). `ufw` allows SSH from the home subnet only, plus everything on `tailscale0` (ADR-024). Ubuntu's `unattended-upgrades` stays on for security patches.

**Alternatives considered:**
- **VPS (e.g. Hetzner):** Reliable hardware and a public IP, but a monthly cost, and the app sits on the internet by default.
- **Keep the existing OS and use Docker Desktop:** No reinstall, but Docker Desktop runs inside a VM, uses more RAM and only starts when a user logs in. That's the wrong fit for an always-on server.
- **Raspberry Pi:** Low power, but needs arm64 images, and an SD card is a poor place for a database.

**Consequences:** No running cost besides power, and full control over the machine. Availability is that of a home PC: power cuts, ISP outages and an old disk. Because the hardware is old, backups (ADR-028) matter more. Ports published by Docker bypass `ufw`, because Docker writes its own iptables rules. The frontend only publishes on `127.0.0.1` (ADR-024), so this doesn't expose anything. Any port published on all interfaces later would bypass the firewall.

**Related:** ADR-010 (container-per-service, hosting deferred), ADR-024 (access), ADR-028 (backups)

---

## ADR-024: Reach the app only over Tailscale, with HTTPS from `tailscale serve`

**Status:** Accepted

**Context:** The app has to be reachable at home, and ADR-021 assumed friends would use it from their own homes too. Putting it on the public internet would need a domain, TLS and the hardening ADR-021 deferred. Plain HTTP on a LAN IP also isn't a secure context. In that case `navigator.clipboard` (the invite dialog's copy button) doesn't work, and the `Secure` auth cookie (ADR-019) is never set, so login fails.

**Decision:**
- Install Tailscale on the host (machine name `packrat`). In the Tailscale admin console, enable MagicDNS and HTTPS certificates.
- `sudo tailscale serve --bg 80` publishes the frontend container at `https://packrat.<tailnet>.ts.net`, with a Let's Encrypt certificate that Tailscale issues and renews. This is the canonical URL, and `FRONTEND_URL` (used for invite links, ADR-021) points to it.
- Friends get access by having the `packrat` machine shared with their own Tailscale account (machine sharing). They don't join the tailnet, so they can't see any other devices.
- The frontend container publishes port 80 on `127.0.0.1` only, so `tailscale serve` is the only way in. At home, your own devices use Tailscale as well.
- Nothing is port-forwarded, so the app isn't on the public internet.

**Alternatives considered:**
- **LAN only:** Simplest, but friends can only use it at your place, and HTTPS would need a self-run CA.
- **Public, via Cloudflare Tunnel or port forwarding plus Let's Encrypt:** Friends don't need a client app, but it needs a domain and makes the login page public, so rate limiting and password reset would have to come first.
- **Also serve plain HTTP on the LAN IP as a fallback:** Works on devices without Tailscale, but login fails there (`Secure` cookie, ADR-019) and the clipboard doesn't work, so it would only be a broken copy of the app.
- **Self-signed certificate or own CA on the LAN:** HTTPS without Tailscale, but every device has to be told by hand to trust the CA.

**Consequences:** Every friend installs Tailscale and creates an account once. The free plan covers this scale. Certificates renew without any work. Every device, including yours at home, needs the Tailscale client. New connections depend on Tailscale's coordination service being up; existing ones keep working.

**Related:** ADR-019 (cookie needs HTTPS), ADR-021 (invite links), ADR-023 (host), ADR-025 (single origin)

---

## ADR-025: One origin in deployment: the frontend's Nginx serves the app and proxies `/api` to the backend

**Status:** Accepted

**Context:** ADR-016 chose CORS over a proxy so the frontend and backend could be deployed separately. In practice both run on one host, reached at its `ts.net` URL (ADR-024). The Angular build bakes `apiUrl` in at build time, and with ADR-026 the image is built in CI, which doesn't know the host. An absolute API URL would also need a CORS entry, and credentialed CORS for the cookie (ADR-019).

**Decision:**
- The frontend image is Nginx serving the Angular build. `location /api/` passes to `http://backend:8080`. Every other path falls back to `index.html` so Angular routes work on reload.
- `environment.ts` uses `apiUrl: '/api'`. Because it's relative, one image works under any hostname. Dev keeps `http://localhost:8080/api` and CORS (ADR-016) unchanged.
- Set `client_max_body_size 5m` in Nginx to match Spring's `max-file-size`. Nginx's default of 1MB would reject valid 2MB uploads (ADR-015) with `413` before they reach the backend.
- Only the frontend's port 80 is published, on `127.0.0.1` (ADR-024). The backend and Postgres are only reachable on the compose network.
- The CORS config stays in the backend. `CORS_ALLOWED_ORIGINS` is set to the `ts.net` URL, but same-origin requests never use it.

**Alternatives considered:**
- **Keep separate origins (frontend on :80, backend on :8080) with CORS:** What ADR-016 planned, but the API host has to be baked into the image or injected at container start, and CORS has to allow credentials.
- **Inject `apiUrl` at container start (`envsubst` into a `config.json`):** Keeps the cross-origin setup working, but adds moving parts for no gain here.
- **A separate reverse-proxy container (Caddy, Traefik):** A fourth container to do what the existing Nginx does in a few lines. TLS already comes from Tailscale.

**Consequences:** Same origin also makes ADR-019's cookie trivially same-site, with no credentialed CORS in production. The frontend and backend now deploy together behind this Nginx. Deploying them apart later means going back to ADR-016's CORS, which stays in place for exactly that. The frontend repo gains one `nginx.conf`.

**Related:** ADR-010 (containers), ADR-015 (upload size), ADR-016 (CORS, still used in dev), ADR-019 (cookie), ADR-026 (images built in CI)

---

## ADR-026: Build images with GitHub Actions, publish them to GHCR and run them from `pack-rat-deploy`

**Status:** Accepted

**Context:** ADR-012 planned a `pack-rat-deploy` repo for the full compose file. The server needs the backend and frontend images from somewhere. Building them on an old computer is slow (Maven plus npm) and would put a JDK and Node on the server.

**Decision:**
- Each code repo gets a multi-stage `Dockerfile`:
  - Backend: build in `maven:3-eclipse-temurin-21`, run in `eclipse-temurin:21-jre`.
  - Frontend: build in `node:22` (`npm ci`, `ng build`), run in `nginx:stable-alpine` (ADR-025).
- Each code repo gets a workflow that, on every push to `main`, builds the image and pushes it to `ghcr.io/marcoschoch1/pack-rat-backend` or `ghcr.io/marcoschoch1/pack-rat-frontend` with the tags `latest` and `sha-<short sha>`. The platform is `linux/amd64`; add `linux/arm64` only if the host turns out to be ARM.
- The packages are public. The images contain no secrets (ADR-014), so the server can pull them without credentials.
- Create `pack-rat-deploy` with `compose.yaml`, `.env.example`, `backup.sh` (ADR-028) and a README that links to the runbook in `deployment/deployment.md`. Image tags come from `.env` (`BACKEND_TAG`, `FRONTEND_TAG`, default `latest`), so rolling back means setting a `sha-…` tag and running `docker compose up -d`.
- Pin Postgres to a major version, `postgres:18`, instead of `latest`, with the volume at `/var/lib/postgresql` as 18 expects. A major upgrade needs a dump and restore, so it has to be a deliberate step. The dev `compose.yaml` in `pack-rat-backend` gets the same pin. Postgres has a `pg_isready` healthcheck, and the backend waits for it with `depends_on: condition: service_healthy`.
- Deploys are manual. On the server: `git pull` (deploy repo), then `docker compose pull && docker compose up -d`. Nothing updates automatically.

**Alternatives considered:**
- **Build on the server from clones of both repos:** No CI and no registry, but slow builds on old hardware, a JDK and Node on the server, and no record of which commit is running.
- **Auto-deploy (Watchtower or a deploy step in CI):** Hands-off, but a broken `main` goes live unattended, and Watchtower needs access to the Docker socket.
- **Docker Hub:** Works the same way, but it's another account and has pull rate limits. GHCR sits next to the code.

**Consequences:** Every merge to `main` produces an image named after its commit, and a rollback is one env var. The cost is two Dockerfiles and two workflows to maintain. Actions minutes are free for public repos. If a repo is private, its package is too, and the server needs `docker login ghcr.io` with a `read:packages` token. Updating stays a manual SSH step, which is intended at this scale.

**Related:** ADR-009 (repo split), ADR-010 (containers), ADR-012 (deploy repo), ADR-014 (secrets via env), ADR-025 (frontend Nginx)

---

## ADR-027: Create the first account from environment variables when the database is empty

**Status:** Accepted

**Context:** Sign-up needs an invite (ADR-021), and only a logged-in user can create an invite. The `dev` user is only seeded under the `local` profile. On a fresh production database nobody can log in, so nobody can invite anyone.

**Decision:**
- At startup, an `ApplicationRunner` checks two things: the `users` table is empty, and both `ADMIN_USERNAME` and `ADMIN_PASSWORD` are set. If both hold, it creates that user with the same validation and `PasswordEncoder` as register (ADR-021). Otherwise it does nothing.
- It isn't tied to a profile. The `local` profile never sets these vars, so dev is unaffected.
- After the first successful start, delete `ADMIN_PASSWORD` from `.env` on the server. The account has no special rights. It's an ordinary user who happens to send the first invites.

**Alternatives considered:**
- **Insert the user by hand with SQL:** No code, but hashing a BCrypt password by hand is easy to get wrong, and it's one more runbook chore.
- **Open registration while the table is empty:** The first visitor becomes the first user, which is a race on any reachable instance.
- **Run the dev seeding in prod:** Creates an account with a known, hardcoded password.

**Consequences:** About 15 lines plus one test. It's safe to leave in: it can't create a second user or reset a password, because it only acts on an empty table. Restoring an empty database would trigger it again only if the vars are still set. Until it's removed, the password sits in plain text in `.env`, readable only by the server's admin.

**Related:** ADR-007 (auth), ADR-014 (secrets via env), ADR-021 (invite-only sign-up)

---

## ADR-028: Nightly `pg_dump` backups, stored on a different disk

**Status:** Accepted

**Context:** All data, including images (ADR-018), lives in Postgres, on one Docker volume, on the old computer's disk (ADR-023). A volume is not a backup. Also, `ddl-auto=update` changes the schema when a new backend version starts.

**Decision:**
- `backup.sh` in `pack-rat-deploy` runs nightly at 03:00 from cron. It runs `docker compose exec -T postgres pg_dump -Fc` and writes `/srv/backups/packrat-YYYY-MM-DD.dump`, then deletes dumps older than 14 days.
- `/srv/backups` is a different physical disk than the system disk: a USB drive or a second internal disk. Now and then, copy the latest dump to the Mac with `scp` for a copy outside the machine.
- Run `backup.sh` by hand before every deploy.
- The restore steps are in the runbook. Test them once right after setup: restore into a fresh database and log in.

**Alternatives considered:**
- **Copy the Docker volume's files:** Postgres has to be stopped for a consistent copy, and the copy is tied to the Postgres major version.
- **Off-site cloud backup (restic to Backblaze B2 or S3):** Survives fire or theft, but needs an account, credentials and a small cost. This is the natural next step.

**Consequences:** Covers a dead disk and a bad deploy, losing at most a day of data. A fire or theft is only covered by the occasional copy on the Mac. The dumps are small (around 50KB per image, ADR-018).

**Related:** ADR-018 (images in Postgres), ADR-023 (old hardware), ADR-026 (deploy repo)
