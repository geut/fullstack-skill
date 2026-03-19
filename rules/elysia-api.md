---
title: Elysia.js API Patterns
impact: MEDIUM
tags: [api, elysia, fullstack]
---

# Elysia.js API Patterns

Use Elysia.js with type-safe routes, plugin factories, and macros for building robust APIs in full-stack Astro applications. This ensures end-to-end type safety and maintainable backend code.

## Why

- **Type Safety**: End-to-end type inference from route definitions to client usage
- **Performance**: Bun-native framework with minimal overhead
- **Composability**: Plugin system enables reusable, testable route modules
- **Developer Experience**: TypeBox validation with automatic type inference
- **Full-Stack Context**: Seamless integration with Astro frontend via Eden client

## Plugin Factory Pattern

Create reusable Elysia plugins with consistent setup using a factory function.

```typescript
// Bad: Inline plugin creation without standard setup
const authRoutes = new Elysia({ prefix: '/auth' })
  .post('/login', () => { /* ... */ })
  .post('/logout', () => { /* ... */ })

// Good: Factory function with consistent middleware
// src/api/plugins/common/elysia.ts
import Elysia from 'elysia'
import { requestIdPlugin } from './request-id'
import { loggerPlugin } from './logger'

export function createPlugin({ name }: { name?: string } = {}) {
  return new Elysia({ name })
    .use(requestIdPlugin)
    .use(loggerPlugin)
}

// src/api/auth.ts
import { createPlugin } from './plugins/common/elysia'

const authApi = createPlugin({ name: 'auth' })
  .use(signUp)
  .use(signIn)
  .use(signOut)

export default authApi
```

## Type-Safe Route Definitions

Define routes with TypeBox schemas for automatic validation and type inference.

```typescript
// Bad: Manual validation without types
app.post('/users', async (ctx) => {
  const { email, name } = ctx.body as any
  if (!email || !email.includes('@')) {
    ctx.set.status = 400
    return { error: 'Invalid email' }
  }
  // ...
})

// Good: TypeBox schemas with automatic validation
import { t } from 'elysia'

const signUpSchema = {
  body: t.Object({
    email: t.String({ format: 'email' }),
    password: t.String({ minLength: 8 }),
    name: t.String({ minLength: 1, maxLength: 100 })
  }),
  response: {
    200: t.Object({
      user: t.Object({
        id: t.String(),
        email: t.String(),
        name: t.String()
      })
    }),
    400: t.Object({ error: t.String() })
  },
  detail: {
    summary: 'Sign up new user',
    tags: ['Auth']
  }
}

export default createPlugin()
  .post('/sign-up', async ({ body, set, log }) => {
    // body is fully typed: { email: string, password: string, name: string }
    const user = await createUser(body)
    
    log.info({ userId: user.id }, 'User created')
    
    return { user }
  }, signUpSchema)
```

## Macros for Reusable Logic

Use Elysia macros to inject common behavior across routes.

```typescript
// Bad: Repeating auth check in every route
app.get('/profile', async (ctx) => {
  const session = await getSession(ctx.headers.authorization)
  if (!session) {
    ctx.set.status = 401
    return { error: 'Unauthorized' }
  }
  return { user: session.user }
})

app.post('/update', async (ctx) => {
  const session = await getSession(ctx.headers.authorization)
  if (!session) {
    ctx.set.status = 401
    return { error: 'Unauthorized' }
  }
  // ...
})

// Good: Auth macro for reusable authentication
const authMacro = new Elysia()
  .macro('auth', {
    async resolve({ headers, set }) {
      const session = await getSession(headers.authorization)
      if (!session) {
        set.status = 401
        return { user: null }
      }
      return { user: session.user }
    }
  })

export default createPlugin()
  .use(authMacro)
  .get('/profile', async ({ user }) => {
    // user is guaranteed to exist here
    return { user }
  }, { auth: true })  // Apply auth macro
  .post('/update', async ({ user, body }) => {
    await updateUser(user.id, body)
    return { success: true }
  }, { auth: true })
```

## Error Handling

Centralize error handling with typed error responses.

```typescript
// src/api/plugins/errors.ts
export const errorPlugin = new Elysia()
  .onError(({ code, error, set }) => {
    switch (code) {
      case 'VALIDATION':
        set.status = 400
        return { 
          error: 'Validation failed',
          details: error.message 
        }
      case 'NOT_FOUND':
        set.status = 404
        return { error: 'Resource not found' }
      case 'INTERNAL_SERVER_ERROR':
        set.status = 500
        return { error: 'Internal server error' }
      default:
        set.status = 500
        return { error: 'Unknown error' }
    }
  })

// Use in all plugins
export default createPlugin()
  .use(errorPlugin)
  .get('/users/:id', async ({ params: { id } }) => {
    const user = await getUser(id)
    if (!user) throw new NotFoundError('User not found')
    return user
  })
```

## API Client Integration

Use Eden Treaty for type-safe API clients in the frontend.

```typescript
// src/lib/api-client.ts
import { treaty } from '@elysiajs/eden'
import type { App } from '../api'  // Import Elysia app type

export const api = treaty<App>('http://localhost:3000')

// Usage in Astro/React components
import { api } from '@/lib/api-client'

// Fully typed API calls
const { data, error } = await api.auth['sign-up'].post({
  email: 'user@example.com',
  password: 'securepassword',
  name: 'John Doe'
})

// data is typed as { user: { id: string, email: string, name: string } }
// error is typed as { error: string } | null
```

## Route Organization

Organize routes by domain with clear file structure.

```
src/
├── api/
│   ├── index.ts           # Main Elysia app
│   ├── plugins/
│   │   ├── common/        # Shared utilities
│   │   │   ├── elysia.ts  # createPlugin factory
│   │   │   ├── request-id.ts
│   │   │   └── logger.ts
│   │   ├── errors.ts
│   │   └── auth.ts        # Auth macro
│   ├── auth/
│   │   ├── index.ts       # Routes index
│   │   ├── sign-up.ts     # Individual routes
│   │   ├── sign-in.ts
│   │   └── sign-out.ts
│   ├── users/
│   │   ├── index.ts
│   │   ├── get-user.ts
│   │   └── update-user.ts
│   └── stores/
│       └── ...
```

## Rules

1. Always use `createPlugin()` factory for consistent middleware setup
2. Define TypeBox schemas for all route inputs (body, params, query) and responses
3. Use macros for cross-cutting concerns like authentication and logging
4. Include `detail` property with summary and tags for OpenAPI documentation
5. Use the Eden Treaty client for type-safe frontend API consumption
6. Implement centralized error handling with `onError` handler
7. Structure routes by domain/feature with index.ts barrel exports
8. Use explicit HTTP status codes with typed response schemas
9. Enable logging with request IDs for request tracing
10. Validate at the edge with TypeBox schemas before business logic
