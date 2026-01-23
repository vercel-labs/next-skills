---
name: hydration-error
description: Diagnose and fix React hydration mismatch errors in Next.js applications
---

# Hydration Error

Diagnose and fix React hydration mismatch errors in Next.js applications.

## Instructions

1. **Identify the error**: Look for messages like:
   - "Hydration failed because the initial UI does not match"
   - "Text content does not match server-rendered HTML"

2. **Use the error overlay**: In development, click the hydration error to see the server/client diff.

3. **Common causes**: See the `react-best-practices` skill for detailed patterns:
   - Browser-only APIs (`window`, `document`, `localStorage`)
   - Date/Time rendering (timezone differences)
   - Invalid HTML nesting
   - Random values or IDs (use `useId()` hook)

4. **Next.js specific fixes**:

   ### Third-party scripts
   Scripts that modify DOM during hydration.

   **Fix**: Use `next/script` with `strategy="afterInteractive"`:
   ```tsx
   import Script from 'next/script'

   export default function Page() {
     return (
       <>
         <Script
           src="https://example.com/script.js"
           strategy="afterInteractive"
         />
       </>
     )
   }
   ```

## Reference

- React Hydration Errors: https://nextjs.org/docs/messages/react-hydration-error
- Client Components: https://nextjs.org/docs/app/building-your-application/rendering/client-components
- next/script: https://nextjs.org/docs/app/api-reference/components/script
