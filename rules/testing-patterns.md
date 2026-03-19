---
title: Bun Testing Patterns
impact: MEDIUM
tags: [testing, bun, typescript]
---

# Bun Testing Patterns

Use Bun's built-in test runner for fast, type-safe testing with factory patterns for multi-adapter testing. This ensures consistent test behavior and optimal performance.

## Why

- **Performance**: Bun test runner is significantly faster than Jest or Vitest
- **Native Integration**: Built into Bun runtime, no additional dependencies
- **Type Safety**: Full TypeScript support without transpilation
- **Async Support**: Excellent async/await handling with proper timeout controls
- **Factory Pattern**: Reusable test factories for testing multiple implementations

## Bun Test Configuration

Configure tests to use Bun's native test runner with proper setup.

```typescript
// Bad: Using Vitest with complex config
// vitest.config.ts
import { defineConfig } from 'vitest/config'
export default defineConfig({
  test: {
    environment: 'node',
    globals: true,
  }
})

// Good: Use bun:test directly
// package.json
{
  "scripts": {
    "test": "bun test --concurrent",
    "test:watch": "bun test --watch"
  }
}
```

## Basic Test Structure

Use bun:test imports with descriptive test organization.

```typescript
// Bad: Mixed test utilities
import { describe, it, expect } from 'vitest'

// Good: Explicit bun:test imports
import { afterEach, beforeEach, describe, expect, it } from 'bun:test'

describe('User Service', () => {
  describe('createUser', () => {
    it('should create a user with valid data', async () => {
      const user = await createUser({
        email: 'test@example.com',
        name: 'Test User'
      })
      
      expect(user.email).toBe('test@example.com')
      expect(user.name).toBe('Test User')
      expect(user.id).toBeDefined()
    })
    
    it('should throw error for invalid email', () => {
      expect(async () => {
        await createUser({ email: 'invalid', name: 'Test' })
      }).toThrow(ValidationError)
    })
  })
})
```

## Factory Pattern for Multi-Adapter Testing

Use factory functions to test multiple implementations with the same test suite.

```typescript
// Bad: Duplicated tests for each adapter
// postgres.test.ts
describe('Postgres Adapter', () => {
  it('should create job', async () => {
    const adapter = new PostgresAdapter()
    const id = await adapter.createJob({ name: 'test' })
    expect(id).toBeDefined()
  })
})

// pglite.test.ts
describe('PGlite Adapter', () => {
  it('should create job', async () => {
    const adapter = new PGliteAdapter()
    const id = await adapter.createJob({ name: 'test' })
    expect(id).toBeDefined()
  })
})

// Good: Factory pattern for reusable tests
type AdapterFactory = {
  name: string
  create: () => Promise<Adapter>
  cleanup: (adapter: Adapter) => Promise<void>
}

function runAdapterTests(factory: AdapterFactory) {
  describe(`${factory.name} Adapter`, () => {
    let adapter: Adapter
    
    beforeEach(async () => {
      adapter = await factory.create()
      await adapter.start()
    }, { timeout: 60_000 })
    
    afterEach(async () => {
      await adapter.stop()
      await factory.cleanup(adapter)
    })
    
    describe('createJob', () => {
      it('should create job with valid data', async () => {
        const id = await adapter.createJob({
          name: 'test-job',
          payload: { foo: 'bar' }
        })
        
        expect(id).toBeString()
        expect(id.length).toBeGreaterThan(0)
      })
      
      it('should store job data correctly', async () => {
        const payload = { key: 'value' }
        const id = await adapter.createJob({
          name: 'test-job',
          payload
        })
        
        const job = await adapter.getJob(id)
        expect(job.payload).toEqual(payload)
      })
    })
    
    describe('getJob', () => {
      it('should return null for non-existent job', async () => {
        const job = await adapter.getJob('non-existent-id')
        expect(job).toBeNull()
      })
    })
  })
}

// Run tests with multiple factories
runAdapterTests({
  name: 'Postgres',
  create: async () => new PostgresAdapter({ connectionString: process.env.TEST_DATABASE_URL }),
  cleanup: async (adapter) => { /* cleanup */ }
})

runAdapterTests({
  name: 'PGlite',
  create: async () => new PGliteAdapter({ dataDir: ':memory:' }),
  cleanup: async () => { /* no cleanup needed for in-memory */ }
})
```

## Setup Files

Use setup files for test environment initialization.

```typescript
// test/setup.ts
import { createContainer, waitForContainer } from './docker'

console.log('🔄 Creating postgres container...')
const container = await createContainer({
  image: 'postgres:16',
  env: {
    POSTGRES_USER: 'test',
    POSTGRES_PASSWORD: 'test',
    POSTGRES_DB: 'test'
  },
  ports: { '5432': '5433' }
})

await waitForContainer(container, { timeout: 30_000 })
console.log('✅ Postgres container ready')

// Global teardown
globalThis.afterAll?.(async () => {
  console.log('🧹 Cleaning up containers...')
  await container.stop()
})
```

## Async Testing

Handle async operations with proper timeouts and error handling.

```typescript
// Bad: Tests without timeout control
describe('Async Operations', () => {
  it('should complete', async () => {
    const result = await slowOperation()
    expect(result).toBeDefined()
  })  // May hang indefinitely
})

// Good: Explicit timeouts and concurrent execution
describe('Async Operations', () => {
  it('should complete within timeout', async () => {
    const result = await slowOperation()
    expect(result).toBeDefined()
  }, { timeout: 5_000 })  // 5 second timeout
  
  it('should handle multiple concurrent operations', async () => {
    const promises = [
      operation1(),
      operation2(),
      operation3()
    ]
    
    const results = await Promise.all(promises)
    expect(results).toHaveLength(3)
  }, { timeout: 10_000 })
})
```

## Mocking with Bun

Use Bun's mock capabilities for dependency injection.

```typescript
import { mock, spyOn } from 'bun:test'

describe('Email Service', () => {
  it('should send email via service', async () => {
    const sendMock = mock(() => Promise.resolve({ success: true }))
    
    const service = new EmailService({
      provider: { send: sendMock }
    })
    
    await service.send({ to: 'user@example.com', subject: 'Test' })
    
    expect(sendMock).toHaveBeenCalledTimes(1)
    expect(sendMock).toHaveBeenCalledWith({
      to: 'user@example.com',
      subject: 'Test'
    })
  })
})
```

## Rules

1. Always import from `bun:test` instead of external test libraries
2. Use factory functions for testing multiple implementations with the same suite
3. Set explicit timeouts using `{ timeout: ms }` option on tests
4. Use `beforeEach`/`afterEach` for test isolation with proper cleanup
5. Create setup files for environment initialization (databases, containers)
6. Run tests with `--concurrent` flag for faster execution
7. Use bun's built-in mocking instead of external mock libraries
8. Test both success and error paths with specific error assertions
9. Use descriptive test names that explain behavior, not implementation
10. Group related tests in nested `describe` blocks for better organization
