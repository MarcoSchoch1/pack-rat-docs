# Pack-Rat — API Design (v1)

## Authentication

- **Mechanism:** JWT (stateless token)
- **Flow:** `POST /api/auth/login` validates credentials (hardcoded dev user for MVP) and returns a signed JWT
- **Usage:** Angular attaches the token as `Authorization: Bearer <token>` on all subsequent requests
- Structured so real accounts for friends can be added later without changing the auth mechanism — only how credentials are validated changes.

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/auth/login` | Validates credentials, returns a JWT |

## Collections

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/collections` | List the current user's collections (MVP: returns one) |
| POST | `/api/collections` | Create a collection |
| GET | `/api/collections/{id}` | Get a single collection |
| GET | `/api/collections/{id}/overview` | Dashboard data: name, total price paid, total price now |

**GET `/api/collections/{id}/overview`** — response:
```json
{
  "id": "9f2b...",
  "name": "My One Piece Collection",
  "totalPricePaid": 1240.50,
  "totalPriceNow": null
}
```
`totalPriceNow` aggregation logic is an open decision — see the data model doc.

## Items

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/collections/{collectionId}/items` | List items in a collection |
| POST | `/api/collections/{collectionId}/items` | Add a new item |
| GET | `/api/items/{id}` | Get a single item |
| PUT | `/api/items/{id}` | Update an item |
| DELETE | `/api/items/{id}` | Remove an item |

**POST `/api/collections/{collectionId}/items`**
```json
// Request
{
  "name": "OP01 Monkey D. Luffy Alt Art",
  "pricePaid": 45.00,
  "currency": "CHF",
  "dateAcquired": "2026-03-14",
  "condition": "NEAR_MINT",
  "marketplaceLink": "https://www.tcgplayer.com/..."
}

// Response — 201 Created
{
  "id": "c3a1e2b0-...",
  "collectionId": "9f2b...",
  "name": "OP01 Monkey D. Luffy Alt Art",
  "pricePaid": 45.00,
  "currency": "CHF",
  "dateAcquired": "2026-03-14",
  "condition": "NEAR_MINT",
  "marketplaceLink": "https://www.tcgplayer.com/...",
  "createdAt": "2026-09-17T10:22:00Z",
  "updatedAt": "2026-09-17T10:22:00Z"
}
```

## Images

Images are their own resource, uploaded against an existing item — this keeps item creation a clean JSON request, with the multipart upload handled separately.

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/items/{itemId}/images` | Upload a new image for an item (multipart) |
| GET | `/api/items/{itemId}/images` | List an item's images |
| DELETE | `/api/images/{id}` | Remove a specific image |

**POST `/api/items/{itemId}/images`** — response:
```json
{
  "id": "a1b2...",
  "itemId": "c3a1...",
  "url": "/uploads/a1b2....jpg",
  "originalFilename": "luffy_front.jpg",
  "contentType": "image/jpeg",
  "fileSizeBytes": 84213,
  "createdAt": "2026-09-17T10:25:00Z"
}
```

## Error handling

All endpoints return a consistent error body:

```json
{
  "error": "VALIDATION_ERROR",
  "message": "pricePaid must be positive",
  "field": "pricePaid"
}
```

Standard HTTP status codes:

| Status | Meaning |
|---|---|
| 400 | Validation error |
| 401 | Unauthenticated (missing/invalid JWT) |
| 403 | Forbidden (authenticated but not permitted) |
| 404 | Resource not found |
| 500 | Server error |

## Open decisions

- **Total price now** aggregation — no stored current-value field exists yet; each item carries a marketplace link for manual lookup instead. Logic to be defined in a GitHub issue.
