---
name: directives
description: Next.js directives - 'use cache' for caching functions and components
user-invocable: false
---

# Next.js Directives

Next.js specific directives for optimizing your application.

## 'use cache' (Experimental)

Marks a function or component for caching. This is a Next.js specific directive.

```tsx
'use cache'

export async function getCachedData() {
  return await fetchData()
}
```

Can be used:
- At the top of a file (marks all exports as cached)
- Inside an async function (inline caching)

### Cache options

```tsx
'use cache'
import { cacheLife } from 'next/cache'

export async function getCachedData() {
  cacheLife('hours')
  return await fetchData()
}
```

## Reference

- Caching: https://nextjs.org/docs/app/building-your-application/caching
- use cache directive: https://nextjs.org/docs/app/api-reference/directives/use-cache
