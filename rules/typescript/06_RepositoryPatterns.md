---
trigger: always_on
---

# Repository Layer Patterns (Backend)

## 1. Interface + Class

```typescript
export interface IUserRepository {
  FindById(id: string): Promise<UserEntity | null>;
  FindByUsername(username: string): Promise<UserEntity | null>;
  FindFiltered(input: FilterRequestInput): Promise<{ data: UserEntity[]; total: number }>;
  Create(data: Partial<UserEntity>): Promise<UserEntity>;
  Update(id: string, data: Partial<UserEntity>, version: number): Promise<UserEntity | null>;
  SoftDelete(id: string, version: number): Promise<boolean>;
}

export class UserRepository implements IUserRepository {
  constructor(private db: MySql2Database) {}
  // ...
}
```

## 2. Query Patterns

**SELECT single:**
```typescript
async FindById(id: string): Promise<UserEntity | null> {
  const [row] = await this.db
    .select()
    .from(users)
    .where(and(eq(users.id, id), eq(users.deletedAt, null)))
    .limit(1);
  return row || null;
}
```

**INSERT:**
```typescript
async Create(data: Partial<UserEntity>): Promise<UserEntity> {
  const [row] = await this.db.insert(users).values(data).$returningId();
  return row;
}
```

**UPDATE with optimistic locking:**
```typescript
async Update(id: string, data: Partial<UserEntity>, version: number): Promise<UserEntity | null> {
  const [row] = await this.db
    .update(users)
    .set({ ...data, version: version + 1 })
    .where(and(eq(users.id, id), eq(users.version, version)))
    .$returningId();
  return row || null; // null = version mismatch → conflict
}
```

**SOFT DELETE:**
```typescript
async SoftDelete(id: string, version: number): Promise<boolean> {
  const result = await this.db
    .update(users)
    .set({ deletedAt: new Date(), version: version + 1 })
    .where(and(eq(users.id, id), eq(users.version, version)));
  return result.affectedRows > 0;
}
```

**Filter with pagination:**
```typescript
async FindFiltered(input: FilterRequestInput): Promise<{ data: UserEntity[]; total: number }> {
  let conditions = [eq(users.deletedAt, null)];

  if (input.search) {
    conditions.push(sql`CONCAT(first_name, ' ', last_name) LIKE ${`%${input.search}%`}`);
  }
  if (input.role) {
    conditions.push(eq(users.role, input.role));
  }

  const data = await this.db.select().from(users)
    .where(and(...conditions))
    .limit(input.pageSize)
    .offset((input.page - 1) * input.pageSize);

  const [{ count }] = await this.db.select({ count: count() })
    .from(users).where(and(...conditions));

  return { data, total: count };
}
```

**Transaction with FOR UPDATE:**
```typescript
await this.db.transaction(async (tx) => {
  const [product] = await tx.select().from(products)
    .where(eq(products.id, id))
    .for('update');
  // ... mutations
});
```

## 3. MongoDB Repository

```typescript
export class AuditLogRepository {
  constructor(private mongo: MongoClient) {}

  async Insert(doc: AuditLogDoc): Promise<void> {
    await this.mongo.db().collection('audit_logs').insertOne(doc);
  }

  async FindByRecordId(tableName: string, recordId: string): Promise<AuditLogDoc[]> {
    return this.mongo.db().collection('audit_logs')
      .find({ tableName, recordId })
      .sort({ createdAt: -1 })
      .toArray();
  }
}
```

## 4. Rules

- Interface prefix `I`: `I{Name}Repository`
- Return `null` on not found (NOT throw)
- Return `null` on version mismatch (service layer throws ConflictError)
- Soft delete: always filter `deletedAt = null`
- Constructor injection: `constructor(private db: MySql2Database)`
- No global `db` import
