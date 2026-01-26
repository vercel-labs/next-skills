---
name: next-og-image
description: Generate Open Graph images dynamically in Next.js using next/og
argument-hint: "[route-path]"
---

# OG Image Generation

Generate dynamic Open Graph images in Next.js using the built-in `next/og` package.

## Important: Use next/og

**Always use `next/og` instead of installing `@vercel/og`**. The `next/og` package is built into Next.js and provides the same `ImageResponse` API without requiring an additional dependency.

```tsx
// Correct
import { ImageResponse } from 'next/og'

// Avoid - don't install @vercel/og separately
// import { ImageResponse } from '@vercel/og'
```

## Important: searchParams Not Available

**`searchParams` cannot be accessed in OG image routes**. The OG image generation runs at build time or is cached, so dynamic search parameters are not available.

If you need dynamic content in OG images:
- Use route parameters (dynamic segments like `[slug]`) instead
- Pass data through the URL path, not query strings

## Important: Avoid Edge Runtime

**Do not use Edge runtime for OG image routes**. Use the default Node.js runtime instead.

```tsx
// Avoid this
export const runtime = 'edge'

// Use default Node.js runtime (no export needed)
```

The Node.js runtime provides better compatibility and fewer limitations for image generation.

## Instructions

1. **Create the OG image route**: Create `opengraph-image.tsx` in the route segment where you want the OG image.

2. **Basic static OG image**:
   ```tsx
   // app/opengraph-image.tsx
   import { ImageResponse } from 'next/og'

   export const alt = 'Site Name'
   export const size = { width: 1200, height: 630 }
   export const contentType = 'image/png'

   export default function Image() {
     return new ImageResponse(
       (
         <div
           style={{
             fontSize: 128,
             background: 'white',
             width: '100%',
             height: '100%',
             display: 'flex',
             alignItems: 'center',
             justifyContent: 'center',
           }}
         >
           Hello World
         </div>
       ),
       { ...size }
     )
   }
   ```

3. **Dynamic OG image with route params**:
   ```tsx
   // app/blog/[slug]/opengraph-image.tsx
   import { ImageResponse } from 'next/og'

   export const alt = 'Blog Post'
   export const size = { width: 1200, height: 630 }
   export const contentType = 'image/png'

   type Props = { params: Promise<{ slug: string }> }

   export default async function Image({ params }: Props) {
     const { slug } = await params
     const post = await getPost(slug)

     return new ImageResponse(
       (
         <div
           style={{
             fontSize: 48,
             background: 'linear-gradient(to bottom, #1a1a1a, #333)',
             color: 'white',
             width: '100%',
             height: '100%',
             display: 'flex',
             flexDirection: 'column',
             alignItems: 'center',
             justifyContent: 'center',
             padding: 48,
           }}
         >
           <div style={{ fontSize: 64, fontWeight: 'bold' }}>{post.title}</div>
           <div style={{ marginTop: 24, opacity: 0.8 }}>{post.description}</div>
         </div>
       ),
       { ...size }
     )
   }
   ```

4. **Using custom fonts**:
   ```tsx
   import { ImageResponse } from 'next/og'
   import { join } from 'path'
   import { readFile } from 'fs/promises'

   export default async function Image() {
     // process.cwd() returns the app root directory
     const fontPath = join(process.cwd(), 'assets/fonts/Inter-Bold.ttf')
     const fontData = await readFile(fontPath)

     return new ImageResponse(
       (
         <div style={{ fontFamily: 'Inter', fontSize: 64 }}>
           Custom Font Text
         </div>
       ),
       {
         width: 1200,
         height: 630,
         fonts: [{ name: 'Inter', data: fontData, style: 'normal' }],
       }
     )
   }
   ```

5. **File naming conventions**:
   - `opengraph-image.tsx` - For Open Graph (Facebook, LinkedIn, etc.)
   - `twitter-image.tsx` - For Twitter/X cards (optional, falls back to OG image)

## Styling Notes

ImageResponse uses a subset of CSS with Flexbox layout:
- Use `display: 'flex'` for layouts
- Most common CSS properties are supported
- No CSS Grid support
- Styles must be inline objects, not CSS strings
