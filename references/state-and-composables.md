# State and Composables

## useState vs Pinia vs plain composable

| Need | Use |
|---|---|
| Small piece of state shared across a few components, no complex mutations | `useState('key', () => initial)` |
| App-wide store with actions, getters, modularized by domain, devtools inspection | Pinia (`@pinia/nuxt`) |
| Pure derived/computed logic with no shared mutable state | A plain composable (`composables/useThing.ts`) returning refs/computed — no `useState` needed if nothing needs to be shared or SSR-persisted |

## Why module-level state is unsafe under SSR

```ts
// ❌ composables/useCart.ts
const cart = reactive({ items: [] }) // module-level — created ONCE per server process
export const useCart = () => cart
```
Under SSR, this object is created once when the module is first imported by the Node process — not once per request. Every concurrent request on that server instance shares the same object, so User A's cart can leak into User B's response. This is invisible in dev (single request at a time) and shows up only under real concurrent traffic.

```ts
// ✅ composables/useCart.ts
export const useCart = () => useState('cart', () => ({ items: [] as CartItem[] }))
```
`useState` is request-scoped on the server (backed by Nuxt's payload/SSR context) and becomes a shared singleton client-side after hydration — the correct behavior in both environments.

## Composable conventions

- Name files `useX.ts` in `composables/` — Nuxt auto-imports anything matching this pattern, no manual import needed.
- A composable should return refs/computed/functions, not a plain object of raw values — callers expect reactivity.
- Composables can call other composables (including `useState`, `useRoute`, `useFetch`) but must do so synchronously at the top of the function body — same top-level-call rule as in components.
- For logic that's genuinely stateless (formatting, validation), a plain function in `utils/` (also auto-imported) is more appropriate than a composable — don't wrap everything in `use*` out of habit.

## Pinia specifics

- Define stores in `stores/*.ts` using `defineStore`; prefer the setup-store syntax (`defineStore('id', () => { ... })`) for consistency with composable style and full type inference.
- Don't destructure reactive state directly off a Pinia store (`const { count } = useCounterStore()`) — this breaks reactivity. Use `storeToRefs()`:
  ```ts
  const store = useCounterStore()
  const { count } = storeToRefs(store)
  ```
- Actions can be destructured directly (they aren't reactive state), only state/getters need `storeToRefs`.
