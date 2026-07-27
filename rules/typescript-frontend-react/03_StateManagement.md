---
trigger: always_on
---

# State Management & API Patterns (Frontend)

## 1. API Client (Axios)

```typescript
// config/api.ts
import axios from 'axios';

export const api = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL || '/api',
  timeout: 30000,
  headers: { 'Content-Type': 'application/json' },
});

// Auth interceptor
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// 401 interceptor → redirect to login
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  },
);
```

## 2. Model (API Functions)

```typescript
// modules/customer/model.ts
import { z } from 'zod';
import { api } from '@/config/api';

export const customerSchema = z.object({
  id: z.string(),
  firstName: z.string(),
  lastName: z.string(),
  email: z.string().email(),
  phone: z.string().nullable(),
});

export type Customer = z.infer<typeof customerSchema>;

export const filterSchema = z.object({
  search: z.string().optional(),
  page: z.number().min(1).default(1),
  pageSize: z.number().min(1).max(100).default(20),
});

export type FilterInput = z.infer<typeof filterSchema>;

export const customerApi = {
  FindFiltered: async (input: FilterInput) => {
    const { data } = await api.post('/customers/filter', input);
    return data;
  },
  FindById: async (id: string) => {
    const { data } = await api.get(`/customers/${id}`);
    return data;
  },
};
```

## 3. Auth Store (Zustand)

```typescript
// stores/authStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface AuthState {
  token: string | null;
  user: { id: string; role: string; displayName: string } | null;
  login: (token: string, user: AuthState['user']) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      token: null,
      user: null,
      login: (token, user) => set({ token, user }),
      logout: () => {
        set({ token: null, user: null });
        localStorage.removeItem('token');
        window.location.href = '/login';
      },
    }),
    { name: 'auth' },
  ),
);
```

## 4. Rules

- Zustand for auth state only — other state uses React hooks
- API functions live in `model.ts` — never in components
- Axios interceptors handle auth headers and 401 redirects
- Zod schemas for API response validation
- `localStorage` for token storage (SPA pattern)
