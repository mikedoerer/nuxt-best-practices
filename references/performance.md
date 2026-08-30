# Performance

## Images

Use `@nuxt/image`'s `<NuxtImg>` / `<NuxtPicture>` instead of a bare `<img>` for any content/hero image:
```vue
<NuxtImg src="/hero.jpg" width="1200" height="600" format="webp" loading="lazy" />
```
This gets automatic responsive `srcset` generation, lazy loading, and format negotiation (AVIF/WebP with fallback) without hand-writing multiple `<source>` tags. Configure the image provider (local, Cloudinary, ipx, etc.) in `nuxt.config.ts` under `image:`.

Plain `<img>` remains fine for tiny fixed-size icons/logos where responsive variants add no value.

## Fonts

`@nuxt/fonts` auto-detects font usage in CSS and self-hosts Google/custom fonts with correct `font-display` and preload hints — prefer it over manually linking to `fonts.googleapis.com`, which adds an extra render-blocking third-party request.

## Third-party scripts

`@nuxt/scripts` provides optimized wrappers (`useScriptGoogleAnalytics`, `useScriptStripe`, etc.) that lazy-load and defer third-party scripts appropriately instead of a raw `<script src="...">` dropped in `app.head`, which blocks or competes with the main bundle.

## Lazy hydration (Nuxt 3.13+)

For components that are heavy but not needed immediately (below-the-fold widgets, modals, tabs not yet active), use lazy hydration strategies instead of eagerly hydrating everything on load:
```vue
<LazyMyHeavyWidget hydrate-on-visible />
```
Available strategies: `hydrate-on-visible`, `hydrate-on-idle`, `hydrate-on-interaction`, `hydrate-on-media-query`, `hydrate-after="<ms>"`. Reach for these before reaching for manual `IntersectionObserver`/`v-if` hacks to defer hydration.

## Bundle size

- Check `nuxi analyze` before assuming a bundle-size problem needs manual code-splitting — Nuxt/Vite already code-splits per route by default.
- Prefer named imports from a library over importing the whole package when the library supports tree-shaking (e.g. `import { debounce } from 'lodash-es'`, not `import _ from 'lodash'`).
- For a large prerendered site (thousands of static pages), use `nuxi generate` with `crawlLinks`/an explicit route list, and consider `nitro.prerender.concurrency` tuning rather than letting a naive crawl run unbounded.

## Payload size

Data returned from `useAsyncData`/`useFetch` is serialized into the SSR payload sent to the client. Don't return more than the page needs (e.g. select only the fields used) — a composable that fetches an entire large object when the page renders three fields inflates every page load.
