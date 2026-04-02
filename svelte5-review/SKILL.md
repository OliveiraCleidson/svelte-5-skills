---
name: svelte5-review
description: Reviews Svelte 5 code for correct rune usage, reactivity patterns, component architecture, cohesion, decoupling, and TypeScript best practices. Use when the user asks to review, audit, lint, or check Svelte 5 components, runes, state management, or code quality.
---

# Svelte 5 Code Review

Expert review of Svelte 5 code focusing on correct rune usage, reactivity patterns, component cohesion, decoupling, and type safety.

## Review workflow

1. Read all files the user provides (or scan `src/` for `.svelte`, `.svelte.ts`, `.svelte.js`, `.ts` files)
2. Evaluate each file against the checklist categories below
3. Report findings grouped by severity: **Error** (broken/incorrect), **Warning** (suboptimal), **Suggestion** (improvement)
4. For each finding, show the problematic code and the corrected version

## Checklist

### 1. Rune correctness

**$state**
- Only used for values that must trigger UI updates
- Objects/arrays that are only reassigned (never mutated) use `$state.raw` instead
- No destructuring of `$state` objects expecting reactivity (breaks the proxy link)
- Class fields use `$state` correctly with arrow functions or proper method binding
- `$state.snapshot()` used when passing proxied state to external libraries (structuredClone, postMessage, third-party libs)
- `$state.eager()` used only for immediate UI feedback (navigation indicators, active states)

**$derived**
- Used instead of `$effect` for any computed value
- Expression is pure (no side effects inside `$derived`)
- `$derived.by(() => {...})` used for multi-statement logic
- Props-dependent values always wrapped in `$derived`, never computed once at init:

```svelte
<!-- WRONG -->
<script lang="ts">
  let { type } = $props();
  let color = type === 'error' ? 'red' : 'blue'; // computed once, never updates
</script>

<!-- CORRECT -->
<script lang="ts">
  let { type } = $props();
  let color = $derived(type === 'error' ? 'red' : 'blue');
</script>
```

**$effect**
- Treated as an escape hatch, not normal control flow
- Never used to synchronize state (use `$derived` instead):

```svelte
<!-- WRONG: effect setting state -->
<script lang="ts">
  let count = $state(0);
  let doubled = $state(0);
  $effect(() => { doubled = count * 2; });
</script>

<!-- CORRECT: derived value -->
<script lang="ts">
  let count = $state(0);
  let doubled = $derived(count * 2);
</script>
```

- Returns cleanup functions when creating subscriptions, intervals, or event listeners
- `$effect.pre` used only when DOM measurement before update is needed
- No `if (browser)` wrapping (effects already skip SSR)

**$props**
- Always destructured with type annotations
- Fallback values provided for optional props
- `$props.id()` used for accessibility IDs instead of manual ID generation
- Rest props (`...rest`) used to forward unknown attributes
- Props not mutated unless marked with `$bindable`

**$bindable**
- Used sparingly and only when two-way data flow is truly needed
- Parent uses `bind:` directive when consuming bindable props
- Not used as a shortcut to avoid proper event callbacks

**$inspect**
- Only present in development code, never in production paths
- Used instead of `console.log` for reactive debugging
- `$inspect.trace()` used as first statement in functions to trace re-execution

**$host**
- Only used inside components compiled as custom elements (`<svelte:options customElement="...">`)

### 2. Template syntax

**Snippets and rendering**
- `{#snippet}` + `{@render}` used instead of `<slot>` (legacy)
- Snippet props typed with `Snippet<[ParamTypes]>` from `'svelte'`
- Optional snippets rendered with `{@render snippet?.()}`
- Reusable markup extracted into snippets to eliminate duplication

**Attachments**
- `{@attach fn}` used instead of `use:action` for DOM integration
- Attachment factories used for reactive parameters
- Cleanup functions returned from attachments

**Event handlers**
- `onclick={handler}` syntax used instead of `on:click={handler}` (legacy)
- Inline handlers for simple operations: `onclick={() => count++}`
- Named functions for complex handlers

**Each blocks**
- Keyed with unique identifiers: `{#each items as item (item.id)}`
- Never keyed by index when items can reorder
- Destructuring used in each blocks when accessing multiple properties

**Dynamic components**
- Direct `<Component>` usage instead of `<svelte:component this={Component}>` (legacy)

### 3. State management and architecture

**Component-level state**
- State declared at the top of `<script>` block
- Related state grouped together
- Derived values placed after the state they depend on

**Shared state**
- Shared reactive state lives in `.svelte.ts` or `.svelte.js` files (not plain `.ts`)
- Uses exported functions or classes with `$state` fields:

```ts
// counter.svelte.ts
let count = $state(0);

export function increment() { count++; }
export function decrement() { count--; }
export function getCount() { return count; }
```

Or with classes:

```ts
// counter.svelte.ts
export class Counter {
  count = $state(0);
  doubled = $derived(this.count * 2);

  increment() { this.count++; }
  decrement() { this.count--; }
}
```

- No duplicated state across components (centralize in shared modules)
- No global module-level state in SSR apps (use context API instead)

**Context API**
- `createContext<T>()` used for type-safe context (Svelte 5.40+)
- Context used to avoid prop drilling through intermediate components
- Reactive state passed via context uses `$state` objects (mutate, don't reassign)
- Context values not reassigned after creation (breaks the reference link)

**Stores**
- Avoided in new code (use runes instead)
- If present, flagged for migration to `$state` / `$derived` / context

### 4. Cohesion and decoupling

**Single responsibility**
- Each component has one clear purpose
- Components under 200 lines (extract sub-components if larger)
- Logic separated from presentation where complex

**Separation of concerns**
- Business logic in `.svelte.ts` files, not inline in components
- API calls abstracted into service modules in `$lib/`
- Types defined in dedicated `.ts` files or co-located with their module
- Server-only code strictly in `$lib/server/` or `+page.server.ts`

**Reusability**
- Shared components in `$lib/components/`
- Shared state in `$lib/stores/` (`.svelte.ts` files)
- Shared utilities in `$lib/utils/`
- No circular dependencies between modules

**Props interface**
- Components accept the minimum props needed (interface segregation)
- Complex data transformed before passing to children
- Callback props preferred over two-way binding for actions

### 5. TypeScript and type safety

- All component props typed (inline or interface)
- Generic components use proper generics with `generics` attribute
- Event handler types use Svelte's built-in types
- Return types on exported functions
- `satisfies` used for type narrowing where appropriate
- No `any` without explicit justification
- `$lib` alias used instead of deep relative paths

### 6. Performance

- `$state.raw` for large, immutable data structures (API responses, config)
- `$state.snapshot()` for serialization boundaries
- Keyed `{#each}` blocks for lists that reorder
- Lazy imports for heavy components: `const Component = await import('./Heavy.svelte')`
- No unnecessary reactivity (static values should not use `$state`)

## Report format

```
## Svelte 5 Code Review

### Errors (must fix)
- [FILE:LINE] Description
  Before: `code`
  After: `corrected code`

### Warnings (should fix)
- [FILE:LINE] Description
  Before: `code`
  After: `corrected code`

### Suggestions (nice to have)
- [FILE:LINE] Description
  Recommendation: explanation

### Summary
- X errors, Y warnings, Z suggestions
- Overall assessment: [score/description]
```
