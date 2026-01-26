---
name: next-fundamentals
description: Next.js fundamentals - RSC boundaries, async patterns, metadata, image/font optimization, bundling, hydration errors
user-invocable: false
---

# Next.js Fundamentals

Apply these rules when writing or reviewing Next.js code.

## RSC Boundaries

Detect invalid React Server Component patterns.

See [rsc-boundaries.md](./rsc-boundaries.md) for:
- Async client component detection (invalid)
- Non-serializable props detection
- Server Action exceptions

## Async Patterns

Next.js 15+ async API changes.

See [async-patterns.md](./async-patterns.md) for:
- Async `params` and `searchParams`
- Async `cookies()` and `headers()`
- Migration codemod

## Runtime Selection

See [runtime-selection.md](./runtime-selection.md) for:
- Default to Node.js runtime
- When Edge runtime is appropriate

## Directives

See [directives.md](./directives.md) for:
- `'use cache'` directive
- Cache lifecycle options

## Metadata & OG Images

See [metadata.md](./metadata.md) for:
- Static and dynamic metadata
- `generateMetadata` function
- OG image generation with `next/og`
- File-based metadata conventions

## Image Optimization

See [image.md](./image.md) for:
- Always use `next/image` over `<img>`
- Remote images configuration
- Responsive `sizes` attribute
- Blur placeholders
- Priority loading for LCP

## Font Optimization

See [font.md](./font.md) for:
- `next/font` setup
- Google Fonts, local fonts
- Tailwind CSS integration
- Preloading subsets

## Bundling

See [bundling.md](./bundling.md) for:
- Server-incompatible packages
- ESM/CommonJS issues
- Bundle analysis
- Webpack to Turbopack migration

## Hydration Errors

See [hydration-error.md](./hydration-error.md) for:
- Common causes (browser APIs, dates, invalid HTML)
- Debugging with error overlay
- Fixes for each cause
