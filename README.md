# Next.js Agent Skills have moved

The Next.js agent skills that used to live here now live in the Next.js
repository, so they stay version-matched with the framework instead of
drifting in a separate repo.

New home: https://github.com/vercel/next.js/tree/canary/skills

## Install from the new location

```bash
# Current Next.js workflow skills
npx skills add vercel/next.js

# Or a specific skill
npx skills add vercel/next.js --skill next-cache-components-optimizer
```

Browse the directory above for the current list.

## Where each old skill went

Skills were split by type. Workflow skills still install via `npx skills`.
Reference knowledge is now delivered automatically through Next.js bundled
docs (`next/dist/docs/`) and the `AGENTS.md` / `CLAUDE.md` agent rules that
`next dev` generates (Next.js 16.3+), so it no longer ships as a skill.

- **`next-cache-components`** moved to two workflow skills:
  `next-cache-components-optimizer` and `next-cache-components-adoption`.
  Install them from the new location above.
- **`next-best-practices`** is no longer a skill. This knowledge is now
  delivered through the bundled docs and the auto-generated `AGENTS.md` /
  `CLAUDE.md` written by `next dev` (Next.js 16.3+). No separate install.
- **`next-upgrade`** is no longer a skill. Migration guides ship in the
  bundled docs; run `npx @next/codemod@latest upgrade` to upgrade.

## On Next.js 16.1 or earlier?

The auto-generated `AGENTS.md` / `CLAUDE.md` require Next.js 16.3+. On older
versions you can still get the version-matched docs — pull them into your
project manually:

```bash
npx @next/codemod@canary agents-md
```

This downloads the bundled docs to `.next-docs/` and points your `AGENTS.md`
at them. See [Set up your Next.js project for AI coding agents](https://nextjs.org/docs/app/guides/ai-agents)
for the full setup.

## Already installed the old skills?

Your local copies still work, but will not receive updates. Re-install from
the new location to stay current.
