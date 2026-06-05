# REES46 JS SDK v3 Examples

## Browser bootstrap

```html
<script>
  window.r46 =
    window.r46 ||
    function () {
      ;(window.r46.q = window.r46.q || []).push(arguments)
    }
  window.r46('init', 'SHOP_TOKEN')
</script>
<script async src="https://example-cdn/r46v3.js"></script>
```

Replace `SHOP_TOKEN` and SDK URL with client-specific values.

## Safe helper for apps with SSR

```js
export function r46(...args) {
  if (typeof window === 'undefined') return
  window.r46 =
    window.r46 ||
    function () {
      ;(window.r46.q = window.r46.q || []).push(arguments)
    }
  window.r46(...args)
}
```

## React/SPA initialization

Initialize once, for example in app bootstrap or a top-level provider effect:

```js
import { useEffect } from 'react'

export function Rees46Bootstrap({ shopToken }) {
  useEffect(() => {
    if (!shopToken) return

    window.r46 =
      window.r46 ||
      function () {
        ;(window.r46.q = window.r46.q || []).push(arguments)
      }

    window.r46('init', shopToken, '', undefined, undefined, undefined, true)
  }, [shopToken])

  return null
}
```

Do not initialize inside product cards or components that render repeatedly.

## Product page

```js
window.r46('track', 'view', {
  id: product.id,
  price: product.price,
})

window.r46(
  'recommend',
  'product-related',
  {
    code: 'product-related',
    item: product.id,
    limit: 8,
  },
  renderRecommendations,
  console.error
)
```

## Category page

```js
window.r46('track', 'category', category.id)

window.r46(
  'recommend',
  'category-popular',
  {
    code: 'category-popular',
    category: category.id,
    limit: 12,
  },
  renderProducts,
  console.error
)
```

## Add to cart

```js
window.r46('track', 'cart', {
  id: item.productId,
  amount: item.quantity,
  price: item.price,
  recommended_by: item.recommendedBy,
  recommended_code: item.recommendedCode,
})
```

## Full cart snapshot

Use this when synchronizing the complete current cart:

```js
window.r46(
  'track',
  'cart',
  cart.items.map((item) => ({
    id: item.productId,
    amount: item.quantity,
    price: item.price,
    line_id: item.lineId,
  }))
)
```

## Purchase after checkout success

```js
window.r46('track', 'purchase', {
  order: order.id,
  email: order.customerEmail,
  phone: order.customerPhone,
  order_price: order.total,
  delivery_type: order.deliveryType,
  payment_type: order.paymentType,
  promocode: order.promocode,
  products: order.items.map((item) => ({
    id: item.productId,
    amount: item.quantity,
    price: item.price,
    line_id: item.lineId,
  })),
})
```

Fire this once per completed order. Guard against duplicate route renders on success pages.

## Full search

```js
window.r46(
  'search',
  {
    search_query: query,
    limit: 24,
    page,
    filters,
    sort_by: sortBy,
    order: sortOrder,
  },
  function (response) {
    renderSearchResults(response)
  },
  function (error) {
    console.error('REES46 search failed', error)
  }
)
```

## Instant suggest

```js
window.r46(
  'suggest',
  {
    search_query: inputValue,
    extended: true,
  },
  function (response) {
    renderSuggest(response)
  },
  console.error
)
```

## Custom event

```js
window.r46('track', 'lead_form_submit', {
  value: 1,
  form_id: 'footer',
})
```

If `value` is provided for a custom event, it must be an integer.

## Get device/session IDs

```js
window.r46('did', (did) => {
  console.log('REES46 device id', did)
})

window.r46('sid', (sid) => {
  console.log('REES46 session id', sid)
})
```

## Consent-delayed initialization

When consent is required, do not send regular tracking before consent. Initialize with consent disabled or wait until consent is granted, depending on client requirements:

```js
window.r46(
  'init',
  'SHOP_TOKEN',
  '',
  undefined,
  undefined,
  { consent: false },
  true
)

// Later, after consent is granted, initialize/enable according to the client's consent architecture.
```

If changing this flow, verify against SDK behavior in source because consent handling affects init request and storage.
