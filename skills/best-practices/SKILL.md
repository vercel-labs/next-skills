---
name: best-practices
description: Next.js best practices - runtime selection, async params patterns, and modern conventions
---

# Best Practices

Follow Next.js best practices when writing or reviewing code.

## Guidelines

### 1. Use Node.js Runtime by Default

Use the default Node.js runtime for new routes and pages. Only use edge runtime if the project already uses it or there's a specific requirement.

```tsx
// ✅ Default - no runtime config needed (uses Node.js)
export default function Page() { ... }

// ⚠️ Only if already used in project or specifically required
export const runtime = 'edge'
```

Node.js runtime provides full API support and works best for most applications. Edge runtime has limitations (no `fs`, limited `crypto`, etc.) and is only beneficial for specific edge-location latency requirements.

### 2. Async Params and SearchParams (Next.js 15+)

In Next.js 15+, `params` and `searchParams` are asynchronous. Always type them as `Promise<...>` and await them.

**Pages and Layouts:**
```tsx
type Props = { params: Promise<{ slug: string }> }

export default async function Page({ params }: Props) {
  const { slug } = await params
}
```

**Route Handlers:**
```tsx
export async function GET(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params
}
```

**SearchParams:**
```tsx
type Props = {
  params: Promise<{ slug: string }>
  searchParams: Promise<{ query?: string }>
}

export default async function Page({ params, searchParams }: Props) {
  const { slug } = await params
  const { query } = await searchParams
}
```

**Synchronous components** - use `React.use()`:
```tsx
import { use } from 'react'

type Props = { params: Promise<{ slug: string }> }

export default function Page({ params }: Props) {
  const { slug } = use(params)
}
```

**generateMetadata:**
```tsx
type Props = { params: Promise<{ slug: string }> }

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { slug } = await params
  return { title: slug }
}
```

**Codemod for migration:**
```bash
npx @next/codemod@latest next-async-request-api .
```

## Reference

- Async Request APIs: https://nextjs.org/docs/app/building-your-application/upgrading/version-15#async-request-apis-breaking-change
- Edge Runtime: https://nextjs.org/docs/app/api-reference/edge
