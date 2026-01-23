# Next.js Agent Skills

Agent skills for common Next.js workflows.

## Skills

| Skill | Description |
|-------|-------------|
| `upgrade` | Upgrade to the latest Next.js version following official migration guides |
| `metadata` | Add SEO metadata to pages (Server Components only) |
| `hydration-error` | Diagnose and fix React hydration errors |
| `best-practices` | Next.js best practices (runtime selection, async params) |
| `debug-tricks` | Speed up debugging - MCP endpoint, rebuild specific routes |
| `directives` | Background knowledge: what 'use client', 'use server' actually mean (auto-applied) |

## Installation

```bash
# List available skills
npx add-skill huozhi/next-skills --list

# Install a specific skill
npx add-skill huozhi/next-skills --skill debug-tricks

# Install multiple skills
npx add-skill huozhi/next-skills --skill upgrade --skill best-practices

# Install all skills
npx add-skill huozhi/next-skills
```

## Usage

Once installed, invoke skills using slash commands:

```
/upgrade
/metadata
/hydration-error
/best-practices
/debug-tricks
```

**Background skills** (like `directives`) are automatically applied by Claude when relevant - they don't appear in the slash command menu but provide context to prevent common mistakes.

## Structure

```
next-skills/
├── skills/
│   ├── upgrade/SKILL.md
│   ├── metadata/SKILL.md
│   ├── hydration-error/SKILL.md
│   ├── best-practices/SKILL.md
│   ├── debug-tricks/SKILL.md
│   └── directives/SKILL.md
└── README.md
```

## Contributing

Each skill follows the [Agent Skills open standard](https://github.com/anthropics/skills):

1. Create a directory under `skills/` with the skill name
2. Add a `SKILL.md` file with YAML frontmatter:
   ```yaml
   ---
   name: skill-name
   description: Brief description (used for discovery and auto-suggestion)
   ---
   ```
3. Reference official Next.js documentation rather than duplicating content
