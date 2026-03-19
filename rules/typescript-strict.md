---
title: Strict TypeScript Configuration
impact: HIGH
tags: [typescript, configuration, type-safety]
---

# Strict TypeScript Configuration

Use strict TypeScript configuration with modern ESM module resolution, path aliases, and Bun-specific optimizations. This ensures type safety, better editor support, and optimal runtime performance.

## Why

- **Type Safety**: Strict mode catches more errors at compile time
- **ESM Compatibility**: Modern module system with proper tree-shaking
- **Developer Experience**: Path aliases clean up imports
- **Performance**: `noEmit` with `verbatimModuleSyntax` for better bundling
- **Bun Optimization**: Bun-specific types and runtime features

## Strict TypeScript Config

Configure TypeScript for strict type checking and modern ESM.

```json
// Bad: Permissive TypeScript config
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "strict": false,
    "skipLibCheck": false,
    "noEmit": false
  }
}

// Good: Strict TypeScript with modern ESM
{
  "compilerOptions": {
    "target": "ESNext",
    "lib": ["ESNext"],
    "module": "Preserve",
    "moduleDetection": "force",
    "jsx": "react-jsx",
    "allowJs": true,
    "checkJs": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "verbatimModuleSyntax": true,
    "noEmit": true,
    "strict": true,
    "skipLibCheck": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "noPropertyAccessFromIndexSignature": false,
    "esModuleInterop": true,
    "isolatedModules": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "paths": {
      "*": ["./*"],
      "@/*": ["./src/*"]
    },
    "types": ["bun"]
  },
  "include": ["src/**/*", "*.config.ts"],
  "exclude": ["node_modules", "dist"]
}
```

## Path Aliases

Configure path aliases for clean imports using `@/` prefix.

```json
// tsconfig.json
{
  "compilerOptions": {
    "paths": {
      "*": ["./*"],
      "@/*": ["./src/*"],
      "@/components/*": ["./src/components/*"],
      "@/lib/*": ["./src/lib/*"],
      "@/hooks/*": ["./src/hooks/*"],
      "@/db/*": ["./src/db/*"]
    }
  }
}
```

```tsx
// Bad: Relative path imports
import { Button } from '../../components/ui/button'
import { useMobile } from '../../hooks/use-mobile'
import { cn } from '../../lib/utils'
import { userModel } from '../../db/models/user'

// Good: Path alias imports
import { Button } from '@/components/ui/button'
import { useMobile } from '@/hooks/use-mobile'
import { cn } from '@/lib/utils'
import { userModel } from '@/db/models/user'
```

## verbatimModuleSyntax

Use `verbatimModuleSyntax` to preserve ESM imports/exports exactly as written.

```typescript
// Bad: Type-only imports without 'type' keyword
import { User } from './types'
import type { User } from './types'  // This gets erased

// Good: Explicit type imports with verbatimModuleSyntax
import type { User } from './types'
import { type User, createUser } from './types'
```

## noUncheckedIndexedAccess

Enable stricter array and object access checking.

```typescript
// Bad: Potential undefined not caught
const users: User[] = fetchUsers()
const firstUser = users[0]  // Could be undefined, no error
console.log(firstUser.name)  // Runtime error possible

// Good: With noUncheckedIndexedAccess, must check for undefined
const users: User[] = fetchUsers()
const firstUser = users[0]
if (firstUser) {
  console.log(firstUser.name)
}

// Or use optional chaining
console.log(firstUser?.name)
```

## Bun Types

Include Bun types for Bun-specific APIs.

```json
{
  "compilerOptions": {
    "types": ["bun"]
  }
}
```

```typescript
// Now Bun APIs are typed
const file = Bun.file('./data.json')
const content = await file.text()

Bun.serve({
  port: 3000,
  fetch(req) {
    return new Response('Hello')
  }
})
```

## Rules

1. Always use `"strict": true` for maximum type safety
2. Use `"module": "Preserve"` with `"moduleResolution": "bundler"` for modern ESM
3. Enable `"verbatimModuleSyntax": true` to preserve exact import/export syntax
4. Set `"noEmit": true` and let the bundler handle compilation
5. Enable `"noUncheckedIndexedAccess": true` for safer array/object access
6. Configure path aliases with `@/` prefix for clean imports
7. Include `"types": ["bun"]` when using Bun runtime
8. Use `"jsx": "react-jsx"` for automatic JSX runtime
9. Keep `"noUnusedLocals": false` to allow debugging variables
10. Use `"allowImportingTsExtensions": true` for direct .ts imports
