# Next.js Agent Skills

Agent skills for common Next.js workflows.

## Skills

| Skill | Description |
|-------|-------------|
| `next-upgrade` | Upgrade to the latest Next.js version following official migration guides |
| `next-debug-tricks` | Speed up debugging - MCP endpoint, rebuild specific routes |
| `next-fundamentals` | Background knowledge: RSC boundaries, async patterns, metadata, image/font optimization, bundling, hydration errors (auto-applied) |

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
```

**Background skills** (like `next-fundamentals`) are automatically applied by Claude when relevant - they don't appear in the slash command menu but provide context to prevent common mistakes.

## Structure

```
next-skills/
├── skills/
│   ├── next-upgrade/SKILL.md
│   ├── next-debug-tricks/SKILL.md
│   └── next-fundamentals/
│       ├── SKILL.md              # Entry point
│       ├── rsc-boundaries.md     # RSC serialization rules
│       ├── async-patterns.md     # Next.js 15 async APIs
│       ├── runtime-selection.md  # Node.js vs Edge
│       ├── directives.md         # use cache, use client, use server
│       ├── metadata.md           # Metadata API and OG images
│       ├── image.md              # next/image optimization
│       ├── font.md               # next/font optimization
│       ├── bundling.md           # Package bundling issues
│       └── hydration-error.md    # Hydration mismatch fixes
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
