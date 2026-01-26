---
name: next-hydration-error
description: Diagnose and fix React hydration mismatch errors in Next.js applications
---

# Hydration Error

Diagnose and fix React hydration mismatch errors in Next.js applications.

## Instructions

1. **Identify the error**: Look for messages like:
   - "Hydration failed because the initial UI does not match"
   - "Text content does not match server-rendered HTML"

2. **Use the error overlay**: In development, click the hydration error to see the server/client diff.

3. **Common causes**:
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

   ### Browser-only rendering
   When content depends on browser state:

   ```tsx
   'use client'
   import { useState, useEffect } from 'react'

   export function ClientOnly({ children }: { children: React.ReactNode }) {
     const [mounted, setMounted] = useState(false)
     useEffect(() => setMounted(true), [])
     return mounted ? children : null
   }
   ```

   ### Date/Time
   Server and client may be in different timezones:

   ```tsx
   // ❌ Causes mismatch
   <span>{new Date().toLocaleString()}</span>

   // ✅ Render on client only
   'use client'
   const [time, setTime] = useState<string>()
   useEffect(() => setTime(new Date().toLocaleString()), [])
   ```

   ### useId for unique IDs
   ```tsx
   import { useId } from 'react'

   function Input() {
     const id = useId()
     return <input id={id} />
   }
   ```
