---
name: sveltekit-review
description: Reviews SvelteKit applications for correct routing, load functions, form actions, rendering modes (SSR/SSG/SPA), hooks, error handling, security, testing, accessibility, and architectural best practices. Use when the user asks to review, audit, or check a SvelteKit project structure, routing, performance, or server-side code.
---

# SvelteKit Code Review

Expert review of SvelteKit applications focusing on routing correctness, rendering strategy, data loading, security, testing, and architectural balance between simplicity, safety, and robustness.

## Review workflow

1. Read the project structure (`src/routes/`, `src/lib/`, `src/hooks.*`, `svelte.config.js`, `vite.config.ts`)
2. Evaluate each area against the checklist categories below
3. Report findings grouped by severity: **Error**, **Warning**, **Suggestion**
4. For each finding, show the problematic pattern and the corrected version

## Checklist

### 1. Routing and file conventions

**File naming**
- Pages use `+page.svelte`, not arbitrary filenames
- Layouts use `+layout.svelte` at the correct hierarchy level
- Server endpoints use `+server.ts` (not `.js` unless the project is JS-only)
- Error boundaries `+error.svelte` placed at strategic levels (root + critical sections)
- Parameter matchers in `src/params/` for dynamic route validation

**Route organization**
- Route groups `(group)` used to share layouts without affecting URLs
- No conflicting routes (e.g., `foo/+page.svelte` and `foo/bar/+page.svelte` without a layout)
- Dynamic routes `[param]` validated with matchers when the param has constraints
- Rest params `[...rest]` have a clear catch-all purpose (404 pages, nested paths)
- Optional params `[[param]]` used correctly

**Navigation**
- Standard `<a>` elements used for links (not custom Link components)
- `data-sveltekit-preload-data="hover"` on `<body>` (default) or tuned per section
- `data-sveltekit-reload` only on links that truly need full-page navigation
- No manual `window.location` assignments for internal navigation (use `goto()`)

### 2. Load functions

**Universal vs server load**
- `+page.server.ts` used when: accessing database, private env vars, cookies, or sensitive logic
- `+page.ts` used when: fetching public APIs, returning non-serializable data (components, functions)
- Both used together when server provides data that universal transforms

**Data loading correctness**
- Load functions are pure (no side effects, no writing to stores or globals)
- `fetch` from the event used (not global `fetch`) for credential forwarding and SSR optimization
- No sequential awaits when requests are independent (parallelize with `Promise.all`):

```ts
// WRONG: waterfall
export async function load({ fetch }) {
  const user = await fetch('/api/user').then(r => r.json());
  const posts = await fetch('/api/posts').then(r => r.json());
  return { user, posts };
}

// CORRECT: parallel
export async function load({ fetch }) {
  const [user, posts] = await Promise.all([
    fetch('/api/user').then(r => r.json()),
    fetch('/api/posts').then(r => r.json())
  ]);
  return { user, posts };
}
```

- `await parent()` used only when truly dependent on parent data (creates waterfalls)
- `depends('app:key')` used for custom invalidation when automatic tracking is insufficient
- `untrack()` used to exclude values from dependency tracking when needed

**Error handling in load**
- `error(status, message)` used for expected errors (not `throw new Error()`)
- `redirect(status, location)` not wrapped in try/catch (it throws intentionally)
- Unexpected errors handled by `handleError` hook, not in load functions

**Streaming**
- Server load returns promises for slow, non-essential data (streaming pattern)
- Essential data awaited; supplementary data returned as promises

### 3. Form actions

**Action correctness**
- `<form method="POST">` with server actions for data mutations
- Named actions used when a page has multiple forms: `action="?/actionName"`
- `fail(status, data)` returns validation errors (never `throw`)
- Sensitive data never returned in action responses (passwords, tokens)
- Previously submitted safe fields returned for form correction UX

**Progressive enhancement**
- `use:enhance` applied to forms for JS-enhanced experience
- Forms still work without JavaScript (graceful degradation)
- Custom `SubmitFunction` used only when default behavior is insufficient
- `applyAction(result)` called in custom enhance when default behavior is wanted

**GET vs POST**
- `GET` (no `method` attribute) for searches and filters (triggers navigation, not action)
- `POST` for mutations (create, update, delete)

### 4. Rendering modes (SSR / SSG / SPA)

**Page options correctness**
- `export const prerender = true` on static pages (marketing, docs, blog)
- `export const prerender = false` on dynamic/personalized pages
- `export const ssr = false` only when browser-only globals are absolutely required
- `export const csr = false` only for fully static pages with zero interactivity
- Options exported from the correct file (`+page.ts`, `+page.server.ts`, or `+layout.ts`)

**Common mistakes**
- Prerendered pages must not use `url.searchParams` in load
- Prerendered pages must not have form actions
- `ssr = false` not used site-wide (kills SEO and performance)
- `csr = false` not combined with forms (forms need JavaScript for enhancement)
- Child pages that override parent rendering options do so intentionally

**Hybrid rendering**
- Static pages prerendered, dynamic pages server-rendered (mixed strategy)
- `entries()` exported for dynamic routes that need prerendering

### 5. Hooks

**Server hooks** (`src/hooks.server.ts`)
- `handle` populates `event.locals` from cookies/session (authentication)
- `handleError` sanitizes errors before sending to client (no stack traces, sensitive data)
- `handleFetch` modifies cross-origin requests when needed (cookie forwarding)
- `init` used for one-time setup (database connections)
- Multiple handlers composed with `sequence()`

**Client hooks** (`src/hooks.client.ts`)
- `handleError` reports errors to monitoring (Sentry, etc.)

**Universal hooks** (`src/hooks.ts`)
- `reroute` used for URL rewriting before route matching
- `transport` used for custom type serialization across server/client boundary

### 6. State management (SvelteKit-specific)

**Critical rule: no shared server state**
- No module-level variables storing user data (shared across all requests):

```ts
// WRONG: shared between users
let user: User;
export async function load() {
  return { user };
}

// CORRECT: per-request via locals
export async function load({ locals }) {
  return { user: locals.user };
}
```

- Per-request state stored in `event.locals` (populated in `handle` hook)
- Persistent user state stored in database, accessed via cookies/sessions

**State location guide**
- URL params: state that affects SSR and should be shareable/bookmarkable
- Cookies: authentication tokens (`httpOnly`, `secure` in production)
- Context API: component-tree state (avoids prop drilling, SSR-safe)
- `$state` in components: local UI state
- `.svelte.ts` modules: shared client-side state (not SSR-safe for user data)
- Database: persistent user data

**Navigation state**
- Component state survives navigation (wrap in `{#key}` to force recreation)
- Snapshots used for ephemeral UI state (scroll position, accordion state)
- `page.state` used for shallow routing (modals, lightboxes with back-button support)

### 7. Error handling and resilience

**Server-side errors**
- `error(status, message)` for expected errors with actionable messages
- `handleError` strips sensitive details before client response
- Custom `App.Error` type includes only safe fields (message, code)
- Stack traces never sent to client

**Client-side error UX**
- Different error types get appropriate treatment:
  - Network errors: offer retry with exponential backoff
  - Validation errors: highlight specific fields, preserve user input
  - Auth errors (401/403): redirect to login
  - Not found (404): show helpful navigation or search
  - Server errors (500): show generic message with support contact
- `+error.svelte` boundaries provide fallback UI, not blank screens
- Loading states shown during async operations
- Optimistic updates with rollback on failure where appropriate

### 8. Security

**Environment variables**
- Secrets use `$env/static/private` or `$env/dynamic/private` (server-only)
- Client-facing config uses `$env/static/public` (`PUBLIC_` prefix)
- No private env vars imported in client code (build error, but verify)
- Static env preferred over dynamic when possible (enables dead code elimination)

**Server-only code**
- Sensitive logic in `$lib/server/` (cannot be imported by client)
- Database queries, API keys, auth logic never in `+page.ts` (universal load)
- Form actions validate and sanitize all input

**Cookies and auth**
- `httpOnly` flag on auth cookies (prevents XSS theft)
- `secure` flag in production
- `event.cookies` API used (not raw `Set-Cookie` headers)
- Cookie checked in `handle` hook, user stored in `locals`

### 9. Project organization

**Domain-driven structure** (recommended for medium-to-large projects)

```
src/
  lib/
    domains/
      auth/          # Components, state, services for auth
      products/      # Components, state, services for products
      checkout/      # Checkout flow
    shared/          # Generic UI (Button, Modal, Toast, etc.)
    server/          # Server-only (DB, auth, API keys)
    utils/           # Pure utility functions
  routes/            # Thin: data passing + composition only
```

- Each domain is self-contained (components, state, types, services)
- Minimal cross-domain imports
- `src/routes/` pages compose domain modules, not implement business logic
- No circular dependencies (enforce via linting or import rules)

**Naming conventions**
- Consistent across the entire codebase
- Components: PascalCase (`UserCard.svelte`)
- Routes: follow SvelteKit conventions (`+page.svelte`, `+layout.svelte`)
- State modules: `.svelte.ts` extension for reactive code
- Server files: `.server.ts` for server-only modules
- Types: PascalCase interfaces/types in dedicated or co-located `.ts` files

### 10. Performance

**Data loading**
- No sequential awaits for independent requests (parallelize)
- `await parent()` avoided when not needed (creates waterfalls)
- Server load preferred over client fetch (avoids client waterfalls, coalesces requests)
- Cache headers set with `setHeaders()` in load functions

**Core Web Vitals targets**
- LCP (Largest Contentful Paint) <= 2.5s
- INP (Interaction to Next Paint) <= 200ms
- CLS (Cumulative Layout Shift) <= 0.1

**Preloading**
- `data-sveltekit-preload-data="hover"` for fast navigation
- `data-sveltekit-preload-code="viewport"` for critical above-fold links
- Heavy routes use `data-sveltekit-preload-code="eager"` only when justified

**Assets**
- Images optimized with modern formats (WebP/AVIF) and explicit width/height
- `@sveltejs/enhanced-img` used when available
- Static assets in `static/` only when hashing is not needed
- Fonts preloaded in `app.html` or via `resolve({ preload })` in hooks

**Bundle size**
- Dynamic imports for heavy components
- Third-party scripts minimized or loaded with Partytown
- Server-side analytics preferred over client-side scripts
- Manual chunk splitting for vendor/UI/feature code in large apps

### 11. Accessibility and SEO

**Accessibility (WCAG 2.2 AA)**
- Every page has a unique, descriptive `<title>` (required for route announcements)
- `lang` attribute set on `<html>` (static or dynamic via `transformPageChunk`)
- Semantic HTML elements used (`<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`)
- Form inputs have associated `<label>` elements
- Interactive elements are keyboard-navigable
- Touch targets at least 24x24 px
- Visible focus indicators with >= 3:1 contrast ratio
- No drag-only interactions without keyboard alternatives
- Color is not the sole means of conveying information
- Focus management considered for dynamic content and route changes

**SEO**
- SSR enabled for content pages (not SPA mode)
- `<meta name="description">` on key pages
- Structured data (JSON-LD) in `<svelte:head>` for rich results
- Sitemap generated via `+server.ts` endpoint
- Canonical URLs when duplicate content exists
- `robots.txt` in `static/`
- Open Graph and Twitter Card meta tags on shareable pages

### 12. Testing

**Strategy**
- Vitest for unit and component tests
- Playwright for E2E and critical user flows
- Test-first approach for server actions (load functions, form actions are pure functions)

**Server-side testing**
- Load functions tested in isolation (they are just async functions)
- Form actions tested with mock `RequestEvent` objects
- Validation logic tested independently from actions

**Component testing**
- Vitest browser mode preferred over jsdom for interactive components
- Svelte 5 components testable as functions returning objects
- Snapshot tests for stable UI components

**E2E testing**
- Playwright for critical user journeys (login, checkout, form submissions)
- Tests run against `preview` build (not dev) for production-like behavior
- Accessibility audits integrated (axe-core via Playwright)

## Report format

```
## SvelteKit Code Review

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

### Architecture assessment
- Routing: [assessment]
- Rendering strategy: [assessment]
- Data flow: [assessment]
- Security: [assessment]
- Performance: [assessment]
- Testing: [assessment]

### Summary
- X errors, Y warnings, Z suggestions
- Overall assessment: [score/description]
```
