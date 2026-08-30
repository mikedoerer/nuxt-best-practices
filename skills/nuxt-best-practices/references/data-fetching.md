# Data Fetching

## Decision tree

1. **Fetching during component setup, result should render on the server AND hydrate on the client without a duplicate request?**
   → `useFetch(url, options)` if it's a plain GET-ish call to a URL, or `useAsyncData(key, handler)` if you need to transform the response, combine multiple `$fetch` calls, or call a composable that isn't a raw URL.
2. **One-off call triggered by a user action (submit, click, refresh button)?**
   → plain `$fetch(url, options)`. Don't wrap it in `useFetch`/`useAsyncData` — those are for setup-time, SSR-shareable fetches.
3. **Call only ever needed client-side (e.g. depends on `window`, a browser-only SDK, or geolocation)?**
   → `$fetch` inside `onMounted`, or `useFetch` with `{ server: false }` if you still want the composable's loading/error/refresh state.
4. **Needs to hit a secret-holding third-party API?**
   → Don't call the third party directly from the component. Create `server/api/*.ts`, put the secret in `runtimeConfig` there, and call your own endpoint with `useFetch('/api/...')`.

## Keys and loops

`useFetch`/`useAsyncData` derive a cache key from the URL/composable name and call-site by default. That default breaks down when:
- The same composable is called multiple times in a `v-for` (e.g. one card per item) — pass an explicit unique `key`, e.g. `useAsyncData(`item-${id}`, ...)`.
- The same URL is called with different runtime params in a way Nuxt can't distinguish automatically.

Both composables must be called at the top level of `<script setup>` (or a composable invoked from there) — never inside `if`, `for`, `onMounted`, or after an `await`. Same rule as any Vue composition function.

## Avoiding duplicate fetches

The entire point of `useFetch`/`useAsyncData` is that the server-rendered result is serialized into the page payload and reused on the client during hydration — the client does **not** refetch on mount. If you see two network requests (one server, one client) for the same data, check for:
- A second `$fetch` call for the same data made separately in `onMounted` — remove it and read from the existing `data` ref instead.
- A `key` that isn't actually stable/identical between server and client render (e.g. it includes `Date.now()` or `Math.random()`).
- `useFetch` used with reactive params but no `watch`/`immediate` handling considered — check `references/data-fetching.md#reactive-params` pattern below.

## Reactive params

```ts
const page = ref(1)
const { data } = await useFetch('/api/items', {
  query: { page }, // reactive — refetches automatically when page.value changes
})
```
Passing a `ref` directly (not `page.value`) inside `query`/`body` makes Nuxt watch it and refetch automatically — you don't need a manual `watch` + refetch unless you need custom debounce/side-effect logic.

## Caching and refresh

- `refresh()` / `execute()` — returned from both composables, re-runs the fetch on demand (e.g. after a mutation).
- `getCachedData` option — control whether Nuxt reuses payload/cache data instead of refetching on client-side navigation.
- `lazy: true` — don't block navigation on this fetch; combine with checking `pending`/`status` in the template for a loading state.
- `dedupe` option — controls request deduplication when the same key fires again before the first resolves (default `cancel`).

## Error handling

`useFetch`/`useAsyncData` return an `error` ref instead of throwing — check it in the template/script rather than wrapping the call in try/catch. For fatal page-level errors (e.g. a detail page for a nonexistent ID), throw via `createError({ statusCode: 404, statusMessage: '...' })` and let Nuxt's error page handle it, rather than manually redirecting or rendering an inline "not found" state.

## Server-only fetch (no client involvement at all)

If data is only ever needed to render HTML and never needs to be reactive/refetched client-side (e.g. a static content page pulling from a CMS at request time), fetching inside a `server/api` route or directly in a server middleware and returning fully-rendered data is valid — just make sure it's still surfaced to the page via `useAsyncData`/`useFetch` so it's part of the SSR payload, not a bypass that skips Nuxt's data layer entirely.
