# Directives

Next.js specific directives for optimizing your application.

## 'use cache' (Experimental)

Marks a function or component for caching.

```tsx
'use cache'

export async function getCachedData() {
  return await fetchData()
}
```

### Usage Locations

- **Top of file** - marks all exports as cached
- **Inside async function** - inline caching

### Cache Lifecycle

```tsx
'use cache'
import { cacheLife } from 'next/cache'

export async function getCachedData() {
  cacheLife('hours')
  return await fetchData()
}
```

Available presets: `'seconds'`, `'minutes'`, `'hours'`, `'days'`, `'weeks'`, `'max'`

### Cache Tags

```tsx
'use cache'
import { cacheTag } from 'next/cache'

export async function getPost(id: string) {
  cacheTag(`post-${id}`)
  return await fetchPost(id)
}
```

Invalidate with:

```tsx
'use server'
import { revalidateTag } from 'next/cache'

export async function updatePost(id: string) {
  await savePost(id)
  revalidateTag(`post-${id}`)
}
```

## 'use client'

Marks a component as a Client Component. Required for:
- React hooks (`useState`, `useEffect`, etc.)
- Event handlers (`onClick`, `onChange`)
- Browser APIs (`window`, `localStorage`)

```tsx
'use client'

export function Counter() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

## 'use server'

Marks a function as a Server Action. Can be passed to client components.

```tsx
'use server'

export async function submitForm(formData: FormData) {
  // Runs on server
}
```

Or inline:

```tsx
export default function Page() {
  async function submit() {
    'use server'
    // Runs on server
  }
  return <form action={submit}>...</form>
}
```
