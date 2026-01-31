# Forum/Reddit-Style Patterns

Common caching patterns for high-traffic content sites.

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
import { Suspense } from 'react'
import { cacheTag, cacheLife, updateTag } from 'next/cache'

// CACHED FEED - Same for all anonymous users
async function PublicFeed() {
  'use cache'
  cacheTag('feed')
  cacheLife('minutes')

  const posts = await db.posts.findMany({
    orderBy: { createdAt: 'desc' },
    take: 50,
  })
  return <PostList posts={posts} />
}

// PERSONALIZED FEED - Different per user, streams
async function PersonalizedFeed() {
  const session = await getSession()
  const posts = await db.posts.findMany({
    where: { authorId: { in: session.following } },
    orderBy: { createdAt: 'desc' },
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
'use server'

import { updateTag } from 'next/cache'

export async function createPost(data: FormData) {
  await db.posts.create({ data })
  updateTag('feed') // New post appears immediately
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
import { Suspense } from 'react'
import { cacheTag, cacheLife, updateTag } from 'next/cache'

// POST CONTENT - Cached, updates immediately on edit
async function PostContent({ postId }: { postId: string }) {
  'use cache'
  cacheTag(`post-${postId}`)
  cacheLife('days')

  const post = await db.posts.findUnique({
    where: { id: postId },
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

// COMMENTS - Dynamic, streams after shell
async function Comments({ postId }: { postId: string }) {
  const comments = await db.comments.findMany({
    where: { postId },
    orderBy: { createdAt: 'desc' },
    include: { author: true },
  })
  return <CommentList comments={comments} />
}

// PAGE COMPOSITION
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

// app/actions.ts - Server Actions
'use server'

import { updateTag } from 'next/cache'

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

- Post content: cached with long `cacheLife('days')`
- Comments: dynamic in `<Suspense>` - always fresh
- Edit post: `updateTag()` for immediate update

---

## Pattern 3: Upvotes/Reactions

Upvotes can be cached with very short lifetime or kept dynamic.

```tsx
import { cacheTag, cacheLife, updateTag } from 'next/cache'

// OPTION A: Short-cached upvotes (slight delay acceptable)
async function UpvoteCount({ postId }: { postId: string }) {
  'use cache'
  cacheTag(`post-${postId}-votes`)
  cacheLife('seconds') // 1s stale, refresh in background

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
'use server'

import { updateTag } from 'next/cache'

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

Public profiles are cached. Own profile is dynamic for edit capabilities.

```tsx
import { Suspense } from 'react'
import { cacheTag, cacheLife, updateTag } from 'next/cache'

// PUBLIC PROFILE - Cached
async function PublicProfile({ userId }: { userId: string }) {
  'use cache'
  cacheTag(`user-${userId}`)
  cacheLife('hours')

  const user = await db.users.findUnique({
    where: { id: userId },
    include: { _count: { select: { posts: true } } },
  })

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.bio}</p>
      <p>{user._count.posts} posts</p>
    </div>
  )
}

// USER'S POSTS - Cached separately
async function UserPosts({ userId }: { userId: string }) {
  'use cache'
  cacheTag(`user-${userId}-posts`)
  cacheLife('minutes')

  const posts = await db.posts.findMany({
    where: { authorId: userId },
    orderBy: { createdAt: 'desc' },
  })
  return <PostList posts={posts} />
}

// OWN PROFILE EDIT SECTION - Dynamic
async function ProfileEditSection() {
  const session = await getSession()
  const user = await db.users.findUnique({ where: { id: session.userId } })
  return <ProfileEditForm user={user} />
}

export default async function ProfilePage({ params }) {
  const { userId } = await params
  const session = await getSession()
  const isOwnProfile = session?.userId === userId

  return (
    <>
      <PublicProfile userId={userId} />
      <UserPosts userId={userId} />

      {isOwnProfile && (
        <Suspense fallback={<EditSkeleton />}>
          <ProfileEditSection />
        </Suspense>
      )}
    </>
  )
}

// app/actions.ts - Server Action
'use server'

import { updateTag } from 'next/cache'

export async function updateProfile(data: FormData) {
  const session = await getSession()
  await db.users.update({ where: { id: session.userId }, data })
  updateTag(`user-${session.userId}`)
}
```

**Key points:**

- Public profile: cached for all viewers
- Own profile: edit section streams in `<Suspense>`
- Separate cache tags for profile vs posts

---

## Pattern 5: Community Pages with Sort Orders (Subshells)

Community/subreddit pages with different sort orders (hot, new, top) that share a common layout. The community header is cached once and reused across all sort variations.

**Route structure:**

```
app/r/[community]/
├── layout.tsx          # Cached community header (subshell)
├── page.tsx            # Default (hot) sort
└── [sort]/
    └── page.tsx        # Different sorts: new, top, rising
```

```tsx
// app/r/[community]/layout.tsx
// SHARED LAYOUT - Cached community header becomes a subshell
import { Suspense } from 'react'
import { cacheTag, cacheLife } from 'next/cache'

async function CommunityHeader({ community }: { community: string }) {
  'use cache'
  cacheTag(`community-${community}`)
  cacheLife('days')

  const data = await db.communities.findUnique({
    where: { slug: community },
  })

  return (
    <header>
      <h1>r/{data.name}</h1>
      <p>{data.description}</p>
      <p>{data.memberCount} members</p>
      <nav>
        <a href={`/r/${community}`}>Hot</a>
        <a href={`/r/${community}/new`}>New</a>
        <a href={`/r/${community}/top`}>Top</a>
      </nav>
    </header>
  )
}

export default async function CommunityLayout({
  children,
  params,
}: {
  children: React.ReactNode
  params: Promise<{ community: string }>
}) {
  const { community } = await params
  return (
    <>
      <CommunityHeader community={community} />
      {/* Suspense creates subshell boundary - post lists stream in */}
      <Suspense fallback={<PostListSkeleton />}>{children}</Suspense>
    </>
  )
}
```

```tsx
// app/r/[community]/page.tsx (default = hot)
import { cacheTag, cacheLife } from 'next/cache'

async function HotPosts({ community }: { community: string }) {
  'use cache'
  cacheTag(`community-${community}-hot`)
  cacheLife('minutes')

  const posts = await db.posts.findMany({
    where: { communitySlug: community },
    orderBy: { hotScore: 'desc' },
    take: 50,
  })
  return <PostList posts={posts} />
}

export default async function CommunityHotPage({ params }) {
  const { community } = await params
  return <HotPosts community={community} />
}
```

```tsx
// app/r/[community]/[sort]/page.tsx
import { cacheTag, cacheLife, updateTag } from 'next/cache'

type SortOrder = 'new' | 'top' | 'rising'

const sortConfigs: Record<SortOrder, { orderBy: any; cacheProfile: string }> = {
  new: { orderBy: { createdAt: 'desc' }, cacheProfile: 'seconds' },
  top: { orderBy: { upvotes: 'desc' }, cacheProfile: 'minutes' },
  rising: { orderBy: { risingScore: 'desc' }, cacheProfile: 'seconds' },
}

async function SortedPosts({
  community,
  sort,
}: {
  community: string
  sort: SortOrder
}) {
  'use cache'
  cacheTag(`community-${community}-${sort}`)
  cacheLife(sortConfigs[sort].cacheProfile)

  const posts = await db.posts.findMany({
    where: { communitySlug: community },
    orderBy: sortConfigs[sort].orderBy,
    take: 50,
  })
  return <PostList posts={posts} />
}

export default async function CommunitySortPage({ params }) {
  const { community, sort } = await params
  return <SortedPosts community={community} sort={sort as SortOrder} />
}

// Pre-render popular communities with common sorts
export async function generateStaticParams() {
  const popularCommunities = await db.communities.findMany({
    orderBy: { memberCount: 'desc' },
    take: 50,
    select: { slug: true },
  })

  const sorts: SortOrder[] = ['new', 'top', 'rising']

  return popularCommunities.flatMap((c) =>
    sorts.map((sort) => ({ community: c.slug, sort }))
  )
}
```

**How subshells work:**

When you visit `/r/programming/new`:

1. Community header (`/r/programming/[sort]` subshell) served instantly
2. Post list for "new" sort streams in

When you then visit `/r/programming/top`:

1. Same community header reused (already cached)
2. Only the "top" post list needs to load

**Generated at build time:**

- `/r/programming` (default hot)
- `/r/programming/new`
- `/r/programming/top`
- `/r/programming/rising`
- ...for top 50 communities

**On-demand for other communities:**

- First visit renders and caches
- Subsequent visits are instant

---

## Pattern 6: Pre-rendering Popular Posts

Use `generateStaticParams` to pre-render the most popular posts at build time. All other posts render on-demand and get cached.

```tsx
// app/post/[id]/page.tsx
import { Suspense } from 'react'
import { cacheTag, cacheLife, updateTag } from 'next/cache'

async function PostContent({ postId }: { postId: string }) {
  'use cache'
  cacheTag(`post-${postId}`)
  cacheLife('days')

  const post = await db.posts.findUnique({
    where: { id: postId },
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

async function Comments({ postId }: { postId: string }) {
  const comments = await db.comments.findMany({
    where: { postId },
    orderBy: { createdAt: 'desc' },
  })
  return <CommentList comments={comments} />
}

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

// Pre-render top 100 most viewed posts at build time
export async function generateStaticParams() {
  const popularPosts = await db.posts.findMany({
    orderBy: { views: 'desc' },
    take: 100,
    select: { id: true },
  })
  return popularPosts.map((post) => ({ id: post.id }))
}

// app/actions.ts - Server Action
'use server'

import { updateTag } from 'next/cache'

export async function editPost(postId: string, data: FormData) {
  await db.posts.update({ where: { id: postId }, data })
  updateTag(`post-${postId}`)
}
```

**Key points:**

- Top 100 posts: Pre-rendered at build, served instantly from CDN
- Other posts: Rendered on first visit, then cached with `cacheLife('days')`
- Both get immediate updates via `updateTag()` after edits

---

## Pattern 7: Webhook-triggered Revalidation

When external systems (CMS, payment processors, third-party APIs) need to notify your app of changes, use a Route Handler with `revalidateTag`. This is different from Server Actions - webhooks come from external systems, not user interactions.

```tsx
// app/api/webhook/route.ts
import type { NextRequest } from 'next/server'
import { revalidateTag } from 'next/cache'

// Secret token to verify webhook is from trusted source
const WEBHOOK_SECRET = process.env.WEBHOOK_SECRET

export async function POST(request: NextRequest) {
  // Verify the webhook is authentic
  const authHeader = request.headers.get('authorization')
  if (authHeader !== `Bearer ${WEBHOOK_SECRET}`) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const payload = await request.json()

  // Handle different event types from your CMS/external system
  switch (payload.event) {
    case 'post.created':
    case 'post.updated':
      // Revalidate the specific post and feeds
      revalidateTag(`post-${payload.postId}`, 'max')
      revalidateTag('feed', 'max')
      break

    case 'post.deleted':
      revalidateTag(`post-${payload.postId}`, 'max')
      revalidateTag('feed', 'max')
      break

    case 'community.updated':
      revalidateTag(`community-${payload.communityId}`, 'max')
      break

    default:
      return Response.json({ error: 'Unknown event' }, { status: 400 })
  }

  return Response.json({ revalidated: true, event: payload.event })
}
```

### When to use `revalidateTag` vs `updateTag`

| Scenario | Function | Why |
|----------|----------|-----|
| User edits their own post (Server Action) | `updateTag()` | Immediate - user needs to see their changes |
| CMS publishes new content (Webhook) | `revalidateTag(tag, 'max')` | Stale-while-revalidate - slight delay acceptable |
| Webhook needs immediate expiration | `revalidateTag(tag, { expire: 0 })` | When external system requires immediate invalidation |

### Example: CMS Integration

```tsx
// app/api/cms-webhook/route.ts
import { revalidateTag } from 'next/cache'
import { headers } from 'next/headers'
import crypto from 'crypto'

// Verify webhook signature (example for a CMS like Sanity/Contentful)
function verifySignature(payload: string, signature: string): boolean {
  const expected = crypto
    .createHmac('sha256', process.env.CMS_WEBHOOK_SECRET!)
    .update(payload)
    .digest('hex')
  return crypto.timingSafeEqual(Buffer.from(signature), Buffer.from(expected))
}

export async function POST(request: Request) {
  const payload = await request.text()
  const signature = (await headers()).get('x-webhook-signature') || ''

  if (!verifySignature(payload, signature)) {
    return Response.json({ error: 'Invalid signature' }, { status: 401 })
  }

  const data = JSON.parse(payload)

  // Revalidate based on content type
  if (data.type === 'post') {
    revalidateTag(`post-${data.id}`, 'max')
    revalidateTag('feed', 'max')
  } else if (data.type === 'community') {
    revalidateTag(`community-${data.id}`, 'max')
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

### 1. Data fetching without caching on shared pages

```tsx
// ❌ BAD: No caching, every request hits DB
async function PostList() {
  const posts = await db.posts.findMany()
  return <List posts={posts} />
}

// ✅ GOOD: Cached for all users
async function PostList() {
  'use cache'
  cacheTag('posts')
  cacheLife('minutes')

  const posts = await db.posts.findMany()
  return <List posts={posts} />
}
```

### 2. Server Actions without cache invalidation

```tsx
// ❌ BAD: Cache never updates
'use server'
export async function createPost(data: FormData) {
  await db.posts.create({ data })
  // Missing invalidation!
}

// ✅ GOOD: Invalidate relevant caches
'use server'

import { updateTag } from 'next/cache'

export async function createPost(data: FormData) {
  await db.posts.create({ data })
  updateTag('feed')
  updateTag('posts')
}
```

### 3. Dynamic content without Suspense

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

### 4. Client-side fetching for shared data

```tsx
// ❌ BAD: Every user fetches separately, no caching
'use client'
export function PostList() {
  const [posts, setPosts] = useState([])
  useEffect(() => {
    fetch('/api/posts')
      .then((r) => r.json())
      .then(setPosts)
  }, [])
  return <List posts={posts} />
}

// ✅ GOOD: Server Component with caching
async function PostList() {
  'use cache'
  cacheTag('posts')
  cacheLife('minutes')

  const posts = await db.posts.findMany()
  return <List posts={posts} />
}
```

### 5. Using deprecated route segment configs

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
