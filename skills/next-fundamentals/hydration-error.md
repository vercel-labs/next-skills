# Hydration Errors

Diagnose and fix React hydration mismatch errors.

## Error Signs

- "Hydration failed because the initial UI does not match"
- "Text content does not match server-rendered HTML"

## Debugging

In development, click the hydration error to see the server/client diff.

## Common Causes and Fixes

### Browser-only APIs

```tsx
// ❌ Causes mismatch - window doesn't exist on server
<div>{window.innerWidth}</div>

// ✅ Use client component with mounted check
'use client'
import { useState, useEffect } from 'react'

export function ClientOnly({ children }: { children: React.ReactNode }) {
  const [mounted, setMounted] = useState(false)
  useEffect(() => setMounted(true), [])
  return mounted ? children : null
}
```

### Date/Time Rendering

Server and client may be in different timezones:

```tsx
// ❌ Causes mismatch
<span>{new Date().toLocaleString()}</span>

// ✅ Render on client only
'use client'
const [time, setTime] = useState<string>()
useEffect(() => setTime(new Date().toLocaleString()), [])
```

### Random Values or IDs

```tsx
// ❌ Random values differ between server and client
<div id={Math.random().toString()}>

// ✅ Use useId hook
import { useId } from 'react'

function Input() {
  const id = useId()
  return <input id={id} />
}
```

### Invalid HTML Nesting

```tsx
// ❌ Invalid - div inside p
<p><div>Content</div></p>

// ❌ Invalid - p inside p
<p><p>Nested</p></p>

// ✅ Valid nesting
<div><p>Content</p></div>
```

### Third-party Scripts

Scripts that modify DOM during hydration.

```tsx
// ✅ Use next/script with afterInteractive
import Script from 'next/script'

export default function Page() {
  return (
    <Script
      src="https://example.com/script.js"
      strategy="afterInteractive"
    />
  )
}
```
