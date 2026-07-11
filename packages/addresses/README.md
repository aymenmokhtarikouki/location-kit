# @locationkit/addresses

Address book (CRUD + single-default flip logic) generic over your extra columns, plus a Geocoder seam with a ready Mapbox forward-geocoding adapter.

## Install

```bash
npm install @locationkit/addresses
```

Installs with it: nothing else — zero dependencies.

## You provide

- `AddressStore` — your table (yuma keeps 9 H3 columns on it; lineo a lean one)
- For geocoding: your Mapbox token (server-side)
- Optional `onWrite` hook (e.g. compute H3 cells via @clustermap/core)

The package never owns tables, never imports an ORM, HTTP framework, or
provider SDK it can take as a parameter — storage and delivery are seams your
app implements on its own stack.

## Quick example

```ts
import { createAddressService, mapboxGeocoder } from '@locationkit/addresses'

const addresses = createAddressService({ store, onWrite })
const geocoder = mapboxGeocoder({ token: process.env.MAPBOX_TOKEN })
```

## Pairs with

- `@clustermap/core` computeCells in the onWrite hook
- `@authkit/express` guards the routes

Kits pair **by shape, never by import** — pass the sibling kit, your own
service, or a stub in tests.

## Docs

Full contracts and integration guides live in the repo:
https://github.com/aymenmokhtarikouki/location-kit (`contracts/`, `docs/`).

## License

UNLICENSED — published for use by the author's applications.
