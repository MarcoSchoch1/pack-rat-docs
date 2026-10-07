# Pack-Rat — API Design (v1)

## Authentication

- **Mechanism:** JWT (stateless token)
- **Flow:** `POST /api/auth/login` validates credentials against the `users` table (BCrypt) and returns a signed JWT
- **Sign-up:** only through a single-use invite link (ADR-021). A logged-in user creates an invite, sends the link to a friend, and the friend registers with it. There is no open registration.
- **Usage:** Angular attaches the token as `Authorization: Bearer <token>` on all subsequent requests, except `GET /api/images/{id}` (public by unguessable UUID, see Images)

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/auth/login` | Validates credentials, returns a JWT |
| POST | `/api/auth/register` | **No auth.** Creates an account from an invite token, returns a JWT like login (ADR-021) |

**POST `/api/auth/register`**
```json
// Request
{
  "inviteToken": "5e8d...",
  "username": "zoro",
  "password": "at-least-8-chars"
}

// Response — 201 Created, same body as POST /api/auth/login
```

- Username: 3–32 characters, trimmed, unique case-insensitively → `409 USERNAME_TAKEN` if taken
- Password: at least 8 characters, no composition rules → `400 VALIDATION_ERROR` if shorter
- Invite missing, expired or already used → `404 INVITE_NOT_FOUND`
- On success the invite is marked used (`usedAt`, `usedBy`) in the same transaction with a conditional update, so a link works once. The row is kept as an audit trail. The new user has no collection yet.

## Invites

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/invites` | Creates a single-use invite link, valid for 7 days. Any logged-in user can invite |
| GET | `/api/invites/{token}` | **No auth.** `200` if the invite exists, is unused and hasn't expired, otherwise `404`. Lets the register screen reject a dead link before showing the form |

**POST `/api/invites`** — no request body. Response — 201 Created:
```json
{
  "url": "https://<frontend>/register?invite=5e8d...",
  "expiresAt": "2026-10-09T10:00:00Z"
}
```
The token is the invite's random UUID (ADR-002), unguessable like image URLs (ADR-018). The frontend base URL in `url` comes from config.

**GET `/api/invites/{token}`** — response `200`:
```json
{ "expiresAt": "2026-10-09T10:00:00Z" }
```

## Collections

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/collections` | List the current user's collections, each with `itemCount`, `totalPricePaid`, `totalPriceNow` (ADR-022) |
| POST | `/api/collections` | Create a collection |
| GET | `/api/collections/{id}` | Get a single collection |
| PUT | `/api/collections/{id}` | Update a collection |
| DELETE | `/api/collections/{id}` | Remove a collection |
| GET | `/api/collections/{id}/overview` | Dashboard data: name, total price paid, total price now |
| GET | `/api/collections/{id}/items` | List items in a collection |
| POST | `/api/collections/{id}/items` | Add a new item to a collection |

**GET `/api/collections`** — response, one aggregate query (ADR-022):
```json
[
  {
    "id": "9f2b...",
    "name": "My One Piece Collection",
    "itemCount": 42,
    "totalPricePaid": 1240.50,
    "totalPriceNow": 980.00
  },
  {
    "id": "b71c...",
    "name": "Pokémon",
    "itemCount": 7,
    "totalPricePaid": 85.00,
    "totalPriceNow": null
  }
]
```
Totals follow the same rules as `/overview`. The overview dashboard adds them up on the client. Amounts are summed without currency conversion (ADR-006).

**GET `/api/collections/{id}/overview`** — response:
```json
{
  "id": "9f2b...",
  "name": "My One Piece Collection",
  "totalPricePaid": 1240.50,
  "totalPriceNow": 980.00
}
```
`totalPriceNow` sums each item's manually-entered `priceNow` (see ADR-017); `null` if no item in the collection has one set.

## Items

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/items/{id}` | Get a single item |
| PUT | `/api/items/{id}` | Update an item |
| DELETE | `/api/items/{id}` | Remove an item |
| POST | `/api/items/{id}/images` | Upload a new image for an item (multipart) |
| GET | `/api/items/{id}/images` | List an item's images |

**POST `/api/collections/{collectionId}/items`**
```json
// Request
{
  "name": "OP01 Monkey D. Luffy Alt Art",
  "pricePaid": 45.00,
  "selfPulled": false,
  "priceNow": null,
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
  "selfPulled": false,
  "priceNow": null,
  "currency": "CHF",
  "dateAcquired": "2026-03-14",
  "condition": "NEAR_MINT",
  "marketplaceLink": "https://www.tcgplayer.com/...",
  "createdAt": "2026-09-17T10:22:00Z",
  "updatedAt": "2026-09-17T10:22:00Z"
}
```

If `selfPulled` is `true`, the backend stores `pricePaid` as `0` regardless of the value sent (ADR-020). `selfPulled` defaults to `false` if omitted.

## Images

Images are their own resource, uploaded against an existing item — this keeps item creation a clean JSON request, with the multipart upload handled separately.

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/images/{id}` | Image bytes with the stored `Content-Type` and a long `Cache-Control`. **No auth**, so a plain `<img src>` works (ADR-018) |
| DELETE | `/api/images/{id}` | Remove a specific image |

**POST `/api/items/{itemId}/images`** — response:
```json
{
  "id": "a1b2...",
  "itemId": "c3a1...",
  "url": "/api/images/a1b2...",   // derived from id, not stored
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
  "message": "pricePaid must not be negative",
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
| 409 | Conflict (e.g. username already taken) |
| 500 | Server error |

