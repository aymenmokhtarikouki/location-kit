# Integration guide

Same submodule mechanics as clustermap-kit/auth-kit (see clustermap-kit's
INTEGRATION.md for the general update/deploy flow).

## Add + depend

```bash
git submodule add git@github.com:aymenmokhtarikouki/location-kit.git vendor/location-kit
npm --prefix vendor/location-kit run setup
```

```jsonc
"dependencies": {
  "@locationkit/addresses": "file:vendor/location-kit/packages/addresses"
},
"scripts": { "locationkit:setup": "npm --prefix vendor/location-kit run setup" }
```

Deploys/CI: `git submodule update --init` + `npm run locationkit:setup` BEFORE
the consumer `npm install`.

## yuma_backend (Prisma) — AddressStore on `UserAddress`

```ts
import { computeCells } from '@clustermap/core'

const store: AddressStore<YumaExtra> = {
  listByUser: (userId) => prisma.userAddress.findMany({ where: { userId } }).then(rowsToAddresses),
  findById: (userId, id) => prisma.userAddress.findFirst({ where: { id, userId } }).then(rowToAddress),
  create: (userId, d) => prisma.userAddress.create({
    data: { userId, label: d.label, line1: d.line, lat: d.lat, lng: d.lng,
            isDefault: d.isDefault, ...d.extra },          // extra spreads cellR4..R12, placeId…
  }).then(rowToAddress),
  update: (userId, id, d) => prisma.userAddress.update({ where: { id }, data: flatten(d) }).then(rowToAddress),
  delete: (userId, id) => prisma.userAddress.delete({ where: { id } }).then(() => {}),
  clearDefault: (userId) =>
    prisma.userAddress.updateMany({ where: { userId }, data: { isDefault: false } }).then(() => {}),
}

const addresses = createAddressService<YumaExtra>({
  store,
  hooks: { buildExtra: (i) => ({ ...computeCells(i.lat, i.lng) }) }, // keeps map clustering fed
})
```

## lineo-backend (pg) — AddressStore on `customer_addresses`

```ts
const store: AddressStore = {
  listByUser: async (userId) =>
    (await pool.query(`SELECT * FROM customer_addresses WHERE user_id=$1`, [userId])).rows.map(rowToAddress),
  create: async (userId, d) =>
    rowToAddress((await pool.query(
      `INSERT INTO customer_addresses (user_id, label, address, lat, lng, is_default)
       VALUES ($1,$2,$3,$4,$5,$6) RETURNING *`,
      [userId, d.label, d.line, d.lat, d.lng, d.isDefault],
    )).rows[0]),
  clearDefault: (userId) =>
    pool.query(`UPDATE customer_addresses SET is_default=false WHERE user_id=$1`, [userId]).then(() => {}),
  // findById / update / delete analogous — always filter by user_id.
}
```

## Geocoding route (replaces the duplicated proxy modules)

```ts
const geocoder = mapboxGeocoder({ accessToken: env.MAPBOX_ACCESS_TOKEN, country: 'de', language: 'de' })
router.get('/geocoding/autocomplete', async (req, res) =>
  res.json(createApiResponse(await geocoder.autocomplete(String(req.query.q ?? '')))))
router.get('/geocoding/reverse', async (req, res) =>
  res.json(createApiResponse(await geocoder.reverse(Number(req.query.lat), Number(req.query.lng)))))
```
