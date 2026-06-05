# REES46 JS SDK v3 Common Mistakes

## Using a non-REES46 global in REES46 integrations

Bad:

```js
window.someOtherSdk('init', SHOP_TOKEN)
```

Good:

```js
window.r46('init', SHOP_TOKEN)
```

Use REES46/r46 naming consistently unless the client explicitly uses a white-label global.

## Duplicate initialization

Bad:

```js
// Runs on every product card render
window.r46('init', SHOP_TOKEN)
```

Good:

```js
// Runs once at application bootstrap
window.r46('init', SHOP_TOKEN, 'web', undefined, undefined, undefined, true)
```

## Missing SPA mode

Bad:

```js
window.r46('init', SHOP_TOKEN)
```

Good for SPA:

```js
window.r46('init', SHOP_TOKEN, 'web', undefined, undefined, undefined, true)
```

The final `isSpa` flag enables session checks during tracking and refreshes expired sessions.

## Invalid stream values

Bad:

```js
window.r46('init', SHOP_TOKEN, 'web-production-v2-long-name')
window.r46('init', SHOP_TOKEN, 'web-prod!')
```

Good:

```js
window.r46('init', SHOP_TOKEN, 'web_prod')
```

`stream` is merchant-defined, case-sensitive, max 16 chars, and should use Latin letters, digits, and `_`.

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

## Exposing secret-key endpoints in browser

Bad:

```js
fetch('/products/cart/clear?shop_secret=' + SHOP_SECRET)
fetch('/nps/reviews?shop_secret=' + SHOP_SECRET)
```

Good:

```js
// Put secret-backed cart clearing, exports, imports, NPS review listing, etc. on the server.
```

The browser SDK should not contain `shop_secret`.

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

## Clearing cart from browser

Bad:

```js
window.r46('cart', 'clear', { email: customer.email })
```

Good:

```js
// Browser SDK can fetch current cart:
window.r46('cart', 'get', success, failure)

// Cart clearing is a secret-backed server API operation.
```

## Deprecated recommendation syntax

Avoid:

```js
window.r46('recommend', 'dynamic', params, success, failure)
```

Prefer dynamic block code:

```js
window.r46('recommend', 'block-code', { limit: 8 }, success, failure)
```

## Expecting locations without extended recommendations

Bad:

```js
window.r46('recommend', 'block-code', { with_locations: true }, success, failure)
```

Good:

```js
window.r46('recommend', 'block-code', {
  extended: true,
  with_locations: true,
}, success, failure)
```

`with_locations` is ignored unless `extended` is true.

## Incorrect search exclusion assumptions

Bad:

```js
window.r46('search', {
  type: 'full_search',
  search_query: 'phone',
  excluded_brands: ['Samsung'],
})
```

Good:

```js
window.r46('search', {
  type: 'full_search',
  search_query: 'phone',
  excluded_brands: ['samsung'],
})
```

Brand exclusion expects lowercase values and requires the product feed to include `brand`.

## Using unsupported secret/server methods as JS SDK commands

Reference API marks several endpoints as not implemented or not needed in JS SDK. Do not add browser commands for:

- imports and category mutations;
- changed subscription list exports;
- system unsubscribe/complaint/hard-bounce/blacklist operations;
- NPS review listing;
- product info by secret endpoint if current SDK source does not support it;
- not-widgetable products;
- cart clearing;
- custom orders and full user order lists.

## Confusing trigger names from reference docs

The reference examples for product-available and product-price-decrease trigger status are easy to mix up. Before shipping:

- verify `subscribe_trigger`, `unsubscribe_trigger`, and `check_trigger` names in the current SDK source;
- test the exact trigger with a real `item_id` and identifier;
- document whether it is Back in Stock or Price Drop in client code.

## Slider markup with old class names

Bad:

```html
<div class="legacy-slider" data-slider-code="SLIDER_CODE"></div>
```

Good:

```html
<div class="r46-slider" data-slider-code="SLIDER_CODE"></div>
```

## Checklist for reviews

- Exactly one bootstrap/`init` per page lifecycle.
- SPA apps pass the final `isSpa` flag as `true` when needed.
- `stream` is short, valid, and intentionally chosen.
- Browser-only code is guarded in SSR projects.
- No `shop_secret` in browser bundles.
- Product/order IDs match the merchant catalog IDs known to REES46.
- Purchase event fires once per order.
- Full cart snapshots are not confused with add-to-cart events.
- Search callbacks handle success and error.
- Search brand exclusions are lowercase and tested against feed data.
- Recommendation `with_locations` is paired with `extended: true`.
- Consent/cookieless requirements are explicit.
- No imports from `rees46-js-sdk-v3/src/...` in client projects.
- No undocumented `window.r46.someMethod()` usage.
