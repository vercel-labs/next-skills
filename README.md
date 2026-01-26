# Next.js Agent Skills

Agent skills for common Next.js workflows.

## Essential Skills

Start here. These background skills are auto-applied to prevent common mistakes.

### `next-fundamentals`

Core Next.js knowledge:

- [File Conventions](./skills/next-fundamentals/file-conventions.md) - Project structure and special files
- [RSC Boundaries](./skills/next-fundamentals/rsc-boundaries.md) - Server/Client Component rules
- [Data Patterns](./skills/next-fundamentals/data-patterns.md) - Fetching and mutation strategies
- [Async Patterns](./skills/next-fundamentals/async-patterns.md) - Next.js 15+ async APIs
- [Directives](./skills/next-fundamentals/directives.md) - `'use client'`, `'use server'`, `'use cache'`
- [Error Handling](./skills/next-fundamentals/error-handling.md) - Error boundaries and recovery
- [Route Handlers](./skills/next-fundamentals/route-handlers.md) - API routes with `route.ts`
- [Metadata](./skills/next-fundamentals/metadata.md) - SEO and OG images
- [Image](./skills/next-fundamentals/image.md) - `next/image` optimization
- [Font](./skills/next-fundamentals/font.md) - `next/font` optimization
- [Bundling](./skills/next-fundamentals/bundling.md) - Package compatibility
- [Hydration Errors](./skills/next-fundamentals/hydration-error.md) - Debugging mismatches

### `next-caching`

Understanding Next.js caching:

- [Caching](./skills/next-caching/SKILL.md) - 4 cache layers, fetch options, revalidation, v14→v15 changes

## Installation

```bash
# Install essentials (recommended)
npx skills add vercel-labs/next-skills --skill next-fundamentals --skill next-caching

# Or install everything
npx skills add vercel-labs/next-skills
```

## Advanced Use Cases

Optional skills for specific needs. Install as needed and invoke via slash commands.

### `next-upgrade`

Upgrading between Next.js versions with official migration guides.

```bash
npx skills add vercel-labs/next-skills --skill next-upgrade
```

### `next-debug-tricks`

Speed up debugging with MCP endpoint and selective route rebuilding.

```bash
npx skills add vercel-labs/next-skills --skill next-debug-tricks
```

### `next-parallel-routes`

Implement modal patterns with parallel and intercepting routes. Covers `default.tsx`, route matchers (`(.)`, `(..)`, `(...)`), and closing modals correctly.

```bash
npx skills add vercel-labs/next-skills --skill next-parallel-routes
```

### `next-self-hosting`

Deploy Next.js without Vercel. Covers `output: 'standalone'`, Docker, PM2, cache handlers for multi-instance ISR, and what needs extra setup.

```bash
npx skills add vercel-labs/next-skills --skill next-self-hosting
```

### `next-cache-components`

Next.js 16 Cache Components and PPR. Covers `cacheComponents: true`, `'use cache'` directive, cache profiles (`remote`, `private`), `cacheLife()`, `cacheTag()`, and `updateTag()`.

```bash
npx skills add vercel-labs/next-skills --skill next-cache-components
```

## Usage

**Background skills** (`next-fundamentals`, `next-caching`) are automatically applied when relevant.

**Slash commands** for advanced skills:

```
/next-upgrade
/next-debug-tricks
/next-parallel-routes
/next-self-hosting
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
