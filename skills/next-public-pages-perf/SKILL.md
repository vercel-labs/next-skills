---
name: next-public-pages-perf
description: |
  Performance optimization for public pages in high-traffic Next.js sites using Cache Components and Partial Prerendering (PPR).

  **USE CASES**: Optimizing slow Next.js sites, improving page load times, implementing caching strategies, migrating from old Next.js patterns (export const revalidate, unstable_cache), converting client-side data fetching to Server Components.

  **TRIGGERS**: "site is slow", "pages loading slowly", "slow navigation", "high traffic", "performance optimization", "cache", "prerender", "static pages", forum optimization, content site optimization, migrating from useEffect/useSWR/useQuery patterns.

  **TARGET SITES**: Forums (Reddit-style), ecommerce, blogs, news sites, content platforms - any site where many users view the same pages.
---

# Next.js Performance Optimization

Optimize high-traffic Next.js sites using Cache Components and Partial Prerendering (PPR).

## When to Use This Skill

Use when:
- Site is slow or pages load slowly
- High traffic causing server load
- Need to cache shared content
- Migrating from old Next.js patterns
- Converting client-side fetching to Server Components

## Quick Start

### 1. Enable Cache Components

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  cacheComponents: true,
}

export default nextConfig
```

### 2. Apply the PPR Pattern

**Feed/Index pages** - Short cache, updates frequently:

```tsx
import { cacheTag, cacheLife } from 'next/cache'

async function PostFeed() {
  'use cache'
  cacheTag('feed')
  cacheLife('minutes')  // Short - feed updates frequently
  
  const posts = await db.posts.findMany({
    orderBy: { createdAt: 'desc' },
    take: 50,
  })
  return <PostList posts={posts} />
}
```

**Individual post pages** - Longer cache, content is stable:

```tsx
import { Suspense } from 'react'
import { cacheTag, cacheLife } from 'next/cache'

// POST CONTENT - Long cache, content rarely changes
async function PostContent({ postId }: { postId: string }) {
  'use cache'
  cacheTag(`post-${postId}`)
  cacheLife('days')  // Long - post content is stable
  
  const post = await db.posts.findUnique({ where: { id: postId } })
  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  )
}

// COMMENTS - Dynamic, streams after shell loads
async function Comments({ postId }: { postId: string }) {
  const comments = await db.comments.findMany({ where: { postId } })
  return <CommentList comments={comments} />
}

// PAGE - Combines cached post + streaming comments
export default async function PostPage({ params }) {
  const { id } = await params
  return (
    <>
      <PostContent postId={id} />
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments postId={id} />
      </Suspense>
    </>
  )
}
```

### 3. Invalidate on Mutation

```tsx
'use server'
import { updateTag } from 'next/cache'

export async function editPost(postId: string, data: FormData) {
  await db.posts.update({ where: { id: postId }, data })
  updateTag(`post-${postId}`)  // User sees edit immediately
}
```

### 4. Pre-render Popular Posts at Build Time

Use `generateStaticParams` to pre-render popular posts at build time. Other posts are rendered and cached on-demand when first visited.

```tsx
// app/post/[id]/page.tsx
import { cacheTag, cacheLife } from 'next/cache'

async function PostContent({ postId }: { postId: string }) {
  'use cache'
  cacheTag(`post-${postId}`)
  cacheLife('days')
  
  const post = await db.posts.findUnique({ where: { id: postId } })
  return <Article post={post} />
}

export default async function PostPage({ params }) {
  const { id } = await params
  return <PostContent postId={id} />
}

// Pre-render top 100 popular posts at build time
// All other posts render on-demand and get cached
export async function generateStaticParams() {
  const popularPosts = await db.posts.findMany({
    orderBy: { views: 'desc' },
    take: 100,
    select: { id: true },
  })
  return popularPosts.map(post => ({ id: post.id }))
}
```

**Result:**
- Top 100 posts: Pre-rendered at build time, served instantly
- All other posts: Rendered on first visit, then cached
- Both benefit from `'use cache'` + `cacheLife('days')`

## Performance Audit Workflow

Run these commands to identify optimization opportunities:

```bash
# Step 1: Check if Cache Components enabled
grep -r "cacheComponents" next.config.*

# Step 2: Find deprecated/problematic patterns

# Server-side deprecated patterns
grep -r "export const revalidate" app/     # OLD - replace with cacheLife
grep -r "export const dynamic" app/        # OLD - remove or use Suspense
grep -r "unstable_cache" app/              # OLD - replace with 'use cache'

# Client-side data fetching (blocks page, moves work to browser)
grep -r "useEffect.*fetch" app/ components/    # Client fetch in useEffect
grep -r "useSWR\|useQuery" app/ components/    # SWR/React Query
grep -r "'use client'" app/**/page.tsx         # Client component pages

# Step 3: Find pages without caching
grep -rL "'use cache'" app/**/page.tsx
```

## Decision Tree

When optimizing a page, ask:

```
Is this a shared, high-traffic page (feed, post, thread)?
├── YES → Should content be cached?
│   ├── Primary content (post body, title) → 'use cache' + cacheTag + cacheLife
│   │   └── Needs immediate update on edit? → updateTag() in Server Action
│   └── Secondary content (comments) → Can stream → wrap in <Suspense>
│
└── Is there user-specific content (auth, preferences)?
    └── YES → Wrap in <Suspense> (streams at request time)
```

## Key APIs

| API | Purpose | When to Use |
|-----|---------|-------------|
| `'use cache'` | Mark as cacheable | Data same for all users |
| `cacheLife('hours')` | Set cache duration | Inside `'use cache'` scope |
| `cacheTag('posts')` | Tag for invalidation | Inside `'use cache'` scope |
| `updateTag('posts')` | Immediate invalidation | Server Actions (user edits own content) |
| `revalidateTag('posts', 'max')` | Background revalidation | Route Handlers (webhooks, external systems) |
| `<Suspense>` | Define streaming boundary | User-specific or real-time content |

### Cache Profiles

| Profile | Use Case | Example |
|---------|----------|---------|
| `'seconds'` | Near real-time, volatile | Upvote counts, "new" sort, rising posts |
| `'minutes'` | Frequently updated indexes | Home feed, hot posts, community feeds |
| `'hours'` | Moderately stable content | User profiles, community info |
| `'days'` | Stable content | Individual posts, articles |
| `'weeks'` | Rarely changing | About pages, rules, static info |
| `'max'` | Permanent | Legal pages, archived content |

## Migration from Old Patterns

| Problematic Pattern | New Pattern |
|---------------------|-------------|
| `export const revalidate = 3600` | `cacheLife('hours')` inside `'use cache'` |
| `export const dynamic = 'force-static'` | `'use cache'` + `cacheLife('max')` |
| `export const dynamic = 'force-dynamic'` | Remove, or wrap in `<Suspense>` |
| `unstable_cache(fn, tags)` | `'use cache'` + `cacheTag()` |
| `fetch({ next: { revalidate } })` | `'use cache'` with `cacheLife()` |
| `useEffect(() => fetch(...))` | Server Component with `'use cache'` |
| `useSWR()` / `useQuery()` | Server Component + streaming |
| `'use client'` page + fetching | Server Component page |

## Additional Resources

- For detailed patterns, see [PATTERNS.md](PATTERNS.md)
- For debugging issues, see [TROUBLESHOOTING.md](TROUBLESHOOTING.md)

## External References

- [Building Public Pages Guide](https://nextjs.org/docs/app/guides/public-static-pages)
- [Cache Components](https://nextjs.org/docs/app/getting-started/cache-components)
- [Caching in Next.js](https://nextjs.org/docs/app/guides/caching)
- [use cache directive](https://nextjs.org/docs/app/api-reference/directives/use-cache)
- [cacheLife API](https://nextjs.org/docs/app/api-reference/functions/cacheLife)
- [cacheTag API](https://nextjs.org/docs/app/api-reference/functions/cacheTag)
- [updateTag API](https://nextjs.org/docs/app/api-reference/functions/updateTag)
