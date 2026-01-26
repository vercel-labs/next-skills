# Next.js Agent Skills

Agent skills for common Next.js workflows.

## Skills

| Skill | Description |
|-------|-------------|
| `next-upgrade` | Upgrade to the latest Next.js version following official migration guides |
| `next-debug-tricks` | Speed up debugging - MCP endpoint, rebuild specific routes |
| `next-fundamentals` | Background knowledge: file conventions, RSC boundaries, data patterns, async APIs, metadata, error handling, route handlers, image/font optimization, bundling (auto-applied) |
| `next-caching` | Caching deep dive: 4 cache layers, fetch options, revalidation strategies, v14→v15 behavior changes (auto-applied) |
| `next-parallel-routes` | Parallel and intercepting routes: modal patterns, default.tsx, route matchers |
| `next-self-hosting` | Self-host Next.js without Vercel: standalone output, Docker, cache handlers, ISR |
| `next-cache-components` | Next.js 16 caching: PPR, use cache directive, cache profiles, Cache Components |

## Installation

```bash
# List available skills
npx skills add vercel-labs/next-skills --list

# Install a specific skill
npx skills add vercel-labs/next-skills --skill next-debug-tricks

# Install multiple skills
npx skills add vercel-labs/next-skills --skill next-upgrade --skill next-fundamentals

# Install all skills
npx skills add vercel-labs/next-skills
```

## Related Skills

For React-specific patterns (hooks, state management, component composition), install the React skills:

```bash
npx skills add vercel-labs/agent-skills --skill react-best-practices
```

## Usage

Once installed, invoke skills using slash commands:

```
/next-upgrade
/next-debug-tricks
/next-parallel-routes
/next-self-hosting
/next-cache-components
```

**Background skills** (like `next-fundamentals`, `next-caching`) are automatically applied by Claude when relevant - they don't appear in the slash command menu but provide context to prevent common mistakes.

## Structure

```
next-skills/
├── skills/
│   ├── next-upgrade/SKILL.md
│   ├── next-debug-tricks/SKILL.md
│   ├── next-fundamentals/
│   │   ├── SKILL.md              # Entry point
│   │   ├── file-conventions.md   # Project structure, special files
│   │   ├── rsc-boundaries.md     # RSC serialization rules
│   │   ├── async-patterns.md     # Next.js 15 async APIs
│   │   ├── runtime-selection.md  # Node.js vs Edge
│   │   ├── directives.md         # use cache, use client, use server
│   │   ├── data-patterns.md      # Server Components vs Actions vs Route Handlers
│   │   ├── error-handling.md     # error.tsx, try-catch gotchas
│   │   ├── route-handlers.md     # route.ts best practices
│   │   ├── metadata.md           # Metadata API and OG images
│   │   ├── image.md              # next/image optimization
│   │   ├── font.md               # next/font optimization
│   │   ├── bundling.md           # Package bundling issues
│   │   └── hydration-error.md    # Hydration mismatch fixes
│   ├── next-caching/SKILL.md     # Cache layers, revalidation, v15 changes
│   ├── next-parallel-routes/SKILL.md  # Parallel & intercepting routes
│   ├── next-self-hosting/SKILL.md   # Docker, ISR, cache handlers
│   └── next-cache-components/SKILL.md  # PPR, use cache, cache profiles
└── README.md
```

## Contributing

Each skill follows the [Agent Skills open standard](https://github.com/anthropics/skills):

1. Create a directory under `skills/` with the skill name (prefix with `next-`)
2. Add a `SKILL.md` file with YAML frontmatter:
   ```yaml
   ---
   name: next-skill-name
   description: Brief description (used for discovery and auto-suggestion)
   ---
   ```
3. For complex skills, add additional `.md` files and reference them from `SKILL.md`
