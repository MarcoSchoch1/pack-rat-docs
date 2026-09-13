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
 
- **Backend:** Spring Boot (repo-name: pack-rat-backend)
- **Frontend:** Angular (repo-name: pack-rat-frontend)
- **Database:** PostgreSQL (via Docker) (in repo: pack-rat-backend)
## Status
 
Early development — MVP scope is defined, technical setup and first features are in progress. See the issue board for current and upcoming work.
 
