# Project Structure, Auto-Imports, and Modules

## Directory conventions

| Directory | Purpose | Auto-imported? |
|---|---|---|
| `pages/` | File-based routing — filename/folder structure maps to URL routes | n/a (routing, not imports) |
| `components/` | Vue components | Yes — used by filename, no `import` needed |
| `composables/` | `useX()` functions | Yes |
| `utils/` | Plain helper functions | Yes |
| `layouts/` | Named layouts, applied via `definePageMeta({ layout: 'name' })` | n/a |
| `middleware/` | Route middleware (named, or `.global.ts` for every route) | Referenced by filename, not imported |
| `plugins/` | App-wide setup code, run once per app instance | Auto-registered by filename convention (`.client.ts`/`.server.ts` suffix controls context) |
| `server/` | Nitro server routes/middleware | See [server-and-config](server-and-config.md) |
| `stores/` | Pinia stores (if using Pinia) | Not auto-imported by default; `defineStore` itself is auto-imported |
| `assets/` | Build-processed static assets (CSS, images referenced in code) | n/a |
| `public/` | Served as-is at the root, unprocessed | n/a |

## Auto-imports

Nuxt auto-imports Vue APIs (`ref`, `computed`, `watch`, ...), Nuxt composables (`useFetch`, `useState`, `useRoute`, ...), and anything in `components/`, `composables/`, and `utils/`. Do **not** manually `import` these — an explicit relative import of something already auto-imported is redundant, drifts from convention, and in rare cases (e.g. components) can create a second registration path that behaves differently from the auto-imported one.

If you're unsure whether something is auto-imported, check `.nuxt/imports.d.ts` (generated, gitignored) rather than guessing.

## Choosing an official module vs hand-rolling

Before writing custom code for something structural — sitemap generation, PWA setup, i18n, image optimization, SEO meta, auth, content management, testing utilities — check the official Nuxt Modules directory (`nuxt.com/modules`) first. Official/community modules are built to integrate correctly with SSR, prerendering, and the dev/build pipeline; equivalents built from generic npm packages often miss Nuxt-specific edge cases (e.g. a sitemap generator that doesn't know about `routeRules`-based prerendering).

Common ones worth checking for before hand-rolling:
- `@nuxt/image` — images (see [performance](performance.md))
- `@nuxt/fonts` — web fonts
- `@nuxt/scripts` — third-party script loading
- `@pinia/nuxt` — state management
- `@nuxtjs/i18n` — internationalization
- `@nuxtjs/sitemap`, `@nuxtjs/robots` — SEO infrastructure
- `@nuxt/content` — file-based/Markdown content
- `@nuxt/test-utils` — testing (see below)

## Layers

For multi-app setups sharing config/components/composables (e.g. a design-system base extended by several apps), use Nuxt Layers (`extends` in `nuxt.config.ts`) instead of copy-pasting shared code or publishing an internal npm package for things that change frequently.

## Testing

Use `@nuxt/test-utils` with your test runner (Vitest is the standard pairing) rather than testing Nuxt components/composables with a plain Vue Test Utils setup — it provides the Nuxt runtime context (auto-imports, `useFetch` mocking helpers, `mountSuspended` for async setup) that bare Vue Test Utils doesn't know about.
