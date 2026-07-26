---
trigger: always_on
---

# Service Layer Patterns (Backend)

## 1. Interface + Class

```typescript
export interface IUserService {
  Login(input: LoginInput): Promise<{ token: string; user: UserResponse }>;
  FindFiltered(input: FilterRequestInput, adminId: string, meta?: AuditMeta): Promise<FilteredResult<UserResponse>>;
  Update(id: string, input: UpdateInput, adminId: string, meta?: AuditMeta): Promise<UserResponse>;
}

export class UserService implements IUserService {
  constructor(
    private repo: IUserRepository,
    private redis: Redis,
    private auditService: IAuditLogService,
  ) {}
}
```

## 2. CRUD Pattern

```typescript
async FindFiltered(
  input: FilterRequestInput,
  adminId: string,
  meta?: AuditMeta,
): Promise<FilteredResult<UserResponse>> {
  // Guard: access check
  // ...

  const { data, total } = await this.repo.FindFiltered(input);
  return {
    data: data.map((e) => this.toResponse(e)),
    pagination: calculatePagination(input.page, input.pageSize, total),
  };
}

async Update(
  id: string,
  input: UpdateInput,
  adminId: string,
  meta?: AuditMeta,
): Promise<UserResponse> {
  // Guard: existence check
  const existing = await this.repo.FindById(id);
  if (!existing) throw new NotFoundError('User not found');

  // Update with optimistic locking (repo returns null on conflict)
  const updated = await this.repo.Update(id, input, input.version);
  if (!updated) throw new ConflictError('Version mismatch');

  // Audit log
  this.auditService.Insert('UPDATE', 'users', id, adminId, existing, updated, meta);

  return this.toResponse(updated);
}
```

## 3. Transaction Pattern

```typescript
async CreateInvoice(input: CreateInvoiceInput, userId: string): Promise<InvoiceResponse> {
  return await this.db.transaction(async (tx) => {
    // SELECT FOR UPDATE
    const [product] = await tx.select().from(products)
      .where(eq(products.id, input.productId))
      .for('update');

    if (!product || product.stock < input.quantity) {
      throw new BadRequestError('Insufficient stock');
    }

    // INSERT invoice + items
    const [invoice] = await tx.insert(invoices).values({ ... }).$returningId();

    // UPDATE stock
    await tx.update(products).set({ stock: product.stock - input.quantity })
      .where(eq(products.id, input.productId));

    return this.toResponse(invoice);
  });
}
```

## 4. Rules

- Interface prefix `I`: `I{Name}Service`
- Guard clauses first: existence → auth → mutate
- Audit after every mutation (fire-and-forget via `setImmediate` or worker)
- Service never imports DB directly — uses repo interface
- Public methods: PascalCase, max 50 lines
- Private helpers: camelCase
