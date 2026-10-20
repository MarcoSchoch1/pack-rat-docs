# Pack-Rat
 
A personal collection tracker built for tabletop and trading card game collectors who don't fit neatly into mainstream apps.
 
## Why this exists
 
Apps like Collectr try to cover every TCG under the sun, but that breadth comes at a cost: niche sets, regional exclusives, and lesser-known products are often unsearchable or missing entirely. Pack-Rat exists to solve that gap for a small, specific audience — starting with me and my friends — by trading broad catalog coverage for full control over what gets tracked and how.
 
This repository (and its issue board) is where the project's features are planned, built, and tracked from the ground up.
 
## What Pack-Rat does
 
Pack-Rat lets you build and manage your own collection without relying on someone else's product database.
 
**Core functions (MVP):**
 
- **Collection dashboard** — view your collection at a glance, including the total price paid across all items.
- **Add items manually** — since items aren't pulled from an external catalog, you add exactly what you own yourself.
- **Item details** — track name, price paid, date acquired, and condition for each item.
- **Item photos** — upload a picture for each item so your collection is visual, not just a spreadsheet.
- **Marketplace links** — attach a link to TCGPlayer, Cardmarket, or similar so current value is one click away, without building a price-lookup system.
- **Login** — a development login for now, structured so real accounts for friends can be added later without reworking the app.
## Tech stack
 
- **Backend:** Spring Boot ([pack-rat-backend](https://github.com/MarcoSchoch1/pack-rat-backend))
- **Frontend:** Angular + Tailwind CSS ([pack-rat-frontend](https://github.com/MarcoSchoch1/pack-rat-frontend))
- **Database:** PostgreSQL via Docker (in repo: pack-rat-backend)
- **Auth:** Stateless JWT (dev/hardcoded login for now, real accounts later)
- **Hosting:** old home PC on Ubuntu Server, Docker Compose, images from GHCR, reachable over Tailscale ([pack-rat-deploy](https://github.com/MarcoSchoch1/pack-rat-deploy), to be created)

## Design docs

Decisions and specs made before/during implementation, kept in this repo:

- [`adr/adr.md`](adr/adr.md) — architecture decision records (why, not just what)
- [`database/data-model-mvp.md`](database/data-model-mvp.md) — entities, relationships, fixed value sets
- [`backend/api-design.md`](backend/api-design.md) — REST endpoints, request/response shapes
- [`frontend/ui-design.md`](frontend/ui-design.md) — rough screen layouts
- [`deployment/deployment.md`](deployment/deployment.md) — hosting architecture, setup runbook, updates, backups

## GIT

It will use the conventional commits structure of how commit messages are written

For details: https://www.conventionalcommits.org/en/v1.0.0/
## Issue tracking

New issues are auto-added to the project board via a GitHub Actions workflow ([`.github/workflows/add-to-project.yml`](.github/workflows/add-to-project.yml)), so the board stays current without manual triage.

## Status
 
MVP complete — all core functions above work end to end. See the issue board for current and upcoming work.

## Next improvements

Found while using the MVP. Open tickets on the project board:

**Security — JWT in an httpOnly cookie**

- [B1](https://github.com/MarcoSchoch1/pack-rat-backend/issues/28) — Set the JWT as an httpOnly cookie and add logout
- [B2](https://github.com/MarcoSchoch1/pack-rat-backend/issues/29) — Read the JWT from the cookie in the auth filter
- [B3](https://github.com/MarcoSchoch1/pack-rat-backend/issues/30) — Enable credentialed CORS with explicit origins
- [F1](https://github.com/MarcoSchoch1/pack-rat-frontend/issues/20) — Send requests with credentials and stop storing the token
- [F2](https://github.com/MarcoSchoch1/pack-rat-frontend/issues/21) — Track session state from expiresAt, add logout and 401 handling

**Features**

- [B4](https://github.com/MarcoSchoch1/pack-rat-backend/issues/31) — Return totals per collection from `GET /api/collections`
- [F4](https://github.com/MarcoSchoch1/pack-rat-frontend/issues/23) — Route after login by number of collections

**Deployment — app side**

- [B5](https://github.com/MarcoSchoch1/pack-rat-backend/issues/32) — Add a multi-stage Dockerfile
- [B6](https://github.com/MarcoSchoch1/pack-rat-backend/issues/33) — Publish the backend image to GHCR from GitHub Actions
- [B7](https://github.com/MarcoSchoch1/pack-rat-backend/issues/34) — Pin dev Postgres to `postgres:18` with a healthcheck
- [B8](https://github.com/MarcoSchoch1/pack-rat-backend/issues/35) — Make the prod profile fully environment-driven
- [B9](https://github.com/MarcoSchoch1/pack-rat-backend/issues/36) — Create the first account from env vars on an empty database
- [F6](https://github.com/MarcoSchoch1/pack-rat-frontend/issues/24) — Add a multi-stage Dockerfile and nginx.conf
- [F7](https://github.com/MarcoSchoch1/pack-rat-frontend/issues/25) — Use a relative apiUrl in the production build
- [F8](https://github.com/MarcoSchoch1/pack-rat-frontend/issues/26) — Publish the frontend image to GHCR from GitHub Actions

**Deployment — deploy repo**

- [D1](https://github.com/MarcoSchoch1/pack-rat-deploy/issues/13) — Add compose.yaml and .env.example
- [D2](https://github.com/MarcoSchoch1/pack-rat-deploy/issues/14) — Add backup.sh

**Deployment — server setup and operations**

- [O1](https://github.com/MarcoSchoch1/pack-rat-deploy/issues/1) — Create the pack-rat-deploy repository
- [O2](https://github.com/MarcoSchoch1/pack-rat-deploy/issues/2) — Install Ubuntu Server 26.04 on the home computer
- [O3](https://github.com/MarcoSchoch1/pack-rat-deploy/issues/3) — Make the server survive power cuts
- [O4](https://github.com/MarcoSchoch1/pack-rat-deploy/issues/4) — Harden SSH, set up ufw, enable unattended upgrades
- [O5](https://github.com/MarcoSchoch1/pack-rat-deploy/issues/5) — Install Docker Engine and the Compose plugin
- [O6](https://github.com/MarcoSchoch1/pack-rat-deploy/issues/6) — Install Tailscale and publish the app
- [O7](https://github.com/MarcoSchoch1/pack-rat-deploy/issues/7) — Mount the backup disk at `/srv/backups`
- [O8](https://github.com/MarcoSchoch1/pack-rat-deploy/issues/8) — Smoke-test the full compose stack on the Mac
- [O9](https://github.com/MarcoSchoch1/pack-rat-deploy/issues/9) — First deploy on the server
- [O10](https://github.com/MarcoSchoch1/pack-rat-deploy/issues/10) — Schedule nightly backups and test a restore
- [O11](https://github.com/MarcoSchoch1/pack-rat-deploy/issues/11) — Connect your devices and onboard the first friend
- [O12](https://github.com/MarcoSchoch1/pack-rat-deploy/issues/12) — Dry-run an update and a rollback

