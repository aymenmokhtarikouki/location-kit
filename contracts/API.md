# location-kit — HTTP contract

Canonical shapes for the address book + geocoding proxy. Apps may remap paths
and wrap payloads in their envelope (commonly `{ "data": … }`).

## Address book (authed — compose with your auth middleware)

- `GET /me/addresses` → `Address[]`
- `POST /me/addresses` — `{ label?, line?, lat, lng, isDefault? }` → `Address`
  — the FIRST address becomes default automatically.
- `PATCH /me/addresses/:id` — partial update; coordinate changes recompute the
  app's `extra` payload (H3 cells etc.).
- `DELETE /me/addresses/:id` — deleting the default promotes the newest remaining.
- `POST /me/addresses/:id/default` — flips exactly one default on.

```jsonc
// Address
{
  "id": "…", "userId": "…",
  "label": "Home" | null,
  "line": "Gullweg 12, 12203 Berlin" | null,
  "lat": 52.52, "lng": 13.405,
  "isDefault": true,
  "extra": { /* app fields: placeId, city, H3 cells, … */ },
  "createdAt": "…", "updatedAt": "…"
}
```

Errors: `{ "error": { "code", "message" } }` — `NOT_FOUND` (404),
`INVALID_COORDINATES` (400).

## Geocoding proxy (server-side — provider token never reaches clients)

- `GET /geocoding/autocomplete?q=…` → `GeocodeResult[]` (empty for q < 3 chars)
- `GET /geocoding/reverse?lat=…&lng=…` → `GeocodeResult | null`

```jsonc
// GeocodeResult
{
  "label": "Gullweg 12, 12203 Berlin, Germany",
  "lat": 52.52, "lng": 13.405,
  "placeId": "address.123",
  "parts": {
    "street": "Gullweg", "houseNumber": "12",
    "postalCode": "12203", "city": "Berlin",
    "country": "Germany", "countryCode": "DE"
  }
}
```
