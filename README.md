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

## Design docs

Decisions and specs made before/during implementation, kept in this repo:

- [`adr/adr.md`](adr/adr.md) — architecture decision records (why, not just what)
- [`database/data-model-mvp.md`](database/data-model-mvp.md) — entities, relationships, fixed value sets
- [`backend/api-design.md`](backend/api-design.md) — REST endpoints, request/response shapes
- [`frontend/ui-design.md`](frontend/ui-design.md) — rough screen layouts

### Open decisions

ADRs still in status **Proposed**, waiting to be accepted or rejected:

- [ADR-019](adr/adr.md#adr-019-send-the-jwt-in-an-httponly-cookie-instead-of-storing-it-in-localstorage) — send the JWT in an httpOnly cookie instead of localStorage

## GIT

It will use the conventional commits structure of how commit messages are written

For details: https://www.conventionalcommits.org/en/v1.0.0/
## Issue tracking

New issues are auto-added to the project board via a GitHub Actions workflow ([`.github/workflows/add-to-project.yml`](.github/workflows/add-to-project.yml)), so the board stays current without manual triage.

## Status
 
MVP complete — all core functions above work end to end. See the issue board for current and upcoming work.

## Next improvements

Found while using the MVP:

1. **In-app delete confirmation** — replace the browser `confirm()` popup with an in-app dialog. :white_check_mark:
2. **Uniform item tiles** — every tile in the dashboard grid is the same size; price paid and price now fit on the tile (one line or two). :white_check_mark:
3. **Self sign-up** — users can register their own account instead of the hardcoded dev login.
4. **Delete uploaded image** — an image can be removed from an item after it was uploaded. :white_check_mark:
5. **Visual polish** — a more stylish look; the visual direction in [`frontend/ui-design.md`](frontend/ui-design.md) is still open.
6. **Browser tab branding** — the tab shows a Pack-Rat favicon/logo and "Pack-Rat" as the page title. :white_check_mark:
7. **Multiple collections** — users can have more than one collection. With two or more, an overview dashboard tracks totals across all collections, like the item dashboard does for one collection. Data model already allows this (ADR-003). 
