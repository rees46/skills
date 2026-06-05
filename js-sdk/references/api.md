# REES46 JS SDK v3 API Reference

REES46 JS SDK v3 uses a browser queue function, normally `window.r46`:

```js
window.r46(command, ...args)
```

The public web API is the queue command surface. Use `r46`, REES46 domains, and REES46 dashboard terminology consistently.

## General rules from Reference API

- `shop_id` / shop token identifies the shop. JS SDK injects it after `init`; do not hard-code it into later SDK commands.
- `shop_secret` is for server-to-server/secret operations only. Never expose it in browser code.
- `stream` labels the source of data for segmentation. It is merchant-defined, case-sensitive, max 16 chars, and should use Latin letters, digits, and `_`.
- Web SDK should be initialized on every opened page. In SPAs initialize once with `isSpa: true` and let route-level tracking send page events.
- SDK-generated `did` and `sid` are handled automatically by the browser SDK after initialization.

## Loading/setup

Reference setup snippet loads the queue stub and optimizes SDK bundle loading from the default REES46 CDN (`cdn.rees46.ru`) in the page `<head>`:

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
</script>
```

A minimal project-local bootstrap is still valid when CDN URL is provided separately:

```js
window.r46 =
  window.r46 ||
  function () {
    ;(window.r46.q = window.r46.q || []).push(arguments)
  }
```

## Initialization

```js
window.r46('init', shopToken)
```

Full observed/reference signature:

```js
window.r46(
  'init',
  shopToken,
  stream,
  successCallback,
  errorCallback,
  options,
  isSpa
)
```

Arguments:

- `shopToken` — required shop ID/token string.
- `stream` — optional source label, max 16 characters.
- `successCallback` — optional function.
- `errorCallback` — optional function.
- `options` — optional object:
  - `cookieless: boolean`; default is `false`; if cookies are unavailable, SDK can fall back to local storage.
  - `did: string`; use only when an external device-ID system owns the identifier.
  - `api_host: string`; custom API host.
  - `cdn_host: string`; custom CDN host, empty value means the default `cdn.rees46.ru`.
  - `pictures_host: string`; custom pictures host, empty/undefined means default.
  - `consent: boolean`.
- `isSpa` — optional boolean. Pass `true` for SPA session refresh behavior; the SDK checks session ID during tracking and refreshes expired sessions.

Examples:

```js
window.r46('init', 'SHOP_TOKEN')
window.r46('init', 'SHOP_TOKEN', 'web')
window.r46('init', 'SHOP_TOKEN', '', undefined, undefined, { cookieless: true }, true)
window.r46('init', 'SHOP_TOKEN', '', undefined, undefined, { did: 'DEVICE_ID' }, true)
window.r46('init', 'SHOP_TOKEN', '', null, null, {
  api_host: 'api.custom.com',
  cdn_host: '',
  pictures_host: undefined,
})
```

## Identity/session helpers

```js
window.r46('did', (did) => {})
window.r46('sid', (sid) => {})
window.r46('abSegment', (segment) => {})
window.r46('reset_session')
```

## Tracking

```js
window.r46('track', eventName, data, successCallback)
```

Declared ecommerce events:

- `view`
- `cart`
- `remove_from_cart`
- `category`
- `wish`
- `remove_wish`
- `purchase`
- `search`
- `banner_view`
- `banner_click`

Unknown event names are sent as custom events to `/push/custom`; custom event `value`, when present, must be an integer.

### Product item shape

Most product events accept an ID string/number or object:

```js
{ id: 'PRODUCT_ID', amount: 1, price: 1999, line_id: 'LINE_ID' }
```

Supported item fields include:

- `id` — required for object items.
- `amount` or `quantity` — quantity.
- `price`
- `line_id`
- `fashion_size` or `custom.fashion_size`
- `recommended_by`
- `recommended_code`
- `banner_id` for banner/slider tracking.

### Product view

```js
window.r46('track', 'view', { id: 'PRODUCT_ID' })
```

### Category view

```js
window.r46('track', 'category', 'CATEGORY_ID')
```

### Cart

Single add/update:

```js
window.r46('track', 'cart', { id: 'PRODUCT_ID', amount: 2, price: 1999 })
```

Full cart snapshot:

```js
window.r46('track', 'cart', [
  { id: 'PRODUCT_1', amount: 1 },
  { id: 'PRODUCT_2', amount: 3 },
])
```

Remove from cart:

```js
window.r46('track', 'remove_from_cart', { id: 'PRODUCT_ID', amount: 1 })
```

Fetch current SDK cart by current `did`:

```js
window.r46('cart', 'get', successCallback, errorCallback)
```

Response shape from Reference API:

```js
{ status: 'success', data: { items: [{ uniqid: 'SKU_1', quantity: 1 }] } }
```

Clearing a cart requires secret-key server API; do not implement it in browser code.

### Wishlist

```js
window.r46('track', 'wish', { id: 'PRODUCT_ID' })
window.r46('track', 'wish', [{ id: 'PRODUCT_1' }, { id: 'PRODUCT_2' }])
window.r46('track', 'remove_wish', { id: 'PRODUCT_ID' })
```

### Purchase

```js
window.r46('track', 'purchase', {
  order: 'ORDER_ID',
  email: 'customer@example.com',
  phone: '+10000000000',
  order_price: 3998,
  promocode: 'SALE10',
  delivery_type: 'courier',
  payment_type: 'card',
  custom: { source: 'checkout' },
  products: [{ id: 'PRODUCT_1', amount: 2, price: 1999, line_id: '1' }],
})
```

`products` is required and must be non-empty.

### Search tracking

```js
window.r46('track', 'search', 'phone')
window.r46('track', 'search', { query: 'phone', results: ['1', '2'] })
```

### Banner/slider tracking

```js
window.r46('track', 'banner_view', {
  recommended_code: 'SLIDER_CODE',
  banner_id: 'BANNER_ID',
})
window.r46('track', 'banner_click', {
  recommended_code: 'SLIDER_CODE',
  banner_id: 'BANNER_ID',
})
```

## UTM tracking

```js
window.r46('utm', { utm_source: 'newsletter', utm_campaign: 'spring' })
```

## Recommendations

```js
window.r46('recommend', code, params, successCallback, errorCallback)
```

Use the dynamic block code as the second argument. The code is visible in the REES46 dashboard/block markup (for legacy snippets, `data-recommender-code`). Avoid deprecated forms like `recommend`, `dynamic`, or legacy algorithm names as the code argument unless the SDK source/client dashboard explicitly requires them.

Example:

```js
window.r46(
  'recommend',
  'related-products',
  {
    item: 'PRODUCT_ID',
    limit: 8,
    filters: { price: { min: 100, max: 500 } },
    extended: true,
    with_locations: true,
  },
  function (response) {
    // render response
  },
  function (error) {
    console.error(error)
  }
)
```

Request params from Reference API:

- `item` — product ID; required for blocks using Similar/Also bought algorithms.
- `exclude` — product IDs to exclude.
- `category` — category ID; required for category-page blocks.
- `search_query` — required for search-based blocks.
- `limit` — maximum products to return.
- `locations` — comma-separated location IDs; returns available products for those locations.
- `brands` — include only these brands.
- `exclude_brands` — exclude these brands.
- `categories` — include only these categories.
- `discount` — only discounted products when `true`.
- `extended` — request full product info. Reference API mentions `1`; SDK examples use `true`.
- `prevent_shuffle` — disable response shuffling.
- `page` — pagination page, default `1`.
- `resize_image` — supported sizes: `120`, `140`, `160`, `180`, `200`, `220`.
- `with_locations` — only works when `extended` is true; response includes `location_ids`.

Initialization helpers:

```js
window.r46('dynamic_init')
window.r46('category_init') // legacy transition helper
window.r46('products_init')
window.r46('add_css', 'recommendations')
```

## Collections

```js
window.r46('collection', id, params, successCallback, errorCallback)
```

Params:

- `location`
- `email`
- `phone`
- `external_id`
- `loyalty_id`

Example:

```js
window.r46('collection', 'COLLECTION_ID', {
  location: 'LOCATION',
  email: 'EMAIL',
  phone: 'PHONE',
  external_id: 'EXTERNAL_ID',
  loyalty_id: 'LOYALTY_ID',
}, successCallback, errorCallback)
```

Declarative widget markup from reference can be adapted to REES46 branding:

```html
<div
  class="r46-collection-products-results"
  data-collection-id="COLLECTION_ID"
  data-collection-callback="console.log"
  data-collection-error="console.log"
></div>
```

## Search

There are two search modes: instant/typeahead (`suggest`) and full search (`search`). Reference API notes a response limit of 10,000 items even when more products match.

### Full search

```js
window.r46(
  'search',
  {
    type: 'full_search',
    search_query: 'phone',
    limit: 24,
    page: 1,
    offset: 0,
    category_limit: 10,
    brand_limit: 50,
    brands: ['Brand A'],
    excluded_brands: ['brand b'],
    colors: ['black'],
    fashion_sizes: ['M'],
    locations: ['warehouse-1'],
    merchants: ['merchant_1'],
    excluded_merchants: ['merchant_2'],
    price_min: 100,
    price_max: 500,
    categories: ['cat-1'],
    exclude: ['PRODUCT_ID'],
    widgetable: true,
    available: true,
    ignored: false,
    extended: true,
    collapse: false,
    filters: { color: ['black'] },
    segment: 'segment-id',
    sort_by: 'price',
    order: 'asc',
    no_clarification: true,
    filters_search_by: 'popularity',
    vector: true,
    debug: true,
  },
  successCallback,
  errorCallback
)
```

Search params and gotchas:

- `sort_by` values documented in Reference API: `popular`, `price`, `discount`, `sales_rate`, `date`, `price_margin`, `rating`, `relevance`.
- `order`: `asc` or `desc` (`desc` default in Reference API text).
- `filters_search_by`: `name`, `quantity`, or `popularity`; overrides dashboard filter-option sorting.
- `collapse: false` prevents grouping products with same `group_id`.
- `discount` may be `true`, `false`, or omitted.
- `excluded_brands` must be lowercase; brand exclusion requires product feed `brand` field.
- `excluded_merchants` only works with the merchant/location whitelist semantics described by API; test carefully.
- `debug: true` requires debug cookie (`r46_debug=1`) and exposes debug fields in response.
- Price aggregations (`price_range`, `price_ranges`, `price_median`) ignore zero-price products.

### Instant suggest

```js
window.r46(
  'suggest',
  {
    type: 'instant_search',
    search_query: 'pho',
    categories: ['cat-1'],
    locations: ['warehouse-1'],
    segment: 'segment-id',
    extended: true,
    excluded_merchants: ['merchant_2'],
    excluded_brands: ['brand b'],
    search_scope: 'name,brand,categories',
    collapse: false,
    vector: true,
  },
  successCallback,
  errorCallback
)
```

`search_scope` is a comma-separated whitelist. Common parameters include `params`, `brand`, `type`, `model`, `categories`, and `name`; book-specific subparameters include `author`, `publisher`, `illustrator`, `editor`, and `series`.

### Search helpers

```js
window.r46('search_collection', { id: 'COLLECTION_ID' }, successCallback)
window.r46('search_init', true)
window.r46('full_search_init')
```

Blank/zero-results search exists in the broader SDK ecosystem; only use it from web if confirmed in the current JS SDK source.

## Profile

```js
window.r46('profile', 'set', data, successCallback, errorCallback)
window.r46('profile', 'get', params, successCallback)
```

## Reputation

Reference API says shop review fetching is not implemented in JS SDK. Do not use browser code for secret-backed reputation endpoints unless SDK source confirms support.

```js
window.r46('reputation', action, data, successCallback) // source-observed; verify before new integrations
```

## Subscriptions and web push

```js
window.r46('webpush_subscription', 'web_push_subscribe', successCallback, errorCallback)
window.r46('webpush_subscription', 'web_push_supported', (supported) => {})
window.r46('webpush_subscription', 'web_push_subscribed', (subscribed) => {})
```

Subscription aliases `subscription`, `webpush_subscription`, and `email_subscription` share the same command handler in the current skill baseline.

Manage email/SMS subscriptions:

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

Change only selected channels by sending only those fields. Reference API also supports status checks:

```js
window.r46('subscription', 'check', successCallback, errorCallback)
window.r46('subscription', 'manage', params)
window.r46('email_subscription', 'manage', params)
```

Server/system subscription operations such as unsubscribe, complaint, hard bounce, blacklist, changed subscriptions lists, and secret-backed exports are not browser tasks.

## Triggers

```js
window.r46('subscribe_trigger', trigger, params)
window.r46('unsubscribe_trigger', trigger, params)
window.r46('check_trigger', trigger, params, successCallback, errorCallback)
```

Reference trigger status examples are easy to confuse: validate against SDK/source or API behavior before shipping.

```js
// Reference API example labels this as Back in Stock status.
window.r46('check_trigger', 'product_price_decrease', params, successCallback, errorCallback)

// Reference API example labels this as Product Price Drop status.
window.r46('check_trigger', 'product_available', params, successCallback, errorCallback)
```

Trigger params generally include `item_id` plus at least one identifier such as `email`, `phone`, `external_id`, `loyalty_id`, or current SDK `did` where supported.

## Promo codes

```js
window.r46('get_promo_code', params, successCallback, errorCallback)
```

## NPS

```js
window.r46('nps', 'categories', successCallback, errorCallback)
window.r46('nps', 'channels', successCallback, errorCallback)
window.r46('nps', 'review', data, successCallback, errorCallback)
```

Save review:

```js
window.r46('nps', 'review', {
  channel: 'channel_code',
  category: 'category_code',
  rate: 7,
  comment: 'Some comment',
}, successCallback, errorCallback)
```

`rate`, `channel`, and `category` are required. `comment` and identifiers such as email/phone/loyalty/order may be optional depending on the configured survey and channel; SDK uses `did` automatically for web/mobile.

NPS review listing requires `shop_secret` and is not for browser SDK.

## Loyalty

```js
window.r46('loyalty', 'join', data, successCallback, errorCallback)
window.r46('loyalty', 'status', data, successCallback, errorCallback)
```

## Segments

```js
window.r46('segment', 'add', data)
window.r46('segment', 'remove', data)
window.r46('segment', 'get', successCallback)
window.r46('segment', 'get', params, successCallback)
```

Examples:

```js
window.r46('segment', 'add', {
  email: 'jane@example.com',
  phone: '+10000000000',
  segment_id: 'SEGMENT_ID',
})

window.r46('segment', 'remove', {
  email: 'jane@example.com',
  segment_id: 'SEGMENT_ID',
})

// Without contacts, current did is used automatically in JS SDK.
window.r46('segment', 'remove', { segment_id: 'SEGMENT_ID' })

window.r46('segment', 'get', function (segments) {
  // [{ id: 313, type: 'static' }, { id: 314, type: 'dynamic' }]
})
```

For add/remove, Reference API identifies users by `email` and/or `phone`; JS SDK may omit them and use current `did`.

## Orders

```js
window.r46('orders', 'last_for_user', params, successCallback)
```

`last_for_user` fetches products from the user's last purchase; other user-order and custom-order endpoints require server/secret API and are not supported by JS SDK per reference.

## Products

Fetch products by params:

```js
window.r46('products', params, successCallback, errorCallback)
```

Common product-list params:

- `brands`
- `merchants`
- `categories`
- `locations`
- `limit`
- `page`
- `filters`
- `price_min`, `price_max`
- `sort_by`, `sort_dir` / `order`

Example:

```js
window.r46('products', { limit: 5, categories: '313' }, successCallback, errorCallback)
```

Counters:

```js
window.r46('products', 'counters', 'PRODUCT_ID', successCallback, errorCallback)
```

Counters response has `daily` (24h) and `now` (15m) objects with `view`, `cart`, and `purchase` counts.

Info:

```js
window.r46('products', 'info', params, successCallback, errorCallback) // source-observed; Reference API says JS no implementation for /products/get
```

Not-widgetable products, product info by `shop_secret`, product cart clearing, and product subscription exports are server/secret operations; do not implement in browser SDK.

## Popups, stories, speech, sliders, debug

```js
window.r46('popup', popupId)
window.r46('stories_init')
window.r46('start_stories', code)
window.r46('say', text)
window.r46('slider', params, successCallback, errorCallback)
window.r46('slider_init')
window.r46('debug')
```

Stories: Reference API documents `start_stories` for programmatic story campaign opening; second argument is the story block ID/code. Web `stories_init` availability should be verified in SDK source for new work.

Slider examples:

```html
<div class="r46-slider" data-slider-code="SLIDER_CODE"></div>
```

```js
// Raw data for custom rendering.
window.r46('slider', { code: 'SLIDER_CODE' }, (response) => console.log(response))

// Render into a block with config.
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

Slider params include `code` (required), current SDK identifiers (`did`, email, phone, external_id where supported), and `tags` for comma-separated banner-tag filtering.
