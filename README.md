# Next.js Agent Skills

A Claude Code plugin with agent skills for common Next.js workflows.

## Skills

| Skill | Description |
|-------|-------------|
| `/upgrade` | Upgrade to the latest Next.js version following official migration guides |
| `/metadata` | Add SEO metadata to pages (Server Components only) |
| `/hydration-error` | Diagnose and fix React hydration errors |
| `/best-practices` | Next.js best practices (runtime selection, async params) |
| `/debug-tricks` | Speed up debugging - MCP endpoint, rebuild specific routes |

## Installation

### From GitHub (once published)

```
/plugin marketplace add your-org/next-skills
```

### From local path

```
/plugin install /path/to/next-skills
```

### Manual installation

Copy the `skills/` directory to your project's `.claude/skills/` or global `~/.claude/skills/`.

## Usage

Once installed, invoke skills using slash commands:

```
/upgrade
/metadata
/hydration-error
/best-practices
/debug-tricks
```

Skills are also automatically suggested by Claude when relevant to your task.

## Structure

```
next-skills/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── skills/
│   ├── upgrade/SKILL.md
│   ├── metadata/SKILL.md
│   ├── hydration-error/SKILL.md
│   ├── best-practices/SKILL.md
│   └── debug-tricks/SKILL.md
└── README.md
```

## Contributing

Each skill follows the [Agent Skills open standard](https://github.com/anthropics/agent-skills):

1. Create a directory under `skills/` with the skill name
2. Add a `SKILL.md` file with YAML frontmatter:
   ```yaml
   ---
   description: Brief description (used by Claude to decide when to apply)
   argument-hint: "[optional-arg]"
   ---
   ```
3. Reference official Next.js documentation rather than duplicating content
