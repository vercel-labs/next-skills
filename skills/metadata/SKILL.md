---
name: metadata
description: Add SEO metadata to pages using the Next.js Metadata API (Server Components only)
argument-hint: "[page-path]"
---

# Metadata

Add SEO metadata to Next.js pages using the Metadata API.

## Important: Server Components Only

The `metadata` object and `generateMetadata` function are **only supported in Server Components**. They cannot be used in Client Components.

If the target page or layout has `'use client'` directive:
1. Remove the `'use client'` directive if possible, move client-side logic into child Client Components
2. Or extract metadata to a parent Server Component layout
3. Or split the file: keep the page as a Server Component with metadata, import Client Components for interactive parts

## Instructions

1. **Identify the target**: Determine which `page.tsx` or `layout.tsx` needs metadata. Verify it's a Server Component (no `'use client'` directive).

2. **Choose approach**:
   - **Static Metadata** - Export a `metadata` object for fixed values
   - **Dynamic Metadata** - Export `generateMetadata` function when values depend on params or fetched data

3. **Static metadata**:
   ```tsx
   import type { Metadata } from 'next'

   export const metadata: Metadata = {
     title: 'Page Title',
     description: 'Page description for search engines',
   }
   ```

4. **Dynamic metadata**:
   ```tsx
   import type { Metadata } from 'next'

   type Props = { params: Promise<{ slug: string }> }

   export async function generateMetadata({ params }: Props): Promise<Metadata> {
     const { slug } = await params
     const post = await getPost(slug)
     return { title: post.title, description: post.description }
   }
   ```

5. **Avoid duplicate fetches**: Use React `cache()` when the same data is needed for both metadata and the page component:
   ```tsx
   import { cache } from 'react'

   export const getPost = cache(async (slug: string) => {
     return await db.posts.findFirst({ where: { slug } })
   })
   ```

6. **Title templates** in root layout for consistent naming:
   ```tsx
   export const metadata: Metadata = {
     title: { default: 'Site Name', template: '%s | Site Name' },
   }
   ```

7. **File-based metadata**: Place in `app/` directory:
   - `favicon.ico`, `icon.png` - Favicons
   - `opengraph-image.png` - Default OG image (can be generated with `ImageResponse`)
   - `robots.ts` - Search engine directives
   - `sitemap.ts` - XML sitemap

## Reference

- Metadata: https://nextjs.org/docs/app/building-your-application/optimizing/metadata
- generateMetadata: https://nextjs.org/docs/app/api-reference/functions/generate-metadata
