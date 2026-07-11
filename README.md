# location-kit

Shared **location** toolkit — the domain that
repeats in every product but belongs to none of them: user address books,
geocoding, and (roadmap) the geo math both backends currently duplicate.

Consume as a **git submodule** at `vendor/location-kit` with `file:` deps —
same pattern as clustermap-kit / auth-kit. Storage-agnostic: the kit owns
behavior, your app owns tables via store seams (Prisma, raw SQL, anything).

## Packages

| Package | What | Deps |
| --- | --- | --- |
| `@aymenkits/location-addresses` | Generic address book: CRUD, first-address-auto-default, default flipping, default-deletion promotes newest, coordinate validation, app-extension payload via `buildExtra` (e.g. H3 cells via `@aymenkits/clustermap-core`'s `computeCells`), `Geocoder` seam + **Mapbox adapter** (server-side autocomplete/reverse with pre-split street/houseNumber/postalCode/city parts — extracted from production once). | — |
| `@aymenkits/location-geo` *(roadmap)* | The math every delivery app hand-rolls: haversine distance, delivery-radius / service-area checks, bbox helpers. | — |

## Quick start

```ts
import { createAddressService, mapboxGeocoder } from '@aymenkits/location-addresses'
import { computeCells } from '@aymenkits/clustermap-core' // optional geo enrichment

const addresses = createAddressService<MyExtra>({
  store: myAddressStore, // maps to UserAddress / customer_addresses
  hooks: { buildExtra: (i) => ({ ...computeCells(i.lat, i.lng) }) },
})

const geocoder = mapboxGeocoder({
  accessToken: env.MAPBOX_TOKEN, // stays server-side
  country: 'de',
  language: 'de',
})
```

Typical registration composition (with auth-kit — the two kits are independent):

```ts
// app's register endpoint
const session = await auth.verifyOtp({ channel, destination, code, profile })
if (body.address) await addresses.create(session.user.id, body.address)
```

## Docs

- [`contracts/API.md`](contracts/API.md) — HTTP shapes (address book + geocoding proxy) for Flutter/web clients.
- [`docs/INTEGRATION.md`](docs/INTEGRATION.md) — install + store recipes (Prisma / raw SQL).

## Development

```bash
npm install && npm test    # 9 unit tests
npm run build              # tsc → dist
npm run setup              # consumer one-liner
```
