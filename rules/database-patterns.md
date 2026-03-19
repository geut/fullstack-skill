---
title: Database Patterns with Drizzle ORM
impact: MEDIUM
tags: [database, drizzle, postgres, fullstack]
---

# Database Patterns with Drizzle ORM

Use Drizzle ORM with PostgreSQL, soft delete architecture, and model factories for type-safe database operations. These patterns ensure data integrity, maintainability, and consistent querying across your application.

## Why

- **Type Safety**: Full TypeScript inference from schema to queries
- **SQL-Like API**: Write queries that look like SQL with TypeScript types
- **Soft Delete**: Built-in data preservation with automatic filtering
- **Reusable Patterns**: Model factories reduce boilerplate CRUD code
- **Performance**: Efficient query building with proper indexing

## Soft Delete Architecture

Implement soft delete by default with automatic filtering in models.

```typescript
// Bad: Hard delete with data loss
export const users = pgTable('users', {
  id: uuid().primaryKey().defaultRandom(),
  email: varchar({ length: 256 }).notNull(),
  name: varchar({ length: 100 }).notNull(),
})

// Deleting actually removes data forever
await db.delete(users).where(eq(users.id, userId))

// Good: Soft delete with deletedAt timestamp
import { timestampColumn } from './common/columns'

export const users = pgTable('users', {
  id: uuid().primaryKey().defaultRandom(),
  email: varchar({ length: 256 }).notNull(),
  name: varchar({ length: 100 }).notNull(),
  deletedAt: timestampColumn(),  // Soft delete column
})

// Model automatically excludes deleted records
const user = await userModel.getById(userId)  // Returns null if deleted
const allUsers = await userModel.getAll()  // Excludes deleted

// Hard delete available when needed
await userModel.deleteBy({ id: userId }, { hard: true })
```

## Reusable Column Patterns

Define common column patterns for consistency across tables.

```typescript
// src/db/common/columns.ts
import { pgTable, timestamp, uuid, varchar } from 'drizzle-orm/pg-core'

// Primary key with auto-generated UUID
export const primaryKeyColumn = uuid().primaryKey().defaultRandom()

// Email column with validation
export const emailColumn = varchar({ length: 256 }).notNull()

// Timestamp columns factory
export const timestampColumn = () => 
  timestamp({ mode: 'date', withTimezone: true })

// Standard timestamp set (createdAt, updatedAt, deletedAt)
export const timestampsColumns = {
  createdAt: timestampColumn().notNull().defaultNow(),
  updatedAt: timestampColumn().notNull().defaultNow().$onUpdate(() => sql`now()`),
  deletedAt: timestampColumn(),
}

// Usage in schema definition
export const users = pgTable('users', {
  id: primaryKeyColumn,
  email: emailColumn,
  name: varchar({ length: 100 }).notNull(),
  ...timestampsColumns,
})
```

## Model Factory Pattern

Create reusable CRUD models with automatic soft delete filtering.

```typescript
// src/db/common/models.ts
import { eq, isNull, and, sql } from 'drizzle-orm'

export function createModel<T extends PgTable>(table: T, db: PostgresJsDatabase) {
  type InsertData = typeof table.$inferInsert
  type SelectData = typeof table.$inferSelect
  type IdColumn = Extract<keyof SelectData, string>
  
  return {
    // Get all non-deleted records
    async getAll(
      options: { 
        limit?: number
        offset?: number
        orderBy?: any
      } = {}
    ) {
      const { limit = 100, offset = 0, orderBy } = options
      
      return db
        .select()
        .from(table)
        .where(isNull(table.deletedAt))
        .limit(limit)
        .offset(offset)
        .orderBy(orderBy || sql`${table.createdAt} desc`)
    },
    
    // Get by ID (excluding deleted)
    async getById(id: string) {
      const [result] = await db
        .select()
        .from(table)
        .where(and(
          eq(table.id, id),
          isNull(table.deletedAt)
        ))
        .limit(1)
      
      return result || null
    },
    
    // Get by any column (excluding deleted)
    async getBy<K extends keyof SelectData>(
      column: K, 
      value: SelectData[K]
    ) {
      const [result] = await db
        .select()
        .from(table)
        .where(and(
          eq(table[column], value),
          isNull(table.deletedAt)
        ))
        .limit(1)
      
      return result || null
    },
    
    // Create new record
    async create(data: InsertData, tx?: typeof db) {
      const dbInstance = tx || db
      const [result] = await dbInstance
        .insert(table)
        .values(data)
        .returning()
      return result
    },
    
    // Update record (excluding deleted)
    async update(
      id: string, 
      data: Partial<InsertData>,
      tx?: typeof db
    ) {
      const dbInstance = tx || db
      const [result] = await dbInstance
        .update(table)
        .set(data)
        .where(and(
          eq(table.id, id),
          isNull(table.deletedAt)
        ))
        .returning()
      return result || null
    },
    
    // Soft delete by default
    async delete(
      id: string, 
      options: { hard?: boolean } = {},
      tx?: typeof db
    ) {
      const dbInstance = tx || db
      
      if (options.hard) {
        // Permanent deletion
        await dbInstance
          .delete(table)
          .where(eq(table.id, id))
        return true
      }
      
      // Soft delete
      const [result] = await dbInstance
        .update(table)
        .set({ deletedAt: new Date() })
        .where(and(
          eq(table.id, id),
          isNull(table.deletedAt)
        ))
        .returning()
      
      return !!result
    },
    
    // Delete by column (soft delete by default)
    async deleteBy(
      where: Partial<SelectData>,
      options: { hard?: boolean } = {},
      tx?: typeof db
    ) {
      const dbInstance = tx || db
      
      if (options.hard) {
        await dbInstance
          .delete(table)
          .where(and(
            ...Object.entries(where).map(([key, value]) => 
              eq(table[key], value)
            )
          ))
        return true
      }
      
      await dbInstance
        .update(table)
        .set({ deletedAt: new Date() })
        .where(and(
          isNull(table.deletedAt),
          ...Object.entries(where).map(([key, value]) => 
            eq(table[key], value)
          )
        ))
      
      return true
    },
    
    // Include deleted records (admin use)
    async getAllWithDeleted(options = {}) {
      const { limit = 100, offset = 0 } = options
      return db.select().from(table).limit(limit).offset(offset)
    },
    
    // Restore soft-deleted record
    async restore(id: string, tx?: typeof db) {
      const dbInstance = tx || db
      const [result] = await dbInstance
        .update(table)
        .set({ deletedAt: null })
        .where(eq(table.id, id))
        .returning()
      return result || null
    }
  }
}
```

## Schema Relations

Define relations for complex queries with automatic joins.

```typescript
// src/db/schemas/users.ts
import { relations } from 'drizzle-orm'
import { pgTable, uuid, varchar } from 'drizzle-orm/pg-core'
import { timestampsColumns } from '../common/columns'

export const users = pgTable('users', {
  id: uuid().primaryKey().defaultRandom(),
  email: varchar({ length: 256 }).notNull(),
  name: varchar({ length: 100 }).notNull(),
  ...timestampsColumns,
})

export const usersRelations = relations(users, ({ many }) => ({
  orders: many(orders),
  sessions: many(sessions),
}))

// src/db/schemas/orders.ts
export const orders = pgTable('orders', {
  id: uuid().primaryKey().defaultRandom(),
  userId: uuid().notNull().references(() => users.id),
  total: decimal({ precision: 10, scale: 2 }).notNull(),
  status: orderStatusEnum().notNull().default('pending'),
  ...timestampsColumns,
})

export const ordersRelations = relations(orders, ({ one }) => ({
  user: one(users, {
    fields: [orders.userId],
    references: [users.id],
  }),
}))

// Usage - automatic join via relations
const userWithOrders = await db.query.users.findFirst({
  where: eq(users.id, userId),
  with: {
    orders: {
      where: eq(orders.status, 'completed'),
      orderBy: desc(orders.createdAt),
      limit: 10
    }
  }
})
```

## Transaction Support

Support transactions across multiple model operations.

```typescript
// Usage with transactions
async function transferBalance(fromId: string, toId: string, amount: number) {
  return await db.transaction(async (tx) => {
    // All operations use the same transaction
    const fromUser = await userModel.update(
      fromId, 
      { balance: sql`${users.balance} - ${amount}` },
      tx
    )
    
    const toUser = await userModel.update(
      toId,
      { balance: sql`${users.balance} + ${amount}` },
      tx
    )
    
    await transactionModel.create({
      fromUserId: fromId,
      toUserId: toId,
      amount,
    }, tx)
    
    return { fromUser, toUser }
  })
}
```

## Rules

1. Always implement soft delete with `deletedAt` timestamp column
2. Use model factory functions to create reusable CRUD operations
3. Automatically filter deleted records in all standard queries
4. Define reusable column patterns (timestamps, primary keys, common fields)
5. Use snake_case for database column names with explicit mapping
6. Define Drizzle relations for automatic joins and nested queries
7. Support transactions by accepting optional `tx` parameter in model methods
8. Provide `{ hard: true }` option for actual deletion when needed
9. Include `createdAt`, `updatedAt`, and `deletedAt` on all tables
10. Use `defaultRandom()` for UUID primary keys
11. Create indexes on frequently queried columns and foreign keys
12. Use type inference from schema definitions for insert/select types
