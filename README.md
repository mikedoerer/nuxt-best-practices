# nuxt-best-practices

A [Claude Code](https://claude.com/claude-code) skill that teaches Claude to follow
**Nuxt 3/4 best practices** — data fetching, rendering modes, Nitro server routes,
shared state, SEO/head, and config/module setup.

The core idea: prefer Nuxt's built-in conventions (`useFetch`/`useAsyncData`,
`routeRules`, `useState`, Nitro `server/api`, `runtimeConfig`, official modules)
over hand-rolled equivalents that re-introduce hydration mismatches, duplicate
requests, and leaked secrets.

## What it covers

| Topic | Reference |
|---|---|
| useFetch vs useAsyncData vs $fetch, keys, duplicate/double fetches, caching, pagination | [`references/data-fetching.md`](references/data-fetching.md) |
| Execution contexts, SSR/SSG/ISR/CSR, `routeRules`, `useSeoMeta`/`useHead`, `definePageMeta` | [`references/rendering-and-data.md`](references/rendering-and-data.md) |
| `useState` vs Pinia vs plain composables, sharing state without prop drilling | [`references/state-and-composables.md`](references/state-and-composables.md) |
| Nitro routes/middleware, `runtimeConfig` vs `process.env`, proxying third-party APIs | [`references/server-and-config.md`](references/server-and-config.md) |
| `@nuxt/image`, `@nuxt/fonts`, `@nuxt/scripts`, lazy hydration, bundle size | [`references/performance.md`](references/performance.md) |
| Directory conventions, auto-imports, layers, choosing modules vs hand-rolled code | [`references/project-structure.md`](references/project-structure.md) |

[`SKILL.md`](SKILL.md) contains the decision workflow and a table of critical anti-patterns.

## Installation

Clone into your Claude Code skills directory:

```bash
git clone https://github.com/mikedoerer/nuxt-best-practices.git ~/.claude/skills/nuxt-best-practices
```

For a single project instead of globally:

```bash
git clone https://github.com/mikedoerer/nuxt-best-practices.git .claude/skills/nuxt-best-practices
```

Claude Code picks the skill up automatically on the next session.

## Usage

Once installed, Claude invokes the skill on its own when you work on Nuxt code —
creating pages/components/composables, fetching data, choosing a rendering mode,
wiring up server routes, or touching `nuxt.config.ts`. You can also trigger it
explicitly by asking Claude to "follow the nuxt-best-practices skill".

## License

[MIT](LICENSE)
