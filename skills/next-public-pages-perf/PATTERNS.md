# Forum/Reddit-Style Patterns

Common caching patterns for high-traffic content sites.

## Critical Rules (Apply to All Patterns)

1. **Await params INSIDE the cache boundary**

   ```tsx
   // ✅ CORRECT: Await inside 'use cache'
   async function PostContent({ params }: { params: Promise<{ id: string }> }) {
     "use cache"
     const { id } = await params // Inside cache boundary
     cacheTag(`post-${id}`)
     // ...
   }

   // ❌ WRONG: Await at page level
   export default async function Page({ params }) {
     const { id } = await params // At page level - makes page dynamic
     return <PostContent postId={id} />
   }
   ```

2. **Cached components don't need Suspense** (but it's harmless)

   ```tsx
   // ✅ PREFERRED: No Suspense for cached content
   <CachedContent params={params} />
   <Suspense><DynamicContent /></Suspense>     {/* Required for dynamic */}

   // ✅ ALSO FINE: Suspense won't hurt, fallback just won't show
   <Suspense><CachedContent params={params} /></Suspense>
   ```

3. **All dynamic routes MUST have generateStaticParams**
   ```tsx
   // REQUIRED for every [param] route
   export function generateStaticParams() {
     return [{ id: "post-1" }] // At least one param
   }
   ```

---

## Choosing Cache Lifetimes

| Content Type               | Lifetime                   | Rationale                        |
| -------------------------- | -------------------------- | -------------------------------- |
| Index pages (feeds, sorts) | `'minutes'` or `'seconds'` | Updates frequently, many viewers |
| Individual posts           | `'days'`                   | Content stable after creation    |
| Community/user info        | `'hours'` to `'days'`      | Rarely changes                   |
| "New" and "rising" sorts   | `'seconds'`                | Must be fresh                    |
| "Hot" and "top" sorts      | `'minutes'`                | Some staleness acceptable        |
| Upvote counts              | `'seconds'` or dynamic     | Depends on real-time needs       |

---

## Pattern 1: Feed/Home Page

The main feed shows posts that are the same for all users. Cache the feed list and invalidate when new posts are created.

```tsx
import { Suspense } from "react"
import { cacheTag, cacheLife, updateTag } from "next/cache"

// CACHED FEED - Same for all anonymous users
async function PublicFeed() {
  "use cache"
  cacheTag("feed")
  cacheLife("minutes")

  const posts = await db.posts.findMany({
    orderBy: { createdAt: "desc" },
    take: 50,
  })
  return <PostList posts={posts} />
}

// PERSONALIZED FEED - Different per user, streams
async function PersonalizedFeed() {
  const session = await getSession()
  const posts = await db.posts.findMany({
    where: { authorId: { in: session.following } },
    orderBy: { createdAt: "desc" },
  })
  return <PostList posts={posts} />
}

export default async function HomePage() {
  const session = await getSession()

  if (session) {
    // Logged in: personalized feed streams
    return (
      <Suspense fallback={<FeedSkeleton />}>
        <PersonalizedFeed />
      </Suspense>
    )
  }

  // Anonymous: cached public feed
  return <PublicFeed />
}

// app/actions.ts - Server Action
;("use server")

import { updateTag } from "next/cache"

export async function createPost(data: FormData) {
  await db.posts.create({ data })
  updateTag("feed") // New post appears immediately
}
```

**Key points:**

- Public feed: `cacheLife('minutes')` - acceptable staleness
- Personalized feed: wrap in `<Suspense>` - must stream
- New post: `updateTag('feed')` - immediate visibility

---

## Pattern 2: Post/Thread Page

Post content is cached, comments stream in after.

```tsx
import { Suspense } from "react"
import { cacheTag, cacheLife } from "next/cache"

// POST CONTENT - Cached, awaits params INSIDE cache boundary
async function PostContent({ params }: { params: Promise<{ id: string }> }) {
  "use cache"
  const { id } = await params // Await inside cache, not at page level
  cacheTag(`post-${id}`)
  cacheLife("days")

  const post = await db.posts.findUnique({
    where: { id },
    include: { author: true },
  })

  if (!post) return null

  return (
    <article>
      <h1>{post.title}</h1>
      <div className="prose">{post.content}</div>
      <footer>
        <span>By {post.author.name}</span>
        <span>{post.upvotes} upvotes</span>
      </footer>
    </article>
  )
}

// COMMENTS - Dynamic (no cache), streams after shell
async function Comments({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params
  const comments = await db.comments.findMany({
    where: { postId: id },
    orderBy: { createdAt: "desc" },
    include: { author: true },
  })
  return <CommentList comments={comments} />
}

// PAGE - Passes params Promise down, doesn't await
export default async function PostPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  return (
    <>
      {/* Cached - included in static shell */}
      <PostContent params={params} />

      {/* Dynamic - NEEDS Suspense (streams at request time) */}
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments params={params} />
      </Suspense>
    </>
  )
}

// REQUIRED: All dynamic routes need generateStaticParams
export function generateStaticParams() {
  return [{ id: "post-1" }, { id: "post-2" }]
}

// app/actions.ts - Server Actions
;("use server")

import { updateTag } from "next/cache"

export async function editPost(postId: string, data: FormData) {
  await db.posts.update({ where: { id: postId }, data })
  updateTag(`post-${postId}`)
}

export async function addComment(postId: string, data: FormData) {
  await db.comments.create({ data: { postId, ...data } })
  // Comments are dynamic, no cache to invalidate
  // If you cached comments, you would: updateTag(`post-${postId}-comments`)
}
```

**Key points:**

- Pass `params` Promise to components, await INSIDE cache boundary
- Post content: cached with `cacheLife('days')`, NO Suspense needed
- Comments: dynamic in `<Suspense>` - always fresh
- Edit post: `updateTag()` for immediate update
- `generateStaticParams` is required for dynamic routes

---

## Pattern 3: Upvotes/Reactions

Upvotes can be cached with very short lifetime or kept dynamic.

```tsx
import { cacheTag, cacheLife, updateTag } from "next/cache"

// OPTION A: Short-cached upvotes (slight delay acceptable)
async function UpvoteCount({ postId }: { postId: string }) {
  "use cache"
  cacheTag(`post-${postId}-votes`)
  cacheLife("seconds") // 1s stale, refresh in background

  const count = await db.posts.findUnique({
    where: { id: postId },
    select: { upvotes: true },
  })
  return <span>{count?.upvotes ?? 0}</span>
}

// OPTION B: Real-time upvotes (always fresh)
async function RealtimeUpvoteCount({ postId }: { postId: string }) {
  const count = await db.posts.findUnique({
    where: { id: postId },
    select: { upvotes: true },
  })
  return <span>{count?.upvotes ?? 0}</span>
}

// app/actions.ts - Server Action
;("use server")

import { updateTag } from "next/cache"

export async function upvote(postId: string) {
  await db.posts.update({
    where: { id: postId },
    data: { upvotes: { increment: 1 } },
  })

  // Invalidate both the post cache and votes cache
  updateTag(`post-${postId}`)
  updateTag(`post-${postId}-votes`)
}
```

**Key points:**

- `cacheLife('seconds')` for near-real-time with caching benefits
- Or no cache for truly real-time (wrap in `<Suspense>`)
- Upvote action invalidates relevant caches

---

## Pattern 4: User Profiles

For simple pages where everything can be cached, use `'use cache'` at the page level.

```tsx
import { cacheTag, cacheLife } from "next/cache"

// SIMPLE APPROACH: Cache entire page
export default async function UserProfilePage({
  params,
}: {
  params: Promise<{ username: string }>
}) {
  "use cache"
  const { username } = await params
  cacheTag(`user-${username}`, `user-${username}-posts`)
  cacheLife("hours")

  const user = await db.users.findUnique({
    where: { username },
    include: { _count: { select: { posts: true } } },
  })

  if (!user) return null

  const posts = await db.posts.findMany({
    where: { authorId: user.id },
    orderBy: { createdAt: "desc" },
  })

  return (
    <>
      <div>
        <h1>{user.name}</h1>
        <p>{user.bio}</p>
        <p>{user._count.posts} posts</p>
      </div>
      <PostList posts={posts} />
    </>
  )
}

// REQUIRED: All dynamic routes need generateStaticParams
export function generateStaticParams() {
  return [{ username: "alice" }, { username: "bob" }]
}

// app/actions.ts - Server Action
;("use server")

import { updateTag } from "next/cache"

export async function updateProfile(data: FormData) {
  const session = await getSession()
  await db.users.update({ where: { id: session.userId }, data })
  updateTag(`user-${session.username}`)
}
```

**Key points:**

- Simple pages can use `'use cache'` at page level
- Await params inside the cache boundary
- No Suspense needed - entire page is cached
- `generateStaticParams` is required

---

## Pattern 5: Community Pages with Sort Orders

Community/subreddit pages with different sort orders (hot, new, top). All components are cached and included in the static shell.

**Route structure:**

```
app/r/[community]/
├── layout.tsx          # Shared layout (no data fetching)
├── page.tsx            # Default (hot) sort
└── [sort]/
    └── page.tsx        # Different sorts: new, top, rising
```

```tsx
// app/r/[community]/page.tsx (default = hot)
import { cacheTag, cacheLife } from "next/cache"

// Cached header - awaits params INSIDE cache boundary
async function CommunityHeader({
  params,
}: {
  params: Promise<{ community: string }>
}) {
  "use cache"
  const { community } = await params
  cacheTag(`community-${community}`)
  cacheLife("hours")

  const data = await db.communities.findUnique({
    where: { slug: community },
  })

  return (
    <header>
      <h1>r/{data.name}</h1>
      <p>{data.description}</p>
      <p>{data.memberCount} members</p>
    </header>
  )
}

// Cached posts - awaits params INSIDE cache boundary
async function HotPosts({
  params,
}: {
  params: Promise<{ community: string }>
}) {
  "use cache"
  const { community } = await params
  cacheTag(`community-${community}-hot`)
  cacheLife("minutes")

  const posts = await db.posts.findMany({
    where: { communitySlug: community },
    orderBy: { hotScore: "desc" },
    take: 50,
  })
  return <PostList posts={posts} />
}

// Page passes params Promise down - doesn't await
export default async function CommunityPage({
  params,
}: {
  params: Promise<{ community: string }>
}) {
  return (
    <>
      {/* All cached - NO Suspense needed */}
      <CommunityHeader params={params} />
      <SortTabs params={params} />
      <HotPosts params={params} />
    </>
  )
}

// REQUIRED: Pre-render communities
export function generateStaticParams() {
  return [
    { community: "programming" },
    { community: "nextjs" },
    { community: "webdev" },
  ]
}
```

```tsx
// app/r/[community]/[sort]/page.tsx
import { cacheTag, cacheLife } from "next/cache"

type SortOrder = "new" | "top" | "rising"

const sortConfigs: Record<SortOrder, { orderBy: any; cacheProfile: string }> = {
  new: { orderBy: { createdAt: "desc" }, cacheProfile: "minutes" },
  top: { orderBy: { upvotes: "desc" }, cacheProfile: "minutes" },
  rising: { orderBy: { risingScore: "desc" }, cacheProfile: "minutes" },
}

// Cached posts - awaits params INSIDE cache boundary
async function SortedPosts({
  params,
}: {
  params: Promise<{ community: string; sort: string }>
}) {
  "use cache"
  const { community, sort } = await params

  if (!sortConfigs[sort as SortOrder]) return null

  cacheTag(`community-${community}-${sort}`)
  cacheLife(sortConfigs[sort as SortOrder].cacheProfile)

  const posts = await db.posts.findMany({
    where: { communitySlug: community },
    orderBy: sortConfigs[sort as SortOrder].orderBy,
    take: 50,
  })
  return <PostList posts={posts} />
}

// Page passes params Promise down
export default async function CommunitySortPage({
  params,
}: {
  params: Promise<{ community: string; sort: string }>
}) {
  return (
    <>
      <CommunityHeader params={params} />
      <SortTabs params={params} />
      <SortedPosts params={params} />
    </>
  )
}

// REQUIRED: Pre-render sort variants for each community
export function generateStaticParams() {
  const communities = ["programming", "nextjs", "webdev"]
  const sorts: SortOrder[] = ["new", "top", "rising"]

  return communities.flatMap((community) =>
    sorts.map((sort) => ({ community, sort })),
  )
}
```

**Key points:**

- Pass `params` Promise to components, await INSIDE cache boundary
- All components cached - NO Suspense needed
- Each cached component has its own `cacheTag` for independent invalidation
- `generateStaticParams` required for all dynamic routes

---

## Pattern 6: Pre-rendering Popular Posts

Use `generateStaticParams` to pre-render popular posts at build time. All other posts render on-demand and get cached.

```tsx
// app/post/[id]/page.tsx
import { Suspense } from "react"
import { cacheTag, cacheLife } from "next/cache"

// Cached post - awaits params INSIDE cache boundary
async function PostContent({ params }: { params: Promise<{ id: string }> }) {
  "use cache"
  const { id } = await params
  cacheTag(`post-${id}`)
  cacheLife("days")

  const post = await db.posts.findUnique({
    where: { id },
    include: { author: true, community: true },
  })

  if (!post) return null

  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
      <footer>
        <span>Posted in r/{post.community.name}</span>
        <span>by {post.author.name}</span>
      </footer>
    </article>
  )
}

// Dynamic comments - no cache, streams at request time
async function Comments({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params
  const comments = await db.comments.findMany({
    where: { postId: id },
    orderBy: { createdAt: "desc" },
  })
  return <CommentList comments={comments} />
}

// Page passes params Promise down
export default async function PostPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  return (
    <>
      {/* Cached - included in static shell */}
      <PostContent params={params} />

      {/* Dynamic - NEEDS Suspense (streams at request time) */}
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments params={params} />
      </Suspense>
    </>
  )
}

// REQUIRED: Pre-render top 100 most viewed posts at build time
export async function generateStaticParams() {
  const popularPosts = await db.posts.findMany({
    orderBy: { views: "desc" },
    take: 100,
    select: { id: true },
  })
  return popularPosts.map((post) => ({ id: post.id }))
}

// app/actions.ts - Server Action
;("use server")

import { updateTag } from "next/cache"

export async function editPost(postId: string, data: FormData) {
  await db.posts.update({ where: { id: postId }, data })
  updateTag(`post-${postId}`)
}
```

**Key points:**

- Pass `params` Promise to components, await INSIDE cache boundary
- Cached content: NO Suspense (included in static shell)
- Dynamic content: NEEDS Suspense (streams at request time)
- Top 100 posts: Pre-rendered at build, served instantly
- Other posts: Rendered on first visit, then cached
- `generateStaticParams` is required for all dynamic routes

---

## Pattern 7: Webhook-triggered Revalidation

When external systems (CMS, third-party APIs) need to notify your app of changes, use a Route Handler with `revalidateTag`. This is different from Server Actions - webhooks come from external systems, not user interactions.

```tsx
// app/api/webhook/route.ts
import type { NextRequest } from "next/server"
import { revalidateTag } from "next/cache"

// Secret token to verify webhook is from trusted source
const WEBHOOK_SECRET = process.env.WEBHOOK_SECRET

export async function POST(request: NextRequest) {
  // Verify the webhook is authentic
  const authHeader = request.headers.get("authorization")
  if (authHeader !== `Bearer ${WEBHOOK_SECRET}`) {
    return Response.json({ error: "Unauthorized" }, { status: 401 })
  }

  const payload = await request.json()

  // Handle different event types from your CMS/external system
  switch (payload.event) {
    case "post.created":
    case "post.updated":
      // Revalidate the specific post and feeds
      revalidateTag(`post-${payload.postId}`, "max")
      revalidateTag("feed", "max")
      break

    case "post.deleted":
      revalidateTag(`post-${payload.postId}`, "max")
      revalidateTag("feed", "max")
      break

    case "community.updated":
      revalidateTag(`community-${payload.communityId}`, "max")
      break

    default:
      return Response.json({ error: "Unknown event" }, { status: 400 })
  }

  return Response.json({ revalidated: true, event: payload.event })
}
```

### When to use `revalidateTag` vs `updateTag`

| Scenario                                  | Function                            | Why                                                  |
| ----------------------------------------- | ----------------------------------- | ---------------------------------------------------- |
| User edits their own post (Server Action) | `updateTag()`                       | Immediate - user needs to see their changes          |
| CMS publishes new content (Webhook)       | `revalidateTag(tag, 'max')`         | Stale-while-revalidate - slight delay acceptable     |
| Webhook needs immediate expiration        | `revalidateTag(tag, { expire: 0 })` | When external system requires immediate invalidation |

### Example: CMS Integration

```tsx
// app/api/cms-webhook/route.ts
import { revalidateTag } from "next/cache"
import { headers } from "next/headers"
import crypto from "crypto"

// Verify webhook signature (example for a CMS like Sanity/Contentful)
function verifySignature(payload: string, signature: string): boolean {
  const expected = crypto
    .createHmac("sha256", process.env.CMS_WEBHOOK_SECRET!)
    .update(payload)
    .digest("hex")
  return crypto.timingSafeEqual(Buffer.from(signature), Buffer.from(expected))
}

export async function POST(request: Request) {
  const payload = await request.text()
  const signature = (await headers()).get("x-webhook-signature") || ""

  if (!verifySignature(payload, signature)) {
    return Response.json({ error: "Invalid signature" }, { status: 401 })
  }

  const data = JSON.parse(payload)

  // Revalidate based on content type
  if (data.type === "post") {
    revalidateTag(`post-${data.id}`, "max")
    revalidateTag("feed", "max")
  } else if (data.type === "community") {
    revalidateTag(`community-${data.id}`, "max")
  }

  return Response.json({ success: true })
}
```

**Key points:**

- Use Route Handlers for webhooks (external system → your app)
- Use Server Actions with `updateTag()` for user mutations (user → your app)
- `revalidateTag(tag, 'max')` uses stale-while-revalidate (recommended for webhooks)
- `revalidateTag(tag, { expire: 0 })` for immediate expiration when required
- Always verify webhook authenticity with secrets/signatures

---

## Anti-Patterns to Avoid

### 1. Awaiting params at page level (breaks static rendering)

```tsx
// ❌ BAD: Awaiting params at page level can make page dynamic
export default async function Page({ params }) {
  const { id } = await params
  return <PostContent postId={id} />
}

// ✅ GOOD: Pass params Promise, await inside cache boundary
export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  return <PostContent params={params} />
}

async function PostContent({ params }: { params: Promise<{ id: string }> }) {
  "use cache"
  const { id } = await params // Await inside cache
  cacheTag(`post-${id}`)
  // ...
}
```

### 2. Forgetting Suspense for dynamic components

```tsx
// ❌ BAD: Dynamic components NEED Suspense
<CachedContent />
<DynamicContent />  {/* No Suspense - blocks the page! */}

// ✅ GOOD: Dynamic components require Suspense
<CachedContent />  {/* No Suspense needed (but optional) */}
<Suspense fallback={<Skeleton />}>
  <DynamicContent />  {/* Required - streams at request time */}
</Suspense>
```

### 3. Missing generateStaticParams on dynamic routes

```tsx
// ❌ BAD: No generateStaticParams - route may be fully dynamic
// app/post/[id]/page.tsx
export default async function Page({ params }) {
  /* ... */
}

// ✅ GOOD: Always include generateStaticParams
export default async function Page({ params }) {
  /* ... */
}

export function generateStaticParams() {
  return [{ id: "post-1" }, { id: "post-2" }]
}
```

### 4. Data fetching without caching on shared pages

```tsx
// ❌ BAD: No caching, every request hits DB
async function PostList() {
  const posts = await db.posts.findMany()
  return <List posts={posts} />
}

// ✅ GOOD: Cached for all users
async function PostList() {
  "use cache"
  cacheTag("posts")
  cacheLife("minutes")

  const posts = await db.posts.findMany()
  return <List posts={posts} />
}
```

### 5. Server Actions without cache invalidation

```tsx
// ❌ BAD: Cache never updates
"use server"
export async function createPost(data: FormData) {
  await db.posts.create({ data })
  // Missing invalidation!
}

// ✅ GOOD: Invalidate relevant caches
;("use server")

import { updateTag } from "next/cache"

export async function createPost(data: FormData) {
  await db.posts.create({ data })
  updateTag("feed")
  updateTag("posts")
}
```

### 6. Dynamic content without Suspense

```tsx
// ❌ BAD: Blocks entire page
export default async function Page() {
  return (
    <>
      <CachedContent />
      <UserSpecificContent /> {/* Blocks the cached content! */}
    </>
  )
}

// ✅ GOOD: Dynamic content streams separately
export default async function Page() {
  return (
    <>
      <CachedContent />
      <Suspense fallback={<Skeleton />}>
        <UserSpecificContent />
      </Suspense>
    </>
  )
}
```

### 7. Client-side fetching for shared data

```tsx
// ❌ BAD: Every user fetches separately, no caching
"use client"
export function PostList() {
  const [posts, setPosts] = useState([])
  useEffect(() => {
    fetch("/api/posts")
      .then((r) => r.json())
      .then(setPosts)
  }, [])
  return <List posts={posts} />
}

// ✅ GOOD: Server Component with caching
async function PostList() {
  "use cache"
  cacheTag("posts")
  cacheLife("minutes")

  const posts = await db.posts.findMany()
  return <List posts={posts} />
}
```

### 8. Using deprecated route segment configs

```tsx
// ❌ BAD: Old API, doesn't work with Cache Components
export const revalidate = 3600
export const dynamic = 'force-static'

export default async function Page() {
  const data = await fetch(...)
  return <div>{data}</div>
}

// ✅ GOOD: New Cache Components API
export default async function Page() {
  'use cache'
  cacheLife('hours')

  const data = await fetch(...)
  return <div>{data}</div>
}
```
