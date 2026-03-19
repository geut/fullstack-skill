---
title: Error Handling Patterns
impact: MEDIUM
tags: [typescript, errors, error-handling]
---

# Error Handling Patterns

Use structured error handling with custom error hierarchies, explicit error types, and the NonRetriableError pattern for operations that should not be retried. This ensures predictable error handling and better debugging.

## Why

- **Type Safety**: Custom error classes enable instanceof checks and typed catch blocks
- **Clarity**: Specific error types communicate intent and expected handling
- **Debugging**: Proper error chains with causes enable better stack traces
- **Control**: NonRetriableError prevents wasted retries on permanent failures
- **Domain Boundaries**: Error domains organize related errors by feature area

## Custom Error Hierarchy

Create a base error class with proper stack trace capture.

```typescript
// Bad: Using generic Error with string messages
throw new Error('Something went wrong')

// Good: Custom error hierarchy with proper stack traces
export abstract class AppError extends Error {
  public override readonly cause?: unknown

  constructor(message: string, options?: { cause?: unknown }) {
    super(message)
    this.cause = options?.cause
    this.name = this.constructor.name
    if (Error.captureStackTrace) {
      Error.captureStackTrace(this, this.constructor)
    }
  }
}

// Specific error types
export class ValidationError extends AppError {}
export class NotFoundError extends AppError {}
export class UnauthorizedError extends AppError {}
export class NonRetriableError extends AppError {}
```

## NonRetriableError Pattern

Use NonRetriableError for permanent failures that should not trigger retries.

```typescript
// Bad: Retrying an operation that will always fail
async function sendEmail(email: string, content: string) {
  return retry(async () => {
    if (!isValidEmail(email)) {
      throw new Error('Invalid email')  // Wastes retries
    }
    return await emailService.send(email, content)
  }, { retries: 3 })
}

// Good: Immediately fail non-retriable errors
import { NonRetriableError } from '@/lib/errors'

async function sendEmail(email: string, content: string) {
  return retry(async () => {
    if (!isValidEmail(email)) {
      throw new NonRetriableError('Invalid email format', {
        cause: { email, reason: 'missing @ symbol' }
      })
    }
    return await emailService.send(email, content)
  }, { retries: 3, onFailedAttempt: (error) => {
    if (error instanceof NonRetriableError) {
      throw error  // Don't retry
    }
  }})
}
```

## Error Domains

Organize errors by domain/feature for better maintainability.

```typescript
// Bad: Scattered error definitions
// auth.ts
export class AuthError extends Error {}

// store.ts
export class StoreError extends Error {}

// Good: Centralized error domains
// lib/errors.ts
export const AuthErrors = {
  INVALID_CREDENTIALS: (data?: unknown) => 
    new AppError('Invalid credentials', { cause: data }),
  SESSION_EXPIRED: () => 
    new AppError('Session has expired'),
  EMAIL_ALREADY_EXISTS: (email: string) => 
    new NonRetriableError('Email already registered', { 
      cause: { email } 
    }),
} as const

export const StoreErrors = {
  STORE_NOT_FOUND: (slug: string) => 
    new NotFoundError(`Store '${slug}' not found`),
  STORE_DISABLED: () => 
    new AppError('Store is currently disabled'),
} as const

// Usage
import { AuthErrors } from '@/lib/errors'

if (!validPassword) {
  throw AuthErrors.INVALID_CREDENTIALS({ attempted: email })
}
```

## Error Handling in Async Operations

Handle errors explicitly in async flows with proper type guards.

```typescript
// Bad: Generic error handling
async function loadUserData(userId: string) {
  try {
    const user = await fetchUser(userId)
    const orders = await fetchOrders(user.id)
    return { user, orders }
  } catch (error) {
    console.error('Failed to load data')
    return null
  }
}

// Good: Specific error handling with type guards
async function loadUserData(userId: string): Promise<UserData | null> {
  try {
    const user = await fetchUser(userId)
    const orders = await fetchOrders(user.id)
    return { user, orders }
  } catch (error) {
    if (error instanceof NotFoundError) {
      logger.warn({ userId }, 'User not found')
      return null
    }
    if (error instanceof NonRetriableError) {
      logger.error({ error }, 'Permanent failure, not retrying')
      throw error
    }
    // Re-throw retryable errors
    throw error
  }
}
```

## Error Context

Always include relevant context in error causes for debugging.

```typescript
// Bad: Minimal error information
throw new Error('Database connection failed')

// Good: Rich error context
throw new AppError('Database connection failed', {
  cause: {
    host: dbConfig.host,
    port: dbConfig.port,
    database: dbConfig.database,
    attempt: retryCount,
    timeout: connectionTimeout,
  }
})
```

## Rules

1. Create a base AppError class with proper stack trace capture
2. Extend AppError for specific error types (ValidationError, NotFoundError, etc.)
3. Use NonRetriableError for permanent failures that should not be retried
4. Organize errors into domain-specific objects (AuthErrors, StoreErrors, etc.)
5. Always include relevant context in error `cause` property
6. Use `instanceof` checks for typed error handling in catch blocks
7. Re-throw errors that should propagate up the call stack
8. Log errors with their full context and stack traces
9. Use error codes or constants for error identification
10. Prefer custom error classes over generic Error with string messages
