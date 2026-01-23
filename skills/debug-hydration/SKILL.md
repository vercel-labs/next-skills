---
description: Diagnose and fix React hydration mismatch errors in Next.js applications
---

# Debug Hydration Errors

Diagnose and fix React hydration mismatch errors in Next.js applications.

## Instructions

1. **Identify the error**: Look for messages like:
   - "Hydration failed because the initial UI does not match"
   - "Text content does not match server-rendered HTML"

2. **Common causes and fixes**:

   ### Browser-only APIs
   Code using `window`, `document`, `localStorage` renders differently on server vs client.

   **Fix**: Check for client-side or use dynamic import with `ssr: false`:
   ```tsx
   'use client'
   const [mounted, setMounted] = useState(false)
   useEffect(() => setMounted(true), [])
   if (!mounted) return null
   ```

   ### Date/Time rendering
   Server and client may have different timezones.

   **Fix**: Format dates on client only using `useEffect`.

   ### Invalid HTML nesting
   Invalid DOM nesting (e.g., `<p>` inside `<p>`, `<div>` inside `<p>`).

   **Fix**: Correct the HTML structure.

   ### Random values or IDs
   Using `Math.random()` or `Date.now()` for keys.

   **Fix**: Use `useId()` hook for stable identifiers.

   ### Third-party scripts
   Scripts that modify DOM during hydration.

   **Fix**: Use `next/script` with `strategy="afterInteractive"`.

   ### Browser extensions
   Extensions modifying DOM before hydration.

   **Fix**: Use `suppressHydrationWarning` sparingly on affected elements.

3. **Debug with React DevTools**: Enable "Highlight updates" to see re-renders.

4. **Use error overlay**: In development, click the hydration error to see the server/client diff.

## Reference

- React Hydration Errors: https://nextjs.org/docs/messages/react-hydration-error
- Client Components: https://nextjs.org/docs/app/building-your-application/rendering/client-components
