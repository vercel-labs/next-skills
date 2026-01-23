---
name: directives
description: Next.js and React directives - 'use client' does NOT mean browser-only, 'use server' marks Server Actions
user-invocable: false
---

# Next.js Directives

Understanding what directives actually mean in Next.js.

## 'use client'

**Common misconception**: `'use client'` means the code runs only in the browser.

**Reality**: Client Components still render on the server (SSR), then hydrate on the client.

```tsx
'use client'

// This component:
// 1. Renders on the server (SSR) - outputs HTML
// 2. Hydrates on the client - attaches event handlers, enables interactivity
// 3. Can use hooks, event handlers, browser APIs (after hydration)
export function Counter() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

The directive marks the **Server/Client Component boundary** - everything imported into a Client Component becomes part of the client bundle.

Use `'use client'` when you need:
- React hooks (`useState`, `useEffect`, etc.)
- Event handlers (`onClick`, `onChange`, etc.)
- Browser APIs (after hydration)
- Class components with lifecycle methods

## 'use server'

Marks functions as **Server Actions** - functions that execute on the server and can be called from the client.

```tsx
'use server'

// This function runs on the server, even when called from a Client Component
export async function saveData(formData: FormData) {
  await db.save(formData)
}
```

Can be used:
- At the top of a file (marks all exports as Server Actions)
- Inside an async function (inline Server Action)

## 'use cache' (Experimental)

Marks a function or component for caching.

```tsx
'use cache'

export async function getCachedData() {
  return await fetchData()
}
```

## Reference

- Client Components: https://nextjs.org/docs/app/building-your-application/rendering/client-components
- Server Components: https://nextjs.org/docs/app/building-your-application/rendering/server-components
- Server Actions: https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations
- Directives: https://react.dev/reference/rsc/directives
