# Troubleshooting Cache Components

Common issues and how to resolve them.

## Issue: Cache Not Updating After Mutation

**Symptoms:**
- User creates/edits content but doesn't see changes
- Old data persists after form submission
- Other users see updates but the author doesn't

**Causes & Solutions:**

### 1. Missing cache invalidation in Server Action

```tsx
// ❌ Missing invalidation
'use server'
export async function createPost(data: FormData) {
  await db.posts.create({ data })
  // Cache never invalidated!
}

// ✅ Add updateTag() for immediate invalidation
'use server'
import { updateTag } from 'next/cache'

export async function createPost(data: FormData) {
  await db.posts.create({ data })
  updateTag('feed')
  updateTag('posts')
}
```

### 2. Tag name mismatch

```tsx
// ❌ Tags don't match
async function PostContent({ postId }) {
  'use cache'
  cacheTag(`post-${postId}`)  // Uses "post-123"
  // ...
}

'use server'
export async function editPost(postId, data) {
  await db.posts.update({ where: { id: postId }, data })
  updateTag(`posts-${postId}`)  // Uses "posts-123" - WRONG!
}

// ✅ Use consistent tag naming
async function PostContent({ postId }) {
  'use cache'
  cacheTag(`post-${postId}`)
  // ...
}

'use server'
export async function editPost(postId, data) {
  await db.posts.update({ where: { id: postId }, data })
  updateTag(`post-${postId}`)  // Matches!
}
```

### 3. Using revalidateTag instead of updateTag for user actions

```tsx
// revalidateTag = stale-while-revalidate (user may see old data first)
// Good for webhooks/external triggers, not for user actions
revalidateTag('posts', 'max')

// updateTag = immediate invalidation (user sees fresh data)
// Use this for Server Actions where user needs to see their changes
updateTag('posts')
```

Use `updateTag()` for user-initiated mutations where they need to see their own changes immediately. Use `revalidateTag()` in Route Handlers for webhooks.

---

## Issue: Page Still Slow Despite Caching

**Symptoms:**
- Page loads slowly even after implementing caching
- No improvement in performance metrics
- Cache debug shows misses

**Causes & Solutions:**

### 1. Cache Components not enabled

```bash
# Check if enabled
grep -r "cacheComponents" next.config.*
```

```ts
// next.config.ts - ensure this is set
const nextConfig: NextConfig = {
  cacheComponents: true,
}
```

### 2. Dynamic APIs forcing request-time render

```tsx
// ❌ cookies() inside cache scope - will error or force dynamic
async function Content() {
  'use cache'
  const session = await cookies()  // ERROR!
  // ...
}

// ✅ Extract dynamic data outside cache scope
async function Content({ userId }: { userId: string }) {
  'use cache'
  cacheTag(`user-${userId}`)
  // Use userId passed as prop
}

// In parent component
async function Page() {
  const session = await cookies()
  return (
    <Suspense fallback={<Skeleton />}>
      <Content userId={session.get('userId')?.value} />
    </Suspense>
  )
}
```

### 3. Client-side data fetching

```tsx
// ❌ Client fetch - no server caching
'use client'
function Posts() {
  const { data } = useSWR('/api/posts', fetcher)
  return <List posts={data} />
}

// ✅ Server Component with caching
async function Posts() {
  'use cache'
  cacheTag('posts')
  cacheLife('minutes')
  
  const posts = await db.posts.findMany()
  return <List posts={posts} />
}
```

### 4. Missing Suspense boundaries

```tsx
// ❌ Dynamic content blocks cached content
export default async function Page() {
  return (
    <>
      <CachedContent />      {/* Ready instantly */}
      <DynamicComments />    {/* Blocks everything! */}
    </>
  )
}

// ✅ Dynamic content streams separately
export default async function Page() {
  return (
    <>
      <CachedContent />
      <Suspense fallback={<Skeleton />}>
        <DynamicComments />
      </Suspense>
    </>
  )
}
```

---

## Issue: Build Errors with Cache Components

### Error: "Accessing cookies/headers outside Suspense"

```tsx
// ❌ Runtime data without Suspense
export default async function Page() {
  const session = await cookies()  // Error!
  return <Profile userId={session.get('userId')} />
}

// ✅ Wrap in Suspense
export default function Page() {
  return (
    <Suspense fallback={<Skeleton />}>
      <ProfileLoader />
    </Suspense>
  )
}

async function ProfileLoader() {
  const session = await cookies()
  return <Profile userId={session.get('userId')?.value} />
}
```

### Error: "Empty generateStaticParams"

```tsx
// ❌ Empty array with Cache Components
export function generateStaticParams() {
  return []  // Error!
}

// ✅ Provide at least one param
export async function generateStaticParams() {
  const posts = await db.posts.findMany({ take: 10 })
  return posts.map(post => ({ id: post.id }))
}
```

### Error: "Blocking data was accessed outside of Suspense"

This error means you have async data fetching that isn't cached or wrapped in Suspense.

```tsx
// ❌ Uncached async data outside Suspense
export default async function Page() {
  const data = await fetch('...')  // Blocking!
  return <div>{data}</div>
}

// ✅ Option 1: Cache it
export default async function Page() {
  'use cache'
  cacheLife('hours')
  
  const data = await fetch('...')
  return <div>{data}</div>
}

// ✅ Option 2: Wrap in Suspense
export default function Page() {
  return (
    <Suspense fallback={<Loading />}>
      <DataComponent />
    </Suspense>
  )
}
```

---

## Debug Commands

### Enable cache debug logging

```bash
NEXT_PRIVATE_DEBUG_CACHE=1 next start
```

This shows cache hits, misses, and revalidations in the console.

### Enable fetch logging in development

```ts
// next.config.ts
const nextConfig: NextConfig = {
  cacheComponents: true,
  logging: {
    fetches: {
      fullUrl: true,
    },
  },
}
```

### Check build output for caching status

```bash
next build
```

Look for route indicators:
- `○` (Static) - fully prerendered
- `◐` (Partial Prerender) - static shell with dynamic parts
- `λ` (Dynamic) - rendered at request time

---

## Verifying Cache Behavior

### In development

1. Enable debug logging (see above)
2. Load page, check console for cache status
3. Reload, should see cache hits

### In production

1. Build and start: `next build && next start`
2. Load page twice, second should be faster
3. Check response headers for cache status

### Testing invalidation

1. Load cached page
2. Trigger mutation (create/edit)
3. Reload - should see fresh data immediately

```tsx
// Test Server Action
'use server'
export async function testInvalidation() {
  console.log('Invalidating cache...')
  updateTag('test-tag')
  console.log('Cache invalidated')
}
```

---

## Common Mistakes Checklist

- [ ] `cacheComponents: true` in next.config
- [ ] `'use cache'` directive at top of function body
- [ ] `cacheTag()` called inside `'use cache'` scope
- [ ] `cacheLife()` called inside `'use cache'` scope
- [ ] `updateTag()` in Server Actions after mutations
- [ ] Tag names match between cache and invalidation
- [ ] Dynamic content wrapped in `<Suspense>`
- [ ] No `cookies()`/`headers()` inside `'use cache'`
- [ ] No deprecated `export const revalidate`
- [ ] No client-side fetching for shared data
