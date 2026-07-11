# Integrating location-kit

## Install

```bash
npm install @locationkit/addresses
```

## Implement the store

`AddressStore` maps onto your addresses table. The kit owns the behavior
every app reimplements — first address auto-defaults, setting a default flips
the previous one, deleting the default promotes the newest — your app owns
the columns.

`E` (the extension payload) is where app-specific columns live; populate
them in `buildExtra`, which runs on create and whenever coordinates change:

```ts
const addresses = createAddressService<MyExtra>({
  store,
  buildExtra: ({ lat, lng }) => ({ cells: computeCells(lat, lng) }), // optional
})
```

## Geocoding

`mapboxGeocoder({ token })` gives you server-side autocomplete + reverse
geocoding with pre-split address parts. Keep the token server-side; expose
your own thin proxy route.

## Migrating from an existing implementation

The kits were extracted from production systems, and these rules kept those
migrations safe:

1. **Never rewrite a working flow in one step.** Keep your endpoint URLs,
   response envelopes and (for realtime) socket event names byte-identical;
   swap the implementation underneath, one endpoint at a time.
2. **Data stays put.** The store seams map onto your existing tables — new
   capabilities need at most additive columns, never a data migration.
3. **Delete the superseded code in the same change.** Two implementations of
   the same behavior is how drift starts.
4. Where the kit enforces domain rules through policy hooks, your hooks may
   THROW your app's own error types — the kit re-throws them untouched, so
   your API's error contract survives the swap.
