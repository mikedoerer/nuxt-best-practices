---
name: nuxt-best-practices
description: >
  Best practices for Nuxt 3/4: data fetching, rendering modes, server
  routes, state, SEO, and config/module setup.

  TRIGGER WHEN: creating/editing Nuxt pages, components, composables,
  plugins, or middleware; fetching data (useFetch, useAsyncData, $fetch,
  server routes); choosing SSR/SSG/ISR/CSR or routeRules; shared state
  (useState vs Pinia); Nitro server code (server/api, server/middleware);
  SEO/meta/sitemaps; nuxt.config.ts, modules, runtimeConfig, env vars;
  images/fonts/hydration; hydration mismatches, duplicate fetches,
  "window is not defined".

  SYMPTOMS: fetch()/axios in onMounted instead of useFetch/useAsyncData;
  a request firing both server- and client-side; process.env read in app
  code instead of runtimeConfig; window/localStorage with no client
  guard; global state in a module-level variable instead of useState;
  secret API calls made from the Vue app instead of server/api; the
  whole app forced into one rendering mode instead of routeRules;
  explicit imports for already-auto-imported composables/components.
metadata:
  version: "1"
---

# Nuxt Best Practices

**Core principle:** Prefer Nuxt's built-in conventions (file-based routing, auto-imports, `useFetch`/`useAsyncData`, Nitro server routes, `routeRules`) over hand-rolled equivalents. Nuxt's data-fetching and rendering model exists specifically to avoid hydration mismatches and duplicate requests — bypassing it re-introduces exactly those bugs.

## Decision Workflow

Follow this sequence when building or editing a Nuxt feature.

### 1. Where does this code run?

Decide the execution context before writing anything — it determines which API is even valid:

- **Universal (runs on server during SSR, then again on client for hydration):** default for `<script setup>` in pages/components. Never touch `window`/`document`/`localStorage` here without an `import.meta.client` (or legacy `process.client`) guard, or better, isolate it in a `.client.vue` component or a client-only plugin.
- **Server-only (Nitro):** anything under `server/api/`, `server/routes/`, `server/middleware/`. This is where secrets, database access, and third-party API keys belong — never in app code that ships to the browser.
- **Build-time:** `nuxt.config.ts` and modules run in Node at build/dev time only.

See [rendering-and-data](references/rendering-and-data.md#execution-contexts) for the full breakdown.

### 2. Fetching data → use the composable, not a manual fetch

- Fetching from your own Nitro server route or an external API during component setup → `useFetch` (thin wrapper) or `useAsyncData` (when you need to transform, combine multiple sources, or call `$fetch` with custom logic).
- Both **must** be called at the top level of `<script setup>` (or a composable called from there), not inside `onMounted`, a watcher body, or an event handler — that's what makes SSR + client hydration share one result instead of fetching twice.
- Give every call a stable, unique `key` when the URL alone doesn't disambiguate it (e.g. calls inside `v-for`, or the same endpoint called with different runtime params in ways Nuxt can't auto-derive).
- One-off client-triggered calls (form submit, button click, refetch) → plain `$fetch`, not `useFetch`.
- Never call `useFetch`/`useAsyncData` inside a loop or conditionally — same rule as Vue's composition API hooks.

See [data-fetching](references/data-fetching.md) for the full decision tree (useFetch vs useAsyncData vs $fetch vs server-only fetch), caching (`getCachedData`), and avoiding duplicate/double fetches.

### 3. Rendering mode → routeRules, not a global switch

Since Nuxt 3.9+, prefer per-route `routeRules` in `nuxt.config.ts` over forcing the whole app into one mode:
- Marketing/content pages → `{ prerender: true }` (SSG) or `{ isr: <seconds> }`.
- Authenticated/dashboard pages → SSR (default) or `{ ssr: false }` if there's no SEO/first-paint need.
- Pure client widgets/admin tools → `{ ssr: false }`.

Only reach for `ssr: false` app-wide, or ship a `.client.vue` component, when the content genuinely cannot render on the server (e.g. a library that hard-requires `window`).

See [rendering-and-data](references/rendering-and-data.md#rendering-modes) for the mode comparison and `routeRules` examples.

### 4. Shared state → useState (SSR-safe) or Pinia, never a module-level variable

- Small, page/app-scoped shared state → `useState('key', () => initialValue)`. It's SSR-safe (state created per-request, not leaked across requests) and auto-imported.
- A module-level `let state = ...` or `const store = reactive({...})` declared outside a composable is **not request-isolated** — under SSR it leaks between concurrent users' requests. This is a correctness bug, not just a style issue.
- Larger app state with actions/getters, devtools, or persistence needs → Pinia (`@pinia/nuxt`), not Vuex.

See [state-and-composables](references/state-and-composables.md).

### 5. SEO / head → definePageMeta + useSeoMeta, not manual <head> per page

- Per-page title/meta → `useSeoMeta()` (typed, preferred) or `useHead()` for anything `useSeoMeta` doesn't cover (structured data, link tags).
- Site-wide defaults → `app.head` in `nuxt.config.ts`, overridden per page.
- Static route-level config (layout, middleware, page transitions) → `definePageMeta()`, not runtime logic in the component body.

See [rendering-and-data](references/rendering-and-data.md#seo-and-head).

### 6. Config, secrets, env vars → runtimeConfig, not process.env in app code

- Server-only secrets → `runtimeConfig.<key>` (never exposed to the client).
- Values the client needs → `runtimeConfig.public.<key>`.
- Reading `process.env` directly is fine only inside `nuxt.config.ts` itself; app/component code should read via `useRuntimeConfig()` so it works identically in dev, build, and server-rendered contexts.

See [server-and-config](references/server-and-config.md).

### 7. Images, fonts, scripts → the Nuxt module, not a raw tag

- Images → `@nuxt/image`'s `<NuxtImg>`/`<NuxtPicture>` (automatic `srcset`, lazy loading, format conversion), not a bare `<img>` for content images.
- Fonts → `@nuxt/fonts` for automatic self-hosting/optimization instead of manual `<link>` Google Fonts tags.
- Third-party scripts → `@nuxt/scripts` so they're deferred/optimized instead of a raw `<script>` in `app.head`.

See [performance](references/performance.md).

---

## Critical Anti-Patterns

| Anti-pattern | Use instead | Why | Reference |
|---|---|---|---|
| `fetch()`/`axios` inside `onMounted` for initial page data | `useFetch`/`useAsyncData` at setup top level | Runs only client-side → no SSR content, and causes a flash of empty state; breaks the SSR/hydration data-sharing Nuxt relies on | [data-fetching](references/data-fetching.md) |
| `useFetch` called inside a `v-for`, `if`, or after an `await` | Call at the top level, give each call an explicit unique `key` | Composition API + Nuxt payload matching both require a stable call order/key | [data-fetching #keys-and-loops](references/data-fetching.md#keys-and-loops) |
| Same data fetched again with a second `$fetch` after `useAsyncData` already loaded it | Reuse the `data` ref, or share via `useState`/a composable | Duplicate network round-trips, one server-side and one redundant client-side | [data-fetching #avoiding-duplicate-fetches](references/data-fetching.md#avoiding-duplicate-fetches) |
| `process.env.API_SECRET` read inside a component or composable | `useRuntimeConfig().apiSecret` (server) | `process.env` isn't guaranteed to exist client-side and bypasses Nuxt's public/private split — easy way to leak a secret into the client bundle | [server-and-config](references/server-and-config.md) |
| Calling a third-party API with a secret key directly from the browser/component | Proxy through a `server/api/*.ts` route that holds the key server-side | Any key referenced in client code is visible in the shipped bundle | [server-and-config #proxying-third-party-apis](references/server-and-config.md#proxying-third-party-apis) |
| `window`/`document`/`localStorage` accessed at the top level of a universal component | Guard with `if (import.meta.client)`, or move the logic into a `.client.vue` component or plugin | Throws "window is not defined" during SSR | [rendering-and-data #execution-contexts](references/rendering-and-data.md#execution-contexts) |
| Module-level `let`/`reactive()` used as "global state" | `useState()` (SSR-safe) or Pinia store | Module-level state is shared across concurrent SSR requests — one user's data leaks into another's response | [state-and-composables](references/state-and-composables.md) |
| Manually importing a composable/component from its file path (`import Foo from '~/components/Foo.vue'`) that lives in an auto-import directory | Just use it — Nuxt auto-imports `components/`, `composables/`, and `utils/` | Redundant, drifts from the convention, and can create duplicate instances in edge cases | [project-structure](references/project-structure.md#auto-imports) |
| Whole app forced to `ssr: false` because one page needs client-only rendering | `routeRules` for that route, or a `.client.vue` component | Throws away SSR/SEO benefits for every other route unnecessarily | [rendering-and-data #rendering-modes](references/rendering-and-data.md#rendering-modes) |
| Repeating the same `useHead()` title/meta boilerplate in every page | `useSeoMeta()` + `app.head` defaults in `nuxt.config.ts`, override per page | Typos and drift across pages; `useSeoMeta` is typed and deduplicates correctly | [rendering-and-data #seo-and-head](references/rendering-and-data.md#seo-and-head) |
| `<img src="...">` for content/hero images | `<NuxtImg>`/`<NuxtPicture>` from `@nuxt/image` | No automatic responsive `srcset`, lazy-loading, or format negotiation (AVIF/WebP) | [performance](references/performance.md) |
| Business logic that talks to a database or filesystem placed in a component/composable | Move it to `server/api/*.ts` (Nitro) and call it via `$fetch`/`useFetch` | Component/composable code ships to the browser bundle; DB drivers and Node-only APIs don't belong there | [server-and-config](references/server-and-config.md) |
| New page/component added without checking `app.vue` / layout / middleware structure first | Check existing `layouts/`, `middleware/`, and `pages/` conventions before adding a new one | Nuxt resolves layouts and middleware by filename convention — an inconsistent name silently fails to apply | [project-structure](references/project-structure.md) |
| Global npm packages installed for things Nuxt modules already cover (e.g. manual sitemap generation, manual PWA setup) | Check the official Nuxt Modules list (`nuxt.com/modules`) first | Official modules integrate with SSR/build/dev correctly out of the box; hand-rolled equivalents often miss SSR edge cases | [project-structure](references/project-structure.md#modules) |

---

## Reference Files

Read these when you need detailed information:

| File | When to read |
|---|---|
| [data-fetching](references/data-fetching.md) | useFetch vs useAsyncData vs $fetch vs server-only; keys, avoiding duplicate/double fetches, caching, error handling, refresh/watch, pagination |
| [rendering-and-data](references/rendering-and-data.md) | Execution contexts (universal/server/build-time), rendering modes (SSR/SSG/ISR/CSR/hybrid), routeRules, SEO/useSeoMeta/useHead, definePageMeta |
| [state-and-composables](references/state-and-composables.md) | useState vs Pinia vs plain composables, composable design conventions, sharing state across components without prop drilling |
| [server-and-config](references/server-and-config.md) | Nitro server routes/middleware, runtimeConfig vs process.env, proxying third-party APIs, environment variables per deploy target |
| [performance](references/performance.md) | @nuxt/image, @nuxt/fonts, @nuxt/scripts, lazy hydration components, bundle size, prerendering large sites |
| [project-structure](references/project-structure.md) | Directory conventions, auto-imports, layers, official Nuxt Modules ecosystem, choosing modules vs hand-rolled code |
