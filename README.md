# Svelte 5 Skills for Claude Code

[![Svelte](https://img.shields.io/badge/Svelte-5-FF3E00?style=flat&logo=svelte&logoColor=white)](https://svelte.dev)
[![SvelteKit](https://img.shields.io/badge/SvelteKit-2-FF3E00?style=flat&logo=svelte&logoColor=white)](https://kit.svelte.dev)
[![TypeScript](https://img.shields.io/badge/TypeScript-Strict-3178C6?style=flat&logo=typescript&logoColor=white)](https://www.typescriptlang.org)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-v4-06B6D4?style=flat&logo=tailwindcss&logoColor=white)](https://tailwindcss.com)
[![Claude Code](https://img.shields.io/badge/Claude_Code-Skills-D97706?style=flat)](https://docs.anthropic.com/en/docs/claude-code)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat)](LICENSE)

A collection of specialized [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills for scaffolding, reviewing, and architecting **Svelte 5** and **SvelteKit** applications. These skills enforce modern best practices including runes-based reactivity, TypeScript strict mode, and Tailwind CSS v4.

## Skills Overview

| Skill | Purpose | Scope |
|-------|---------|-------|
| [**svelte5-init**](#svelte5-init) | Project scaffolding | New project setup with 6 deployment targets |
| [**svelte5-review**](#svelte5-review) | Component code review | `.svelte` / `.svelte.ts` files |
| [**sveltekit-review**](#sveltekit-review) | Architecture review | Full SvelteKit application |

---

### svelte5-init

Scaffolds production-ready Svelte 5 + SvelteKit projects using the `sv` CLI. Configures the full stack out of the box:

- **Deployment Targets** — Cloudflare, Vercel, Netlify, Node (with Docker), Static (SSG), and SPA
- **Styling** — Tailwind CSS v4 with Vite plugin
- **TypeScript** — Strict mode with proper `App` namespace declarations
- **Project Structure** — `$lib/server/`, `$lib/components/`, `$lib/stores/`, `$lib/utils/`
- **Svelte 5 Conventions** — Runes (`$state`, `$derived`, `$effect`, `$props`), snippets, attachments

### svelte5-review

Expert-level code review focused on Svelte 5 component correctness and quality:

- **Rune Correctness** — Validates proper usage of `$state`, `$derived`, `$effect`, `$props`, `$bindable`
- **Template Syntax** — Enforces `{#snippet}` + `{@render}`, `{@attach}`, and `onclick={}` patterns
- **State Management** — Shared state in `.svelte.ts` files, Context API, no global state in SSR
- **Cohesion & Decoupling** — Single responsibility, components under 200 LOC, no circular deps
- **TypeScript Safety** — Typed props, generic components, no untyped `any`
- **Performance** — `$state.raw` for large data, keyed `{#each}`, lazy imports

### sveltekit-review

Architectural review for SvelteKit applications covering routing, rendering, and server-side patterns:

- **Routing & File Conventions** — Route groups, dynamic params, matchers, preload hints
- **Load Functions** — Server vs. universal load selection, parallel requests, streaming
- **Form Actions** — Progressive enhancement, validation with `fail()`, named actions
- **Rendering Modes** — Hybrid rendering (SSR + SSG), `prerender`, `ssr`, `csr` flags
- **Hooks** — Server/client/universal hooks, `sequence()` composition
- **Security** — Private env isolation, `$lib/server/`, auth cookie flags, input sanitization
- **Performance** — Cache headers, preload hints, no request waterfalls
- **Accessibility & SEO** — Semantic HTML, meta tags, JSON-LD, sitemap

---

## Installation

Clone the repository and install each skill into your Claude Code environment:

```bash
git clone git@github.com:OliveiraCleidson/svelte-5-skills.git
```

Then add the skills to your Claude Code configuration by referencing the `SKILL.md` files in each subdirectory.

## Usage

Once installed, invoke the skills directly in Claude Code:

```
# Scaffold a new Svelte 5 project targeting Vercel
/svelte5-init --deploy vercel

# Review Svelte 5 component code
/svelte5-review

# Review SvelteKit application architecture
/sveltekit-review
```

## Tech Stack

| Category | Technology |
|----------|-----------|
| Framework | Svelte 5 (Runes) |
| Meta-Framework | SvelteKit 2 |
| Language | TypeScript (strict) |
| CSS | Tailwind CSS v4 |
| Build | Vite |
| Package Manager | pnpm (recommended) |
| Node | 18.13+ (22+ recommended) |

## Contributing

Contributions are welcome! If you'd like to improve an existing skill or add a new one:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-skill`)
3. Commit your changes
4. Push to the branch (`git push origin feature/my-skill`)
5. Open a Pull Request

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
