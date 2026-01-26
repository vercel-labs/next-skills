# Next.js Agent Skills

Agent skills for common Next.js workflows.

## Essential Skills

Start here. These background skills are auto-applied to prevent common mistakes.

### `next-best-practices`

Core Next.js knowledge:

- [File Conventions](./skills/next-best-practices/file-conventions.md) - Project structure and special files
- [RSC Boundaries](./skills/next-best-practices/rsc-boundaries.md) - Server/Client Component rules
- [Data Patterns](./skills/next-best-practices/data-patterns.md) - Fetching and mutation strategies
- [Async Patterns](./skills/next-best-practices/async-patterns.md) - Next.js 15+ async APIs
- [Directives](./skills/next-best-practices/directives.md) - `'use client'`, `'use server'`, `'use cache'`
- [Functions](./skills/next-best-practices/functions.md) - Navigation hooks, server functions, generate functions
- [Runtime Selection](./skills/next-best-practices/runtime-selection.md) - Node.js vs Edge runtime
- [Error Handling](./skills/next-best-practices/error-handling.md) - Error boundaries and redirects
- [Route Handlers](./skills/next-best-practices/route-handlers.md) - API routes with `route.ts`
- [Metadata](./skills/next-best-practices/metadata.md) - SEO, OG images, sitemaps
- [Image](./skills/next-best-practices/image.md) - `next/image` optimization
- [Font](./skills/next-best-practices/font.md) - `next/font` optimization
- [Bundling](./skills/next-best-practices/bundling.md) - Package compatibility, CSS imports, polyfills
- [Scripts](./skills/next-best-practices/scripts.md) - Third-party scripts, Google Analytics
- [Hydration Errors](./skills/next-best-practices/hydration-error.md) - Debugging mismatches
- [Suspense Boundaries](./skills/next-best-practices/suspense-boundaries.md) - CSR bailout, Cache Components requirements
- [Parallel Routes](./skills/next-best-practices/parallel-routes.md) - Modal patterns with intercepting routes
- [Self-Hosting](./skills/next-best-practices/self-hosting.md) - Docker, standalone output, ISR
- [Debug Tricks](./skills/next-best-practices/debug-tricks.md) - MCP endpoint, rebuild specific routes

## Installation

```bash
# Install essentials (recommended)
npx skills add vercel-labs/next-skills --skill next-best-practices

# Or install everything
npx skills add vercel-labs/next-skills
```

## Advanced Use Cases

Optional skills for specific needs. Invoke via slash commands.

### `next-upgrade`

Upgrading between Next.js versions with official migration guides.

```bash
npx skills add vercel-labs/next-skills --skill next-upgrade
```

### `next-cache-components`

Next.js 16 Cache Components and PPR. Covers `cacheComponents: true`, `'use cache'` directive, cache profiles, `cacheLife()`, `cacheTag()`, and `updateTag()`.

```bash
npx skills add vercel-labs/next-skills --skill next-cache-components
```

## Usage

**Background skills** (`next-best-practices`) are automatically applied when relevant.

**Slash commands** for advanced skills:

```
/next-upgrade
/next-cache-components
```

## Related Skills

For React-specific patterns (hooks, state management, component composition):

```bash
npx skills add vercel-labs/agent-skills --skill react-best-practices
```

## Contributing

Each skill follows the [Agent Skills open standard](https://github.com/anthropics/skills):

1. Create a directory under `skills/` with the skill name (prefix with `next-`)
2. Add a `SKILL.md` file with YAML frontmatter:
   ```yaml
   ---
   name: next-skill-name
   description: Brief description
   user-invocable: false  # for background skills
   ---
   ```
3. For complex skills, add additional `.md` files and reference them from `SKILL.md`
