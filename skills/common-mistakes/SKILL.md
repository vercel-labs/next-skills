---
description: Identify and fix common Next.js mistakes - edge runtime misuse, async params, and other gotchas
---

# Common Mistakes

Identify and fix common Next.js mistakes when writing or reviewing code.

## Rules

### 1. Avoid Edge Runtime Unless Already Used

**Do not add `runtime: 'edge'`** to new routes or pages unless the codebase already uses edge runtime.

```tsx
// ❌ Don't add this to new code
export const runtime = 'edge'
```

Why:
- Edge runtime has limited Node.js API support (no `fs`, limited `crypto`, etc.)
- Most applications don't benefit from edge and work better with Node.js runtime
- Only use edge if: the project already uses it, or there's a specific requirement (e.g., low latency at edge locations)

If you encounter existing edge runtime code, keep it. If writing new code, use the default Node.js runtime.

### 2. Params and SearchParams Are Promises (Next.js 15+)

In Next.js 15+, `params` and `searchParams` in pages, layouts, routes, and metadata are **asynchronous** and must be awaited.

**Pages and Layouts:**
```tsx
// ❌ Wrong - treating params as synchronous
type Props = { params: { slug: string } }

export default function Page({ params }: Props) {
  const slug = params.slug // Error: params is a Promise
}

// ✅ Correct - await the params
type Props = { params: Promise<{ slug: string }> }

export default async function Page({ params }: Props) {
  const { slug } = await params
}
```

**Route Handlers:**
```tsx
// ❌ Wrong
export async function GET(
  request: Request,
  { params }: { params: { id: string } }
) {
  const id = params.id
}

// ✅ Correct
export async function GET(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params
}
```

**SearchParams in Pages:**
```tsx
// ✅ Correct - searchParams is also a Promise
type Props = {
  params: Promise<{ slug: string }>
  searchParams: Promise<{ query?: string }>
}

export default async function Page({ params, searchParams }: Props) {
  const { slug } = await params
  const { query } = await searchParams
}
```

**Synchronous Components** - use `React.use()`:
```tsx
import { use } from 'react'

type Props = { params: Promise<{ slug: string }> }

export default function Page({ params }: Props) {
  const { slug } = use(params)
}
```

**generateMetadata:**
```tsx
// ✅ Correct
type Props = { params: Promise<{ slug: string }> }

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { slug } = await params
  return { title: slug }
}
```

**Codemod available:**
```bash
npx @next/codemod@latest next-async-request-api .
```

## Instructions

When reviewing or writing Next.js code:

1. **Check for edge runtime**: If you see `runtime: 'edge'` being added to new code, remove it unless there's a specific requirement
2. **Check params/searchParams types**: Ensure they're typed as `Promise<...>` and awaited
3. **Run the codemod**: For large codebases, use the codemod to automatically fix async params issues

## Reference

- Async Request APIs: https://nextjs.org/docs/app/building-your-application/upgrading/version-15#async-request-apis-breaking-change
- Edge Runtime: https://nextjs.org/docs/app/api-reference/edge
