# Error Handling

Handle errors gracefully in Next.js applications.

Reference: https://nextjs.org/docs/app/getting-started/error-handling

## Error Boundaries

### `error.tsx`

Catches errors in a route segment and its children:

```tsx
'use client'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  )
}
```

**Important:** `error.tsx` must be a Client Component.

### `global-error.tsx`

Catches errors in root layout:

```tsx
'use client'

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <html>
      <body>
        <h2>Something went wrong!</h2>
        <button onClick={() => reset()}>Try again</button>
      </body>
    </html>
  )
}
```

**Important:** Must include `<html>` and `<body>` tags.

## Server Actions: Navigation API Gotcha

**Do NOT wrap navigation APIs in try-catch.** They throw special errors that Next.js handles internally.

Reference: https://nextjs.org/docs/app/api-reference/functions/redirect#behavior

```tsx
'use server'

import { redirect } from 'next/navigation'
import { notFound } from 'next/navigation'

// ❌ WRONG: try-catch catches the navigation "error"
async function createPost(formData: FormData) {
  try {
    const post = await db.post.create({ ... })
    redirect(`/posts/${post.id}`)  // This throws!
  } catch (error) {
    // redirect() throw is caught here - navigation fails!
    return { error: 'Failed to create post' }
  }
}

// ✅ CORRECT: Call navigation APIs outside try-catch
async function createPost(formData: FormData) {
  let post
  try {
    post = await db.post.create({ ... })
  } catch (error) {
    return { error: 'Failed to create post' }
  }
  redirect(`/posts/${post.id}`)  // Outside try-catch
}

// ✅ CORRECT: Re-throw navigation errors
async function createPost(formData: FormData) {
  try {
    const post = await db.post.create({ ... })
    redirect(`/posts/${post.id}`)
  } catch (error) {
    if (error instanceof Error && error.message === 'NEXT_REDIRECT') {
      throw error  // Re-throw navigation errors
    }
    return { error: 'Failed to create post' }
  }
}
```

Same applies to:
- `redirect()`
- `permanentRedirect()`
- `notFound()`

## Not Found

### `not-found.tsx`

Custom 404 page for a route segment:

```tsx
export default function NotFound() {
  return (
    <div>
      <h2>Not Found</h2>
      <p>Could not find the requested resource</p>
    </div>
  )
}
```

### Triggering Not Found

```tsx
import { notFound } from 'next/navigation'

export default async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params
  const post = await getPost(id)

  if (!post) {
    notFound()  // Renders closest not-found.tsx
  }

  return <div>{post.title}</div>
}
```

## Error Hierarchy

Errors bubble up to the nearest error boundary:

```
app/
├── error.tsx           # Catches errors from all children
├── blog/
│   ├── error.tsx       # Catches errors in /blog/*
│   └── [slug]/
│       ├── error.tsx   # Catches errors in /blog/[slug]
│       └── page.tsx
└── layout.tsx          # Errors here go to global-error.tsx
```
