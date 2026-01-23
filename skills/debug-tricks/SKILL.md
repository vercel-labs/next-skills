---
name: debug-tricks
description: Speed up Next.js debugging - MCP endpoint for app state, rebuild specific routes
---

# Debug Tricks

Tricks to speed up debugging Next.js applications.

## MCP Endpoint (Dev Server)

In development, Next.js exposes a `/_next/mcp` endpoint for quick access to app state via MCP (Model Context Protocol).

```bash
curl http://localhost:3000/_next/mcp
```

Get instant access to:
- Build errors and warnings
- All routes in the application
- App configuration
- Diagnostic information

Faster than manually checking terminal output or browser console.

## Rebuild Specific Routes (Next.js 16+)

Use `--debug-build-paths` to rebuild only specific routes instead of the entire app:

```bash
# Rebuild a specific route
next build --debug-build-paths "/dashboard"

# Rebuild routes matching a glob
next build --debug-build-paths "/api/*"

# Dynamic routes
next build --debug-build-paths "/blog/[slug]"
```

Use this to:
- Quickly verify a build fix without full rebuild
- Debug static generation issues for specific pages
- Iterate faster on build errors
