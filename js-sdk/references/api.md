# REES46 JS SDK v3 API Reference

REES46 JS SDK v3 uses a browser queue function, normally `window.r46`:

```js
window.r46(command, ...args)
```

The SDK entrypoint initializes a `Core` instance and drains queued calls. The public browser API is the command names handled by `Core.perform()`.

## Initialization

```js
window.r46('init', shopToken)
```

Full observed signature:

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
- `stream` — optional string, max 16 characters.
- `successCallback` — optional function.
- `errorCallback` — optional function.
- `options` — optional object:
  - `cookieless: boolean`
  - `did: string`
  - `api_host: string`
  - `cdn_host: string`
  - `pictures_host: string`
  - `consent: boolean`
- `isSpa` — optional boolean. Pass `true` for SPA session refresh behavior.

Examples:

```js
window.r46('init', 'SHOP_TOKEN')
window.r46(
  'init',
  'SHOP_TOKEN',
  '',
  undefined,
  undefined,
  { cookieless: true },
  true
)
window.r46(
  'init',
  'SHOP_TOKEN',
  '',
  undefined,
  undefined,
  { consent: false },
  true
)
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

Use dynamic block code as the second argument. The deprecated forms `recommend`, `dynamic` and legacy algorithm names should not be used.

Example:

```js
window.r46(
  'recommend',
  'related-products',
  {
    code: 'related-products',
    item: 'PRODUCT_ID',
    limit: 8,
    filters: { price: { min: 100, max: 500 } },
  },
  function (response) {
    // render response
  },
  function (error) {
    console.error(error)
  }
)
```

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

## Search

Full search:

```js
window.r46(
  'search',
  {
    search_query: 'phone',
    limit: 24,
    page: 1,
    offset: 0,
    brands: ['Brand A'],
    excluded_brands: ['Brand B'],
    colors: ['black'],
    fashion_sizes: ['M'],
    locations: ['warehouse-1'],
    merchants: ['merchant_1'],
    excluded_merchants: ['merchant_2'],
    price_min: 100,
    price_max: 500,
    categories: ['cat-1'],
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
    filters_search_by: 'name',
    vector: true,
  },
  successCallback,
  errorCallback
)
```

Instant suggest:

```js
window.r46(
  'suggest',
  {
    search_query: 'pho',
    categories: ['cat-1'],
    locations: ['warehouse-1'],
    segment: 'segment-id',
    extended: true,
    excluded_merchants: ['merchant_2'],
    collapse: false,
    vector: true,
  },
  successCallback,
  errorCallback
)
```

Search collection:

```js
window.r46('search_collection', { id: 'COLLECTION_ID' }, successCallback)
```

Search UI init:

```js
window.r46('search_init', true)
window.r46('full_search_init')
```

## Profile

```js
window.r46('profile', 'set', data, successCallback, errorCallback)
window.r46('profile', 'get', params, successCallback)
```

## Reputation

```js
window.r46('reputation', action, data, successCallback)
```

## Subscriptions and web push

```js
window.r46(
  'webpush_subscription',
  'web_push_subscribe',
  successCallback,
  errorCallback
)
window.r46('webpush_subscription', 'web_push_supported', (supported) => {})
window.r46('webpush_subscription', 'web_push_subscribed', (subscribed) => {})
window.r46('subscription', 'check', successCallback, errorCallback)
window.r46('subscription', 'manage', params)
window.r46('email_subscription', 'manage', params)
```

Aliases `subscription`, `webpush_subscription`, and `email_subscription` share the same command handler.

## Triggers

```js
window.r46('subscribe_trigger', trigger, params)
window.r46('unsubscribe_trigger', trigger, params)
window.r46('check_trigger', trigger, params, successCallback, errorCallback)
```

## Promo codes

```js
window.r46('get_promo_code', params, successCallback, errorCallback)
```

## NPS

```js
window.r46('nps', 'review', data, successCallback, errorCallback)
window.r46('nps', 'categories', params, successCallback)
window.r46('nps', 'channels', params, successCallback)
```

## Loyalty

```js
window.r46('loyalty', 'join', data, successCallback, errorCallback)
window.r46('loyalty', 'status', data, successCallback, errorCallback)
```

## Segments

```js
window.r46('segment', 'add', data)
window.r46('segment', 'remove', data)
window.r46('segment', 'get', params, successCallback)
```

## Orders

```js
window.r46('orders', 'last_for_user', params, successCallback)
```

## Products

Fetch products by params:

```js
window.r46('products', params, successCallback, errorCallback)
```

Counters:

```js
window.r46('products', 'counters', params, successCallback, errorCallback)
```

Info:

```js
window.r46('products', 'info', params, successCallback, errorCallback)
```

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
