---
name: rees46-js-sdk-v3
description: Helps client agents integrate, debug, or review REES46 JS SDK v3 usage in browser websites and SPAs. Use when adding REES46 tracking, recommendations, search, subscriptions, profile, cart, products, loyalty, NPS, or other r46 queue commands.
---

# REES46 JS SDK v3

You are helping a client integrate or maintain REES46 JS SDK v3.

The SDK is a browser queue API. The default global is `window.r46`; white-label builds may expose another global. Do not import SDK internals from `src/` in client projects unless explicitly working inside the SDK repository.

## Mandatory workflow

1. Inspect the target project first: framework, entrypoints, existing REES46 snippets, router/SPA behavior, consent handling, ecommerce data model.
2. Find whether `window.r46` or a white-label global already exists. Avoid duplicate bootstraps and duplicate `init` calls.
3. Initialize SDK before sending commands. Queue commands are allowed before the built script loads.
4. Use only commands documented in `references/api.md`. If a command or parameter is missing, inspect the SDK source before inventing anything.
5. For ecommerce tracking, preserve product IDs, quantities, prices, order IDs, recommendation attribution, and user identifiers from the host app.
6. For SPA projects, initialize with the SPA flag and trigger page-specific tracking on route changes, not on every component render.
7. Keep credentials/configurable values outside reusable components. The shop token must come from project config/env/server-rendered data.
8. Add verification steps: network request check, callback check, console errors, and duplicate event detection.

## Loading pattern

Use the queue stub before the SDK bundle is available:

```html
<script>
  window.r46 =
    window.r46 ||
    function () {
      ;(window.r46.q = window.r46.q || []).push(arguments)
    }
  window.r46('init', 'SHOP_TOKEN')
</script>
<script async src="/path/to/r46v3.js"></script>
```

For SPA mode:

```js
window.r46('init', 'SHOP_TOKEN', '', undefined, undefined, undefined, true)
```

For cookieless/consent/custom device ID options:

```js
window.r46(
  'init',
  'SHOP_TOKEN',
  '',
  undefined,
  undefined,
  {
    cookieless: true,
    did: 'DEVICE_ID',
    consent: true,
  },
  true
)
```

## References

Load these files when relevant:

- `references/api.md` — public queue commands and parameter shapes.
- `references/examples.md` — common integration examples.
- `references/common-mistakes.md` — mistakes to avoid and review checklist.

## Output expectations

When writing client code:

- Show exactly where initialization code should live.
- Explain which values the client must replace, especially `SHOP_TOKEN` and product/order IDs.
- Mention whether code is safe for SSR. Browser-only code must guard `window` access.
- Include a short manual verification checklist.
