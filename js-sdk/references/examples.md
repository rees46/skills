# REES46 JS SDK v3 Examples

Examples use REES46/r46 naming consistently.

## Browser bootstrap with CDN hints

Put this in the page `<head>` when you own the raw HTML shell:

```html
<script>
  ;(function () {
    window.r46 =
      window.r46 ||
      function () {
        ;(window.r46.q = window.r46.q || []).push(arguments)
      }

    var c = 'https://cdn.rees46.ru'
    var v = '/v3.js'
    var s = {
      link: [
        { href: c, rel: 'dns-prefetch' },
        { href: c, rel: 'preconnect' },
        { href: c + v, rel: 'preload', as: 'script' },
      ],
      script: [{ src: c + v, async: '' }],
    }

    Object.keys(s).forEach(function (tag) {
      s[tag].forEach(function (attrs) {
        var el = document.createElement(tag)
        for (var name in attrs) el.setAttribute(name, attrs[name])
        document.head.appendChild(el)
      })
    })
  })()

  window.r46('init', 'SHOP_TOKEN')
</script>
```

Replace `SHOP_TOKEN`; keep `cdn.rees46.ru` as the default bundle CDN unless the client explicitly provides a custom CDN host.

## Minimal browser bootstrap

```html
<script>
  window.r46 =
    window.r46 ||
    function () {
      ;(window.r46.q = window.r46.q || []).push(arguments)
    }
  window.r46('init', 'SHOP_TOKEN')
</script>
<script async src="https://cdn.rees46.ru/v3.js"></script>
```

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

Initialize once, for example in app bootstrap or a top-level provider effect. `stream` is optional and merchant-defined; pass final `true` for SPA mode.

```js
import { useEffect } from 'react'

export function Rees46Bootstrap({ shopToken, stream = 'web' }) {
  useEffect(() => {
    if (!shopToken) return

    window.r46 =
      window.r46 ||
      function () {
        ;(window.r46.q = window.r46.q || []).push(arguments)
      }

    window.r46('init', shopToken, stream, undefined, undefined, undefined, true)
  }, [shopToken, stream])

  return null
}
```

Do not initialize inside product cards or components that render repeatedly.

## Custom API/CDN/picture domains

```js
window.r46('init', 'SHOP_TOKEN', 'web', null, null, {
  api_host: 'api.custom.com',
  cdn_host: '',
  pictures_host: undefined,
})
```

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
    item: product.id,
    limit: 8,
    extended: true,
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
    category: category.id,
    limit: 12,
    extended: true,
  },
  renderProducts,
  console.error
)
```

## Recommendations with locations

`with_locations` is honored only when `extended` is true.

```js
window.r46(
  'recommend',
  'store-available-products',
  {
    item: product.id,
    locations: 'store-1,store-2',
    extended: true,
    with_locations: true,
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

## Fetch SDK cart

```js
window.r46(
  'cart',
  'get',
  function ({ data }) {
    console.log(data.items.length)
  },
  function (error) {
    console.error(error)
  }
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
    type: 'full_search',
    search_query: query,
    limit: 24,
    page,
    category_limit: 10,
    categories,
    filters,
    sort_by: sortBy,
    order: sortOrder,
    extended: true,
    collapse: false,
  },
  function (response) {
    renderSearchResults(response)
  },
  function (error) {
    console.error('REES46 search failed', error)
  }
)
```

## Full search with exclusions

`excluded_brands` must be lowercase. Merchant exclusion depends on location/merchant whitelist semantics, so verify with real catalog data.

```js
window.r46(
  'search',
  {
    type: 'full_search',
    search_query: 'shoes',
    locations: ['store-1'],
    merchants: ['merchant-1', 'merchant-2'],
    excluded_merchants: ['merchant-2'],
    excluded_brands: ['brand to hide'],
    filters_search_by: 'popularity',
    brand_limit: 50,
  },
  renderSearchResults,
  console.error
)
```

## Instant suggest

```js
window.r46(
  'suggest',
  {
    type: 'instant_search',
    search_query: inputValue,
    extended: true,
    collapse: false,
  },
  function (response) {
    renderSuggest(response)
  },
  console.error
)
```

## Search with debug fields

Requires the `r46_debug=1` cookie.

```js
window.r46('search', {
  type: 'full_search',
  search_query: query,
  debug: true,
}, console.log, console.error)
```

## Product collection

```js
window.r46(
  'collection',
  'COLLECTION_ID',
  {
    location: 'LOCATION',
    email: customer.email,
    phone: customer.phone,
    external_id: customer.id,
    loyalty_id: customer.loyaltyId,
  },
  renderCollection,
  console.error
)
```

## Products list and counters

```js
window.r46('products', { limit: 5, categories: '313' }, renderProducts, console.error)

window.r46('products', 'counters', product.id, function (stats) {
  console.log(stats.daily.view, stats.now.cart)
}, console.error)
```

## Subscription management

```js
window.r46('subscription', 'manage', {
  email: 'my@example.com',
  phone: '+100000000000',
  email_bulk: true,
  email_chain: true,
  email_transactional: true,
  sms_bulk: true,
  sms_chain: true,
  sms_transactional: true,
})
```

Send only fields you intend to change for partial updates.

## Segment membership

```js
window.r46('segment', 'add', {
  email: 'jane@example.com',
  phone: '+10000000000',
  segment_id: 'SEGMENT_ID',
})

window.r46('segment', 'remove', {
  segment_id: 'SEGMENT_ID',
})

window.r46('segment', 'get', function (segments) {
  console.log(segments)
})
```

If email/phone are omitted in JS SDK, current `did` is used where supported.

## NPS

```js
window.r46('nps', 'categories', function (categories) {
  renderCategories(categories)
}, console.error)

window.r46('nps', 'channels', function (channels) {
  renderChannels(channels)
}, console.error)

window.r46('nps', 'review', {
  channel: 'website',
  category: 'checkout',
  rate: 9,
  comment: 'Fast checkout',
}, function () {
  console.log('Review saved')
}, console.error)
```

## Custom event

```js
window.r46('track', 'lead_form_submit', {
  value: 1,
  form_id: 'footer',
})
```

If `value` is provided for a custom event, it must be an integer.

## Slider

Standard rendering:

```html
<div class="r46-slider" data-slider-code="SLIDER_CODE"></div>
```

Raw data for custom rendering:

```js
window.r46('slider', { code: 'SLIDER_CODE' }, function (response) {
  renderSlider(response.payload)
}, console.error)
```

Render into a selected block:

```js
window.r46('slider', {
  code: 'SLIDER_CODE',
  block: 'r46-slider',
  config: {
    dotBgColor: 'red',
    dotActiveBgColor: 'green',
    dotHoverBgColor: 'yellow',
  },
})
```

## Stories

```js
window.r46('start_stories', 'STORY_BLOCK_CODE')
```

Use for programmatic story campaign opening, for example on button click.

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
  'web',
  undefined,
  undefined,
  { consent: false },
  true
)
```

If changing this flow, verify against SDK behavior in source because consent handling affects init request and storage.
