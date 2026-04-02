# Svelte 5 Skills

A collection of skills that improve how AI coding agents scaffold, review, and architect **Svelte 5** and **SvelteKit** applications. Instead of generating outdated patterns, the AI follows modern Svelte 5 best practices with runes-based reactivity, TypeScript strict mode, and Tailwind CSS v4.

[![Agent Skills](https://img.shields.io/badge/Agent_Skills-Compatible-blue?style=flat-square)](https://github.com/vercel-labs/agent-skills)
[![GitHub stars](https://img.shields.io/github/stars/OliveiraCleidson/svelte-5-skills?style=flat-square&color=yellow)](https://github.com/OliveiraCleidson/svelte-5-skills/stargazers)
[![AI Supported](https://img.shields.io/badge/AI_Supported-Cursor_%7C_Claude_%7C_Antigravity-black?style=flat-square)](#)
[![Svelte 5](https://img.shields.io/badge/Svelte-5_%7C_Runes-FF3E00?style=flat-square&logo=svelte&logoColor=white)](https://svelte.dev)
[![TypeScript](https://img.shields.io/badge/TypeScript-Strict-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://www.typescriptlang.org)

## Installing

Works via CLI for all major AI coding agents (Cursor, Antigravity, Claude Code, Codex, Windsurf, Copilot, etc.):

```bash
npx skills add https://github.com/OliveiraCleidson/svelte-5-skills
```

## Skills

| Skill | Description |
| --- | --- |
| **svelte5-init** | Scaffolds production-ready Svelte 5 + SvelteKit projects. Covers 6 deployment targets (Cloudflare, Vercel, Netlify, Node, Static, SPA), Tailwind CSS v4, and full project structure. |
| **svelte5-review** | Expert code review for Svelte 5 components. Validates rune correctness, template syntax, state management, cohesion, TypeScript safety, and performance. |
| **sveltekit-review** | Architectural review for SvelteKit apps. Covers routing, load functions, form actions, rendering modes, hooks, security, performance, and accessibility. |

## Skill Details

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

## Common Questions

**Does it work with all AI coding agents?**
Yes. These skills use the [Agent Skills](https://github.com/vercel-labs/agent-skills) standard and work across Cursor, Claude Code, Antigravity, Codex, Windsurf, Copilot, and any agent that supports SKILL.md files.

**What is a SKILL.md file?**
A portable instruction file that AI coding agents detect and follow automatically. No configuration is needed — just install it and your agent reads it.

**Why separate skills instead of one big file?**
Each skill has a focused scope. You can install only what you need: scaffolding for new projects, component review for code quality, or architecture review for full apps.

## Feedback & Contributions

I'd love to hear your thoughts! If you have suggestions or find any bugs:

- Open a Pull Request or Issue right here on [GitHub](https://github.com/OliveiraCleidson/svelte-5-skills)

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
