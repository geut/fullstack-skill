---
name: fullstack-skill
description: This skill is oriented to work with React, Tailwind and Astro projects. It also includes fullstack projects with elysia and some database best practices. Based on the GEUT team learnings. This skill should be used when writing, reviewing, or refactoring any fullstack code to ensure optimal performance and clarity (simple code not simpler).
---

# GEUT Fullstack Skill

GEUT Fullstack Best Practices Skill. Helps you create simple, readable and maintainable fullstack applications. The GEUT way. Contains 9 rules across 28 categories. 

## When to Apply

Reference these guidelines when:
- Writing new React components with shadcn/ui
- Configuring Tailwind CSS v4 with CSS-first approach
- Setting up TypeScript with strict mode and path aliases
- Building APIs with Elysia.js in Astro applications
- Implementing database models with Drizzle ORM
- Creating custom error handling and error hierarchies
- Writing tests with Bun's test runner
- Configuring linting and formatting with OXC tooling
- Starting a new Astro project from scratch
- Implementing data fetching patterns
- Reviewing code for performance issues
- Refactoring existing React code
- Optimizing bundle size or load times
- Setting up fullstack applications

## How to Use

Read individual rule files for detailed explanations and code examples. Check the `rules` directory.

Each rule file contains:
- Brief explanation of why it matters
- Incorrect code example with explanation
- Correct code example with explanation
- Additional context and references

## Rules Summary

### Coding Style @rules/coding-style

GEUT Style Fullstack code alignment. Rules for a clearer code.

### Fullstack UI/UX (HIGH)

#### shadcn/ui Component Patterns @rules/shadcn-components

Use consistent patterns for React components: `data-slot` attributes, CVA variants, `cn()` utility for class merging, and `asChild` composition. Ensures type-safe, composable, and maintainable UI components.

```tsx
// data-slot for debugging + CVA for variants + cn() for merging
function Button({ className, variant, size, asChild }) {
  const Comp = asChild ? Slot : 'button'
  return (
    <Comp
      data-slot="button"
      className={cn(buttonVariants({ variant, size, className }))}
    />
  )
}
```

#### Tailwind CSS v4 Configuration @rules/tailwind-v4

Use CSS-first Tailwind v4 with OKLCH colors and semantic naming. Configure via CSS imports instead of JavaScript for better performance and maintainability.

```css
@import "tailwindcss";
@theme inline {
  --color-background: var(--background);
}
:root {
  --background: oklch(1 0 0);
}
```

### TypeScript & Code Quality (HIGH)

#### Strict TypeScript Configuration @rules/typescript-strict

Use strict TypeScript with `verbatimModuleSyntax`, path aliases (`@/`), and Bun types. Ensures maximum type safety and clean imports across the codebase.

```json
{
  "compilerOptions": {
    "strict": true,
    "verbatimModuleSyntax": true,
    "noUncheckedIndexedAccess": true,
    "paths": { "@/*": ["./src/*"] }
  }
}
```

### TypeScript & Code Quality (MEDIUM)

#### Error Handling Patterns @rules/error-handling

Use custom error hierarchies with NonRetriableError for permanent failures. Organize errors by domain and include rich context in error causes.

```typescript
export class NonRetriableError extends AppError {}
throw new NonRetriableError('Invalid email', { cause: { email } })
```

#### Bun Testing Patterns @rules/testing-patterns

Use Bun's built-in test runner with factory patterns for multi-adapter testing. Faster than Jest/Vitest with native TypeScript support.

```typescript
import { describe, it, expect } from 'bun:test'
describe('Feature', () => {
  it('should work', () => {
    expect(result).toBe(true)
  }, { timeout: 5000 })
})
```

### Full-Stack Patterns (MEDIUM)

#### Elysia.js API Patterns @rules/elysia-api

Build type-safe APIs with Elysia.js: plugin factories, TypeBox schemas, macros for cross-cutting concerns, and Eden client for frontend type safety.

```typescript
const api = createPlugin()
  .post('/users', handler, {
    body: t.Object({ email: t.String() }),
    response: { 200: UserSchema }
  })
```

#### Database Patterns @rules/database-patterns

Use Drizzle ORM with soft delete architecture and model factories. Automatic filtering, reusable column patterns, and transaction support.

```typescript
const userModel = createModel(users, db)
await userModel.getById(id)      // Excludes deleted
await userModel.delete(id)        // Soft delete
await userModel.delete(id, { hard: true })  // Hard delete
```

### Tooling (MEDIUM)

#### OXC Tooling @rules/oxc-tooling

Use Oxlint and Oxfmt as fast Rust-based alternatives to ESLint/Prettier. 50-100x performance improvement with experimental import sorting.

```json
{
  "semi": false,
  "singleQuote": true,
  "experimentalSortImports": {
    "enabled": true,
    "internalPattern": ["@/"]
  }
}
```

