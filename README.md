# nuxt-best-practices

A [Claude Code](https://claude.com/claude-code) plugin whose skill teaches Claude to
follow **Nuxt 3/4 best practices** — data fetching, rendering modes, Nitro server
routes, shared state, SEO/head, and config/module setup.

The core idea: prefer Nuxt's built-in conventions (`useFetch`/`useAsyncData`,
`routeRules`, `useState`, Nitro `server/api`, `runtimeConfig`, official modules)
over hand-rolled equivalents that re-introduce hydration mismatches, duplicate
requests, and leaked secrets.

## What it covers

| Topic | Reference |
|---|---|
| useFetch vs useAsyncData vs $fetch, keys, duplicate/double fetches, caching, pagination | [`data-fetching.md`](skills/nuxt-best-practices/references/data-fetching.md) |
| Execution contexts, SSR/SSG/ISR/CSR, `routeRules`, `useSeoMeta`/`useHead`, `definePageMeta` | [`rendering-and-data.md`](skills/nuxt-best-practices/references/rendering-and-data.md) |
| `useState` vs Pinia vs plain composables, sharing state without prop drilling | [`state-and-composables.md`](skills/nuxt-best-practices/references/state-and-composables.md) |
| Nitro routes/middleware, `runtimeConfig` vs `process.env`, proxying third-party APIs | [`server-and-config.md`](skills/nuxt-best-practices/references/server-and-config.md) |
| `@nuxt/image`, `@nuxt/fonts`, `@nuxt/scripts`, lazy hydration, bundle size | [`performance.md`](skills/nuxt-best-practices/references/performance.md) |
| Directory conventions, auto-imports, layers, choosing modules vs hand-rolled code | [`project-structure.md`](skills/nuxt-best-practices/references/project-structure.md) |

[`SKILL.md`](skills/nuxt-best-practices/SKILL.md) contains the decision workflow and a table of critical anti-patterns.

## Installation

This repo is a Claude Code plugin marketplace. Add it and install the plugin:

```bash
/plugin marketplace add mikedoerer/nuxt-best-practices
```

```bash
/plugin install nuxt-best-practices@mikedoerer
```

Run `/reload-plugins` afterwards if prompted. Update later with `/plugin marketplace update mikedoerer`.

### Without the marketplace

Clone the skill directly into your Claude Code skills directory:

```bash
git clone https://github.com/mikedoerer/nuxt-best-practices.git /tmp/nbp && cp -r /tmp/nbp/skills/nuxt-best-practices ~/.claude/skills/
```

## Usage

Once installed, Claude invokes the skill on its own when you work on Nuxt code —
creating pages/components/composables, fetching data, choosing a rendering mode,
wiring up server routes, or touching `nuxt.config.ts`. You can also trigger it
explicitly with `/nuxt-best-practices:nuxt-best-practices`.

## License

[MIT](LICENSE)
