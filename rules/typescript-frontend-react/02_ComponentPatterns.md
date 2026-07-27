---
trigger: always_on
---

# Component Patterns (Frontend)

## 1. View Component (Props Only)

```typescript
// modules/customer/view.tsx
interface CustomerListViewProps {
  customers: Customer[];
  loading: boolean;
  error: string | null;
  pagination: Pagination | null;
  onSearch: (term: string) => void;
  onPageChange: (page: number) => void;
}

export function CustomerListView({
  customers,
  loading,
  error,
  pagination,
  onSearch,
  onPageChange,
}: CustomerListViewProps) {
  if (loading) return <LoadingSkeleton />;
  if (error) return <ErrorMessage message={error} />;
  if (customers.length === 0) return <EmptyState message="No customers found" />;

  return (
    <Box>
      <SearchInput onSearch={onSearch} />
      <DataGrid rows={customers} columns={columns} />
      {pagination && <Pagination {...pagination} onChange={onPageChange} />}
    </Box>
  );
}
```

## 2. Controller Hook

```typescript
// modules/customer/controller.ts
export function useCustomerList() {
  const [customers, setCustomers] = useState<Customer[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [pagination, setPagination] = useState<Pagination | null>(null);
  const [search, setSearch] = useState('');
  const [page, setPage] = useState(1);

  const fetchCustomers = useCallback(async () => {
    setLoading(true);
    setError(null);
    try {
      const result = await customerApi.FindFiltered({ search, page, pageSize: 20 });
      setCustomers(result.data);
      setPagination(result.pagination);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Failed to load');
    } finally {
      setLoading(false);
    }
  }, [search, page]);

  useEffect(() => { fetchCustomers(); }, [fetchCustomers]);

  return { customers, loading, error, pagination, setSearch, setPage, refetch: fetchCustomers };
}
```

## 3. Route Composer

```typescript
// router.tsx — compose route with view + controller
export function CustomerListRoute() {
  const { customers, loading, error, pagination, setSearch, setPage } = useCustomerList();

  return (
    <CustomerListView
      customers={customers}
      loading={loading}
      error={error}
      pagination={pagination}
      onSearch={setSearch}
      onPageChange={setPage}
    />
  );
}
```

## 4. Rules

- **View**: Never call APIs directly — data comes via props only
- **Controller**: Never render JSX — returns state + actions
- **Model**: Never import React — pure TypeScript types + API functions
- Use named exports only (no `export default` except `App`)
- Components use PascalCase filenames
