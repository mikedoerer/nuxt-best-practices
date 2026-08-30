# Rendering Modes, Execution Contexts, and SEO

## Execution contexts

| Context | Where it runs | Notes |
|---|---|---|
| Universal | Server during SSR, then again in the browser during hydration | Default for `<script setup>` in pages/components/layouts. Code must work in *both* environments — no direct `window`/`document`/`localStorage` without a guard. |
| Client-only | Browser only | `.client.vue` component suffix, or code inside `if (import.meta.client)` / `onMounted` (which never runs during SSR). Also true for browser-only plugins (`.client.ts` plugin filename). |
| Server-only | Node, during SSR render or a Nitro request | `.server.vue` component suffix, `server/` directory (API routes, middleware), `.server.ts` plugins. Secrets and DB access belong here. |
| Build-time | Node, at `nuxt build`/`nuxt dev` startup | `nuxt.config.ts`, modules. `process.env` is fine to read here directly. |

Guard pattern for occasional browser-only logic inside an otherwise-universal component:
```ts
if (import.meta.client) {
  // safe: window, document, localStorage
}
```
(`import.meta.client` / `import.meta.server` are the modern equivalents of the older `process.client` / `process.server`; both work in current Nuxt, prefer `import.meta.*`.)

## Rendering modes

| Mode | `routeRules` | Best for |
|---|---|---|
| SSR (default) | none needed | Pages needing fresh per-request data + SEO + fast first paint |
| SSG / prerender | `{ prerender: true }` | Marketing pages, docs, blog posts — content that's the same for everyone and rarely changes |
| ISR (Incremental Static Regeneration) | `{ isr: <seconds> }` or `{ isr: true }` | High-traffic content pages that change occasionally — served from cache, regenerated in the background |
| CSR / SPA for one route | `{ ssr: false }` on that route | Admin/dashboard-style pages with no SEO need where server rendering adds no value |
| Full SPA (whole app) | `ssr: false` in `nuxt.config.ts` | Rare — only when the entire app is behind auth with zero public/SEO surface |

Example hybrid config:
```ts
// nuxt.config.ts
export default defineNuxtConfig({
  routeRules: {
    '/': { prerender: true },
    '/blog/**': { isr: 3600 },
    '/dashboard/**': { ssr: false },
    '/api/**': { cors: true },
  },
})
```

Prefer this hybrid setup over a single global mode — it's the main reason `routeRules` exists.

## SEO and head

- `useSeoMeta()` — typed, preferred for standard tags (`title`, `description`, `ogTitle`, `ogImage`, `twitterCard`, etc.). Catches typos at compile time that raw `useHead` meta arrays don't.
- `useHead()` — for anything `useSeoMeta` doesn't cover: `link` tags, structured data (`script[type="application/ld+json"]`), custom meta not in the SEO set.
- `app.head` in `nuxt.config.ts` — site-wide defaults (charset, viewport, default title template, favicon). Per-page `useSeoMeta`/`useHead` calls override/extend these.
- `definePageMeta()` — static, compile-time page config: `layout`, `middleware`, `pageTransition`, `keepalive`. Not for dynamic meta content — that's `useSeoMeta`/`useHead`.

Example:
```ts
// pages/blog/[slug].vue
definePageMeta({ layout: 'article' })

const { data: post } = await useAsyncData(`post-${route.params.slug}`, () =>
  queryContent(route.params.slug as string).findOne()
)

useSeoMeta({
  title: () => post.value?.title,
  description: () => post.value?.description,
  ogImage: () => post.value?.image,
})
```

For sitemaps, robots.txt, and OG image generation, use the official `@nuxtjs/sitemap`, `@nuxtjs/robots`, and `nuxt-og-image` modules rather than hand-writing these — they integrate with prerendering and `routeRules` automatically.
