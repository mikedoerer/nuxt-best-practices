# Server Routes (Nitro) and Configuration

## When code belongs in server/

Anything that needs a secret, talks to a database, or uses a Node-only API (filesystem, native modules) belongs under `server/`, never in `pages/`, `components/`, or `composables/` — that code is bundled and shipped to the browser (even the "server-rendered" parts run again client-side during hydration for universal components).

```
server/
├── api/
│   ├── items.get.ts       # GET /api/items
│   ├── items.post.ts      # POST /api/items
│   └── items/[id].get.ts  # GET /api/items/:id
├── routes/                 # non-/api routes, e.g. server/routes/sitemap.xml.ts
└── middleware/              # runs on every request, e.g. auth checks, logging
```

The `.get`/`.post`/`.put`/`.delete` suffix in the filename maps to the HTTP method — no manual method-checking needed for the common case.

## runtimeConfig vs process.env

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  runtimeConfig: {
    apiSecret: '',       // server-only, override via NUXT_API_SECRET env var
    public: {
      apiBase: '',        // exposed to client, override via NUXT_PUBLIC_API_BASE
    },
  },
})
```
- Server code: `const config = useRuntimeConfig(event)` inside a `server/api` handler → `config.apiSecret` is available; `config.public.apiBase` is also available.
- Client/universal code: `const config = useRuntimeConfig()` → only `config.public.*` is populated; the private keys are stripped from the client bundle.
- Values are overridden by matching `NUXT_`-prefixed env vars at runtime without a rebuild — this is the mechanism for per-environment secrets (dev/staging/prod), not hardcoding different values per environment in the config file itself.
- Reading `process.env.X` directly in app code (not `nuxt.config.ts`) skips this system entirely and risks the value being undefined or, worse, statically inlined into the client bundle depending on build tooling — always go through `runtimeConfig`.

## Proxying third-party APIs

Never call a third-party API that requires a secret key directly from a component/composable — the key would ship in the client bundle. Instead:

```ts
// server/api/weather.get.ts
export default defineEventHandler(async (event) => {
  const config = useRuntimeConfig(event)
  return await $fetch('https://api.weatherprovider.com/v1/current', {
    query: getQuery(event),
    headers: { Authorization: `Bearer ${config.weatherApiKey}` },
  })
})
```
```ts
// component
const { data } = await useFetch('/api/weather', { query: { city } })
```

## Server middleware

`server/middleware/*.ts` runs on every matching request before route handlers — typical uses: auth/session validation, request logging, setting response headers. Keep it lightweight; expensive per-request work belongs in the specific route handler that needs it, not global middleware that runs for every request including static assets.

## Validating input

Use `readValidatedBody`/`getValidatedQuery` (from `h3`, re-exported by Nitro) with a schema (e.g. Zod) rather than trusting `readBody`/`getQuery` output unchecked — especially for any route that writes to a database.
