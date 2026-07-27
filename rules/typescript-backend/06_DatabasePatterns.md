---
trigger: always_on
---

# Database Patterns (Backend)

## 1. ORM: Drizzle ORM (MySQL)

```typescript
// backend/src/config/schema.ts — ALL tables in one file
import { mysqlTable, varchar, int, decimal, timestamp, mysqlEnum, boolean, index } from 'drizzle-orm/mysql-core';

export const users = mysqlTable('users', {
  id: varchar('id', { length: 36 }).primaryKey(),
  username: varchar('username', { length: 100 }).notNull().unique(),
  passwordHash: varchar('password_hash', { length: 255 }).notNull(),
  displayName: varchar('display_name', { length: 255 }).notNull(),
  role: mysqlEnum('role', ['ADMIN', 'STAFF', 'USER']).notNull().default('USER'),
  isActive: boolean('is_active').notNull().default(true),
  version: int('version').notNull().default(1),
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow().onUpdateNow(),
  deletedAt: timestamp('deleted_at'),
}, (table) => ({
  usernameIdx: index('idx_users_username').on(table.username),
}));
```

## 2. Entity Interface

```typescript
// backend/src/modules/user/entity.ts
export interface UserEntity {
  id: string;
  username: string;
  passwordHash: string;
  displayName: string;
  role: 'ADMIN' | 'STAFF' | 'USER';
  isActive: boolean;
  version: number;
  createdAt: Date;
  updatedAt: Date;
  deletedAt: Date | null;
}
```

## 3. Conventions

| Item | Convention |
|---|---|
| PK | UUID v4 (`varchar(36)`) |
| Timestamps | `Date` type (ISO in JSON) |
| Soft delete | `deletedAt: timestamp` nullable |
| Locking | `version: int` default 1 |
| Money | `decimal(12, 2)` — never float |
| Charset | `utf8mb4` + `unicode_ci` |
| Engine | InnoDB |
| JSON fields | `json()` type (MySQL 8+) |

## 4. Migrations (Drizzle Kit)

```bash
npx drizzle-kit generate    # Generate SQL from schema changes
npx drizzle-kit migrate     # Apply migrations
npx drizzle-kit studio      # Drizzle Studio UI
```

## 5. Multi-DB Strategy

| Database | Use Case |
|---|---|
| **MySQL** | Core business data (ACID, relations) |
| **MongoDB** | Audit logs, chat history, activity logs (high-write, flexible schema) |
| **Redis** | Cache, sessions, rate limiting |

## 6. MongoDB Collections

```typescript
// Native MongoDB driver (no Mongoose)
audit_logs: {
  _id: string,
  action: 'CREATE' | 'UPDATE' | 'DELETE',
  tableName: string,
  recordId: string,
  changeDatas: Array<{ field, old, new }>,
  userId: string,
  userDisplayName: string,   // denormalized
  createdAt: Date,
}
// Indexes: (tableName+recordId+createdAt), (userId+createdAt)
```
