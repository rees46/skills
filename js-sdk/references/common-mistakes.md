# REES46 JS SDK v3 Common Mistakes

## Duplicate initialization

Bad:

```js
// Runs on every product card render
window.r46('init', SHOP_TOKEN)
```

Good:

```js
// Runs once at application bootstrap
window.r46('init', SHOP_TOKEN, '', undefined, undefined, undefined, true)
```

## Calling before queue stub exists

Bad:

```js
window.r46('track', 'view', product.id)
```

Good:

```js
window.r46 =
  window.r46 ||
  function () {
    ;(window.r46.q = window.r46.q || []).push(arguments)
  }
window.r46('track', 'view', product.id)
```

## Using SDK in SSR without `window` guard

Bad:

```js
window.r46('init', token)
```

Good:

```js
if (typeof window !== 'undefined') {
  window.r46('init', token)
}
```

## Inventing methods

Bad:

```js
window.r46.trackPurchase(order)
window.r46.purchase(order)
```

Good:

```js
window.r46('track', 'purchase', orderData)
```

## Dropping recommendation attribution

When tracking product view, cart, purchase, banner view, or banner click after a recommendation interaction, preserve:

- `recommended_by`
- `recommended_code`
- `banner_id` where relevant

## Empty purchase products

Bad:

```js
window.r46('track', 'purchase', { order: order.id, products: [] })
```

Good:

```js
window.r46('track', 'purchase', {
  order: order.id,
  products: order.items.map((item) => ({
    id: item.productId,
    amount: item.quantity,
    price: item.price,
  })),
})
```

The SDK throws when purchase `products` is missing or empty.

## Wrong custom event value type

For custom events, `value` must be an integer if present.

Bad:

```js
window.r46('track', 'lead', { value: '12.5' })
```

Good:

```js
window.r46('track', 'lead', { value: 12 })
```

## Replacing full cart with single-item event accidentally

An array means full cart snapshot and sets `full_cart = true`. An object means a single cart event.

```js
window.r46('track', 'cart', [{ id: '1', amount: 2 }]) // full cart snapshot
window.r46('track', 'cart', { id: '1', amount: 2 }) // single item event
```

## Deprecated recommendation syntax

Avoid:

```js
window.r46('recommend', 'dynamic', params, success, failure)
```

Prefer dynamic block code:

```js
window.r46(
  'recommend',
  'block-code',
  { code: 'block-code', limit: 8 },
  success,
  failure
)
```

## Checklist for reviews

- Exactly one bootstrap/`init` per page lifecycle.
- SPA apps pass the final `isSpa` flag as `true` when needed.
- Browser-only code is guarded in SSR projects.
- Product/order IDs match the merchant catalog IDs known to REES46.
- Purchase event fires once per order.
- Full cart snapshots are not confused with add-to-cart events.
- Search callbacks handle success and error.
- Consent/cookieless requirements are explicit.
- No imports from `rees46-js-sdk-v3/src/...` in client projects.
- No undocumented `window.r46.someMethod()` usage.
