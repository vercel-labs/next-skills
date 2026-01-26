# Next.js Agent Skills

Agent skills for common Next.js workflows.

## Essential Skills

Start here. These background skills are auto-applied to prevent common mistakes.

### `next-fundamentals`

Core Next.js knowledge covering:

| Topic | What You'll Learn |
|-------|-------------------|
| File Conventions | Project structure, special files (`page.tsx`, `layout.tsx`, `loading.tsx`), route segments, `middleware.ts` → `proxy.ts` (v16) |
| RSC Boundaries | Server/Client Component rules, serialization constraints, `'use client'` placement |
| Data Patterns | Server Components vs Server Actions vs Route Handlers, avoiding waterfalls, `Promise.all`, preload pattern |
| Async APIs | Next.js 15+ async `params`, `searchParams`, `cookies()`, `headers()` |
| Directives | `'use client'`, `'use server'`, `'use cache'` |
| Error Handling | `error.tsx`, `global-error.tsx`, `not-found.tsx`, Server Action error patterns |
| Route Handlers | `route.ts` conventions, GET/POST handlers, when to use vs Server Actions |
| Metadata | Static/dynamic metadata, `generateMetadata`, OG images with `next/og` |
| Image Optimization | `next/image`, remote images, `sizes`, blur placeholders, priority loading |
| Font Optimization | `next/font`, Google Fonts, local fonts, Tailwind integration |
| Bundling | Server-incompatible packages, ESM/CJS issues, Turbopack migration |
| Hydration Errors | Common causes, debugging, fixes |

### `next-caching`

Understanding Next.js caching:

| Topic | What You'll Learn |
|-------|-------------------|
| 4 Cache Layers | Request Memoization, Data Cache, Full Route Cache, Router Cache |
| fetch() Options | `cache`, `next.revalidate`, `next.tags` |
| Revalidation | `revalidatePath()` vs `revalidateTag()`, when to use each |
| v14 → v15 Changes | Default behavior shift from cached to uncached |
| Common Gotchas | Router Cache minimum, third-party libraries, stale-while-revalidate |

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
