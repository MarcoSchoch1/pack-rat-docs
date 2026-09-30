# Pack-Rat — UI Design (rough)

Rough screen layouts agreed on during design discussion, before implementation. These are layout/field references, not final specs — actual styling happens in Angular with Tailwind CSS (see ADR-011).

## 1. Login

![Login screen mockup](images/login.png)

- Dev/hardcoded login for MVP
- Fields: username, password
- Centered card on a plain background
- After login: `GET /api/collections` — empty list → Create collection screen, otherwise → Dashboard

## 2. Create collection (first login only)

![Create collection mockup](images/create-collection.png)

- Shown once, when the user has no collection yet
- Same centered-card layout as Login
- Field: collection name; submit → `POST /api/collections` → Dashboard
- MVP: one collection per user (ADR-003), so no collection list/switcher — this screen never shows again

## 3. Dashboard (collection overview)

![Dashboard mockup](images/dashboard.png)

- Header: collection name + "Add item" button
- Two metric cards: total price paid, total price now (sum of each item's manually-entered current price — see ADR-017; shows "—" if no item has one set)
- Item grid: photo-first cards (name + price paid), click through to item detail

## 4. Add item form

![Add item form mockup](images/add-item-form.png)

- Fields match the finalized data model: name, self-pulled, price paid, current price (optional), currency, date acquired, condition, marketplace link, picture
- Condition is a dropdown of the fixed grading scale (see `data-model.md`)
- Image upload happens as a separate step from item creation (see ADR-008)
- "Self-pulled" checkbox: when ticked, price paid is set to 0 and disabled; unticking re-enables it (see ADR-020)

## 5. Item detail

![Item detail mockup](images/item-detail.png)

- Thumbnail strip at top reflects the `Image` one-to-many relationship (ADR-004) — multiple photos per item, "+" tile to add another
- Detail table: price paid, date acquired, condition, record-created date
- Edit / Delete actions

## Design direction

- Styling approach: Tailwind CSS utility classes (see ADR-011), not a component library like Angular Material
- Visual direction not yet locked — these layouts are about structure and fields, not final colors/typography
