---
name: tanstack-api-integration
description: Rules and patterns for API data-fetching with TanStack React Query (useQuery, useMutation, useInfiniteQuery). Trigger when the user mentions "fetch data", "API hook", "query key", "invalidate cache", "useMutation", "optimistic update", "prefetch", "stale time", "useQuery", "queryClient", "server state", "cache invalidation", "background refetch", "infinite scroll", "pagination", or any server-state management in React. Also trigger for "create a service", "add an API call", "integrate an endpoint", "add a query hook", or "implement data fetching".
---

# TanStack API Integration

This skill teaches you how to implement robust API integrations using TanStack React Query (v5) and Axios. Follow these patterns when building or modifying data-fetching layers in React applications.

> **Scope:** This skill covers TanStack React Query for **server-state** management only.
> For form submissions, see `react-hook-form-with-zod`.
> For client-state, see `zustand-usage`.
> For folder structure conventions, see `react-folder-structure`.
> For general React patterns, see `react-quality-rules`.

## Core Laws

- ALL data fetching uses `useQuery` — never `useEffect` + `useState` for API calls
- ALL mutations use `useMutation` — never raw `axios.post()` in event handlers
- Every query has a unique, deterministic `queryKey` — keys are **arrays**, not strings
- Query keys follow hierarchy: `["entity", id, "sub-entity"]` → `["users", 5, "posts"]`
- Use a **Query Key Factory** for all entities — never scatter string literals
- Mutations ALWAYS invalidate affected queries in `onSettled` (not `onSuccess` — covers errors too)
- Service files export BOTH raw API functions AND custom hooks — never only one
- **NEVER use `any`** — not in generics, not in function params, not in type assertions. Use proper interfaces, `unknown`, or generic type parameters instead. This is non-negotiable.
- Type every request and response with explicit interfaces — no implicit or default types
- `refetchOnWindowFocus: false` is the project default
- Use `isPending` for initial loading states — never the deprecated `isLoading`
- Hooks handle cache; components handle UI feedback (toasts, redirects)

---

## Architecture

Concerns are separated into five distinct layers:

1. **API Wrapper (`api.ts`)**: Base Axios configuration, interceptors, and global error handling.
2. **Query Key Factory (`queryKeys.ts`)**: Centralized, type-safe key definitions for all entities.
3. **Type Definitions (`/types/**/*`)**: Request and Response interfaces, stored **outside** the `app` folder at the root level.
4. **Service Files (`services/*.ts`)**: Raw API functions + TanStack Query hooks, co-located per domain.
5. **UI Components (`components/*.tsx` & `pages/*.tsx`)**: React components that consume the hooks.

---

## Type Organization

All API-related types must be stored in a centralized `types` directory at the project root (outside of `app/` or `src/app/`). Group types by module and separate Requests from Responses.

**Structure:**
- `types/[ModuleName]API/api-request.type.ts`
- `types/[ModuleName]API/api-response.type.ts`

**Example (Agency Module):**

```typescript
// types/AgencyModuleAPI/api-request.type.ts
export interface UpdateAgencyRequest {
  company_name: string;
  status: string;
  owner_name: string;
  owner_email: string;
  owner_phone: string;
}

// types/AgencyModuleAPI/api-response.type.ts
export interface Agency {
  company_id: string;
  company_name: string;
  owner_id: number;
  owner_name: string;
  owner_email: string;
  owner_phone: string | null;
  status: "active" | "inactive";
  subscription: "paid" | "unpaid" | "expired";
}
```

---

## Step 0: QueryClient Setup (Prerequisite)

Before anything works, configure `QueryClient` with sensible defaults and wrap your app in `QueryClientProvider`.

```tsx
// providers/QueryProvider.tsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";
import { ReactNode } from "react";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 2,
      staleTime: 0,
      gcTime: 1000 * 60 * 5, // 5 minutes (formerly cacheTime)
      refetchOnWindowFocus: false,
      throwOnError: false,
    },
    mutations: {
      retry: 0, // mutations should NOT auto-retry
    },
  },
});

export function QueryProvider({ children }: { children: ReactNode }) {
  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

```tsx
// app layout or _app.tsx
import { QueryProvider } from "@/providers/QueryProvider";

export default function App({ children }) {
  return <QueryProvider>{children}</QueryProvider>;
}
```

---

## Step 1: API Wrapper (`api.ts`)

This file configures Axios and handles global concerns like cookies, interceptors, and error toasts.

```typescript
import axios, { AxiosError, AxiosRequestConfig, AxiosResponse, InternalAxiosRequestConfig } from "axios";
import { toast } from "react-toastify";

const API_BASE_URL = process.env.NEXT_PUBLIC_API_URL;

export const axiosInstance = axios.create({
  baseURL: API_BASE_URL,
  timeout: 180000,
  withCredentials: true,
  headers: { "Content-Type": "application/json" },
});

// Request Interceptor
axiosInstance.interceptors.request.use(
  (config: InternalAxiosRequestConfig) => {
    // Let browser set boundary for FormData automatically
    if (config.data instanceof FormData && config.headers) {
      delete config.headers["Content-Type"];
    }
    return config;
  },
  (error: AxiosError) => Promise.reject(error)
);

// Type for API error responses — NEVER use `any` for error data
interface ApiErrorResponse {
  message?: string;
  error?: string;
  statusCode?: number;
}

// Response Interceptor
axiosInstance.interceptors.response.use(
  (response: AxiosResponse) => response,
  async (error: AxiosError<ApiErrorResponse>) => {
    const status = error.response?.status;
    const data = error.response?.data;

    if (status === 401) {
      window.location.href = "/login";
    } else if (status === 403) {
      toast.error(data?.message || "Forbidden");
    } else if (status === 422 || status === 400) {
      toast.error(data?.message || "Validation Error");
    } else if (status && status >= 500) {
      toast.error("Server Error: Please try again later");
    }

    return Promise.reject(error);
  }
);

// Reusable typed REST methods — generic T is REQUIRED at call site, no defaults
export const get = async <T>(url: string, config?: AxiosRequestConfig): Promise<T> =>
  (await axiosInstance.get<T>(url, config)).data;

export const post = async <T, D = unknown>(url: string, data?: D, config?: AxiosRequestConfig): Promise<T> =>
  (await axiosInstance.post<T>(url, data, config)).data;

export const patch = async <T, D = unknown>(url: string, data?: D, config?: AxiosRequestConfig): Promise<T> =>
  (await axiosInstance.patch<T>(url, data, config)).data;

export const put = async <T, D = unknown>(url: string, data?: D, config?: AxiosRequestConfig): Promise<T> =>
  (await axiosInstance.put<T>(url, data, config)).data;

export const del = async <T>(url: string, config?: AxiosRequestConfig): Promise<T> =>
  (await axiosInstance.delete<T>(url, config)).data;
```

---

## Step 2: Query Key Factory (`queryKeys.ts`)

Centralize all query keys in a factory. This prevents typos, enables granular invalidation, and makes keys discoverable.

```typescript
// api/queryKeys.ts

export const userKeys = {
  all: ["users"] as const,
  lists: () => [...userKeys.all, "list"] as const,
  list: (filters: UserFilters) => [...userKeys.lists(), filters] as const,
  details: () => [...userKeys.all, "detail"] as const,
  detail: (id: string) => [...userKeys.details(), id] as const,
};

export const productKeys = {
  all: ["products"] as const,
  lists: () => [...productKeys.all, "list"] as const,
  list: (filters?: ProductFilters) => [...productKeys.lists(), filters] as const,
  details: () => [...productKeys.all, "detail"] as const,
  detail: (id: string) => [...productKeys.details(), id] as const,
};
```

**Usage in hooks:**
```typescript
useQuery({ queryKey: userKeys.detail(userId), queryFn: ... });
```

**Usage in invalidation — invalidate ALL user queries at once:**
```typescript
queryClient.invalidateQueries({ queryKey: userKeys.all });
```

**Key hierarchy rule:** Keys are matched by prefix. Invalidating `["users"]` also invalidates `["users", "list"]`, `["users", "detail", "5"]`, etc.

---

## Step 3: Raw API Functions (Service Files)

Create domain-specific service files (e.g., `userService.ts`, `productService.ts`) to house both raw functions and hooks. Define Request/Response types explicitly.

```typescript
// api/services/userService.ts
import { get, patch, post, del } from "../api";

// ---- Types ----

export interface UserProfileResponse {
  id: string;
  first_name: string;
  last_name: string;
  email: string;
  is_active: boolean;
}

export interface UpdateProfileRequest {
  first_name: string;
  last_name: string;
}

export interface UserListResponse {
  users: UserProfileResponse[];
  total: number;
  page: number;
  hasNextPage: boolean;
}

export interface UserFilters {
  page?: number;
  search?: string;
  status?: "active" | "inactive";
}

// ---- Raw API Functions ----

/** GET /users/profile */
export const getUserProfile = async (): Promise<UserProfileResponse> => {
  return await get<UserProfileResponse>("users/profile");
};

/** GET /users/:id */
export const getUserById = async (id: string): Promise<UserProfileResponse> => {
  return await get<UserProfileResponse>(`users/${id}`);
};

/** GET /users?page=1&search=... */
export const getUsers = async (filters: UserFilters): Promise<UserListResponse> => {
  return await get<UserListResponse>("users", { params: filters });
};

/** PATCH /users/profile */
export const updateUserProfile = async (payload: UpdateProfileRequest): Promise<UserProfileResponse> => {
  return await patch<UserProfileResponse>("users/profile", payload);
};

/** DELETE /users/:id */
export const deleteUser = async (id: string): Promise<void> => {
  return await del<void>(`users/${id}`);
};
```

---

## Step 4: TanStack Custom Hooks

Wrap raw functions with `useQuery` for fetching and `useMutation` for creating/updating/deleting. Place hooks in the **same service file** as the raw functions.

### Query Hooks

```typescript
import { useQuery, useMutation, useQueryClient, useInfiniteQuery } from "@tanstack/react-query";
import { userKeys } from "../queryKeys";

/**
 * Hook for fetching current user's profile
 */
export const useGetUserProfile = () => {
  return useQuery({
    queryKey: userKeys.detail("me"),
    queryFn: getUserProfile,
  });
};

/**
 * Hook for fetching a user by ID
 * Uses `enabled` to prevent fetching until ID is available
 */
export const useGetUserById = (userId: string | undefined) => {
  return useQuery({
    queryKey: userKeys.detail(userId!),
    queryFn: () => getUserById(userId!),
    enabled: !!userId, // Don't fetch until userId exists
  });
};

/**
 * Hook for fetching filtered user list
 */
export const useGetUsers = (filters: UserFilters) => {
  return useQuery({
    queryKey: userKeys.list(filters),
    queryFn: () => getUsers(filters),
  });
};
```

### Mutation Hooks

```typescript
/**
 * Hook for updating user profile
 * - Invalidation goes in `onSettled` (fires on BOTH success and error)
 * - UI feedback (toasts, redirects) goes at the CALL SITE, not here
 */
export const useUpdateUserProfile = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: updateUserProfile,
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: userKeys.all });
    },
  });
};

/**
 * Hook for deleting a user
 */
export const useDeleteUser = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: deleteUser,
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: userKeys.all });
    },
  });
};
```

**Why `onSettled` instead of `onSuccess`?**
`onSettled` fires on both success and error. This ensures the cache always re-syncs with the server, even when a mutation partially succeeds or the optimistic update needs reverting.

### Dealing with Logical vs HTTP Errors

Sometimes APIs return HTTP 200 but include a logical error in the payload (e.g., `status: 400` inside the JSON). If `api.ts` interceptors don't catch this, throw it in the raw function:

```typescript
export const processPayment = async (payload: PaymentRequest): Promise<PaymentResponse> => {
  const response = await post<PaymentApiResponse>("payments", payload);
  if (response.status !== "success") {
    throw new Error(response.error_message || "Payment failed");
  }
  return response.data;
};
```

---

## Step 5: Component Usage

### Basic Pattern

```tsx
import { useGetUserProfile, useUpdateUserProfile } from "@/api/services/userService";
import { toast } from "react-toastify";

export function UserProfilePage() {
  const { data: user, isPending, error } = useGetUserProfile();
  const updateMutation = useUpdateUserProfile();

  const handleSave = () => {
    updateMutation.mutate(
      { first_name: "John", last_name: "Doe" },
      {
        // UI feedback at the call site — not in the hook
        onSuccess: () => toast.success("Profile saved successfully"),
        onError: (err) => toast.error(err.message || "Failed to save"),
      }
    );
  };

  if (isPending) return <ProfileSkeleton />;
  if (error) return <ErrorDisplay message="Failed to load profile" />;

  return (
    <div>
      <h1>{user?.first_name}</h1>
      <button onClick={handleSave} disabled={updateMutation.isPending}>
        {updateMutation.isPending ? "Saving..." : "Save"}
      </button>
    </div>
  );
}
```

### `mutateAsync` for Promise Composition

Use `mutateAsync` instead of `mutate` when you need to `await` the result or compose multiple async operations sequentially:

```typescript
const handleBulkSave = async () => {
  try {
    const result = await updateMutation.mutateAsync({ first_name: "John", last_name: "Doe" });
    // Use result for subsequent operations
    await logAuditTrail(result.id);
    toast.success("Profile saved and logged");
  } catch (error) {
    toast.error("Save failed");
  }
};
```

**Rule:** Prefer `mutate` for simple fire-and-forget. Use `mutateAsync` only when you need the returned promise.

### Resetting Mutation State

Use `mutation.reset()` to clear error or success state, typically for dismissible error messages:

```tsx
const mutation = useUpdateUserProfile();

return (
  <div>
    {mutation.isError && (
      <div role="alert">
        {mutation.error.message}
        <button onClick={() => mutation.reset()}>Dismiss</button>
      </div>
    )}
    {/* ... */}
  </div>
);
```

### Dependent Queries (Waterfall)

When one query depends on data from another:

```tsx
export function UserPostsPage({ userId }: { userId: string }) {
  const { data: user } = useGetUserById(userId);
  // This hook internally uses `enabled: !!userId` — won't fetch until userId exists
  const { data: posts, isPending } = useGetUserPosts(user?.id);

  if (isPending) return <Skeleton />;
  return <PostList posts={posts} />;
}
```

### Direct Cache Manipulation

Use `useQueryClient()` in components to interact with the cache directly:

1. **Invalidate Queries** — force a background refetch:
   ```typescript
   queryClient.invalidateQueries({ queryKey: userKeys.all });
   ```
2. **Set Query Data** — instantly update cache without waiting for API:
   ```typescript
   queryClient.setQueryData(userKeys.detail("me"), (old: UserProfileResponse | undefined) => {
     if (!old) return old;
     return { ...old, first_name: "Jane" };
   });
   ```

---

## Caching Semantics

Understanding when data is fresh, stale, or garbage-collected is critical.

| Setting | Default | Meaning |
|---|---|---|
| `staleTime` | `0` | How long data is considered "fresh". While fresh, no background refetch on mount/window focus. |
| `gcTime` | `5 min` | How long **inactive** (unmounted) cache entries stay in memory before garbage collection. |

**Rule of thumb:**
- Static reference data (countries list): `staleTime: Infinity`
- Semi-static data (user profile): `staleTime: 1000 * 60 * 5` (5 min)
- Volatile data (feed, dashboard): `staleTime: 0` (default — always refetch in background)

```typescript
// Example: static data that almost never changes
export const useGetCountries = () => {
  return useQuery({
    queryKey: ["countries"],
    queryFn: getCountries,
    staleTime: Infinity, // Never refetch automatically
  });
};
```

### Loading State Guide

| State | When True | Use For |
|---|---|---|
| `isPending` | No cached data AND query is in-flight | Initial loading skeletons |
| `isFetching` | Any background fetch (initial or refetch) | Subtle spinners, progress indicators |
| `isError` | Query encountered an error | Error fallback UI |
| `isSuccess` | Data is available | Rendering content |

**`fetchStatus` axis** (separate from `status`):

| fetchStatus | Meaning |
|---|---|
| `'fetching'` | Query is currently fetching from the network |
| `'paused'` | Query wanted to fetch but is paused (offline) |
| `'idle'` | Query is not doing anything right now |

`status` tells you about the **data** (do we have it?). `fetchStatus` tells you about the **queryFn** (is it running?). A query can be `status: 'success'` and `fetchStatus: 'fetching'` at the same time (background refetch with stale data).

Use `isPending` for full-page loading. Use `isFetching` for background refresh indicators (e.g., a small spinner in the header).

```tsx
const { data, isPending, isFetching } = useGetUsers(filters);

// Full skeleton on first load
if (isPending) return <TableSkeleton />;

return (
  <div>
    {/* Subtle indicator for background refetches */}
    {isFetching && <RefreshIndicator />}
    <UserTable data={data} />
  </div>
);
```

---

## Data Transformation with `select`

Use the `select` option to transform or filter data from the cache. The transform runs on the cached data — not on every render.

```typescript
// Return only active users — component never sees inactive ones
export const useGetActiveUsers = (filters: UserFilters) => {
  return useQuery({
    queryKey: userKeys.list(filters),
    queryFn: () => getUsers(filters),
    select: (data) => ({
      ...data,
      users: data.users.filter((u) => u.is_active),
    }),
  });
};
```

---

## Type-Safe Query Options with `queryOptions()`

The `queryOptions()` helper provides type-safe, reusable query configurations. Use it to share query options between `useQuery`, `prefetchQuery`, and `invalidateQueries` without duplicating types:

```typescript
import { queryOptions } from "@tanstack/react-query";

// Define once — reuse everywhere
export const userDetailOptions = (userId: string) =>
  queryOptions({
    queryKey: userKeys.detail(userId),
    queryFn: () => getUserById(userId),
    staleTime: 1000 * 60 * 5,
  });

// In hooks
export const useGetUserById = (userId: string) => {
  return useQuery(userDetailOptions(userId));
};

// In prefetching
await queryClient.prefetchQuery(userDetailOptions(userId));

// In invalidation — type-safe key extraction
queryClient.invalidateQueries(userDetailOptions(userId));
```

**Why:** Without `queryOptions()`, you must manually keep `queryKey`, `queryFn`, and types in sync across multiple call sites. `queryOptions()` guarantees type consistency.

---

## Advanced Features

### 1. Optimistic Updates

For instant UI feedback, use `onMutate` to update the cache *before* the mutation finishes, and `onError` to roll back if it fails.

```typescript
export const useToggleUserStatus = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: toggleUserStatus,

    // 1. Cancel outgoing queries & snapshot old data
    onMutate: async (userId: string) => {
      await queryClient.cancelQueries({ queryKey: userKeys.detail(userId) });
      const previousUser = queryClient.getQueryData<UserProfileResponse>(userKeys.detail(userId));

      // 2. Optimistically update the cache
      queryClient.setQueryData<UserProfileResponse>(userKeys.detail(userId), (old) => {
        if (!old) return old;
        return { ...old, is_active: !old.is_active };
      });

      // Return context for rollback
      return { previousUser };
    },

    // 3. Roll back on error
    onError: (_err, userId, context) => {
      if (context?.previousUser) {
        queryClient.setQueryData(userKeys.detail(userId), context.previousUser);
      }
    },

    // 4. Always re-sync after settling
    onSettled: (_data, _err, userId) => {
      queryClient.invalidateQueries({ queryKey: userKeys.detail(userId) });
    },
  });
};
```

### 2. Infinite Queries

For paginated data (infinite scroll), use `useInfiniteQuery`.

```typescript
export const useInfiniteUsers = (filters: Omit<UserFilters, "page">) => {
  return useInfiniteQuery({
    queryKey: [...userKeys.lists(), "infinite", filters],
    queryFn: async ({ pageParam }) => {
      return await getUsers({ ...filters, page: pageParam });
    },
    initialPageParam: 1,
    getNextPageParam: (lastPage) => {
      return lastPage.hasNextPage ? lastPage.page + 1 : undefined;
    },
  });
};

// In component:
function UserList() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteUsers({ status: "active" });

  // Flatten pages into single array
  const allUsers = data?.pages.flatMap((page) => page.users) ?? [];

  return (
    <div>
      {allUsers.map((user) => (
        <UserCard key={user.id} user={user} />
      ))}
      {hasNextPage && (
        <button onClick={() => fetchNextPage()} disabled={isFetchingNextPage}>
          {isFetchingNextPage ? "Loading more..." : "Load More"}
        </button>
      )}
    </div>
  );
}
```

### 3. Prefetching Data

Use `prefetchQuery` to fetch data before the user needs it (e.g., on hover).

```tsx
import { useQueryClient } from "@tanstack/react-query";

function UserLink({ userId }: { userId: string }) {
  const queryClient = useQueryClient();

  const handleMouseEnter = async () => {
    await queryClient.prefetchQuery({
      queryKey: userKeys.detail(userId),
      queryFn: () => getUserById(userId),
      staleTime: 1000 * 60, // Keep fresh for 1 min
    });
  };

  return (
    <a onMouseEnter={handleMouseEnter} href={`/user/${userId}`}>
      View Profile
    </a>
  );
}
```

### 4. Suspense Integration

If the project uses React `<Suspense>` boundaries, use `useSuspenseQuery`. It throws promises to the nearest Suspense boundary.

```typescript
import { useSuspenseQuery } from "@tanstack/react-query";

export const useUserSuspense = () => {
  return useSuspenseQuery({
    queryKey: userKeys.detail("me"),
    queryFn: getUserProfile,
  });
};

// In component tree — no need to check `isPending`:
// <Suspense fallback={<Loading />}>
//   <UserProfile />
// </Suspense>
```

### 5. Error Boundaries with `throwOnError`

For critical data where a failure should show a full error boundary:

```typescript
export const useGetCriticalConfig = () => {
  return useQuery({
    queryKey: ["app-config"],
    queryFn: getAppConfig,
    throwOnError: true, // Throws to nearest ErrorBoundary
  });
};
```

---

## Anti-Patterns

| ❌ Don't | ✅ Do |
|---|---|
| `useEffect` + `useState` for fetching | `useQuery` |
| Raw `axios.post()` in click handlers | `useMutation` + `.mutate()` |
| String query keys `"users"` | Array keys `["users"]` / key factory |
| Same key for different data shapes | Hierarchical keys `["users", id]` |
| Invalidate in `onSuccess` only | Invalidate in `onSettled` (covers errors too) |
| `isLoading` (deprecated v5 alias) | `isPending` (initial) + `isFetching` (background) |
| `...options` spread after inline callbacks | Define callbacks cleanly, no merge conflicts |
| Ignoring error states in components | Always handle `error` / `isError` |
| `enabled: data !== undefined` (guard on output) | `enabled: !!id` (guard on input) |
| Manually refetching on a timer | `refetchInterval: 5000` option |
| Large `staleTime` with no invalidation | Pair with proper mutation invalidation |
| `queryKey: ["users"]` scattered across files | `userKeys.all` from key factory |
| Putting toast/redirect in hook definition | Put UI feedback at the `.mutate()` call site |
| `cacheTime` (old v4 name) | `gcTime` (v5 name) |
| `any` in generics (`<T = any>`) | Explicit type param `<T>` — required at call site |
| `any` for request/response data | Typed interfaces (`ApiErrorResponse`, `UserProfileResponse`) |
| `config?: any` in API wrappers | `config?: AxiosRequestConfig` from Axios types |
| `(data as any).message` type assertion | Typed `AxiosError<ApiErrorResponse>` generic |

---

## Dependencies

```bash
npm install @tanstack/react-query @tanstack/react-query-devtools axios react-toastify
# @tanstack/react-query@^5  axios@^1  react-toastify@^10
```

---

## New API Integration Checklist

When integrating a new API endpoint, follow this checklist:

- [ ] **Types**: `Request` + `Response` interfaces defined in service file
- [ ] **Key factory**: Entity keys added to `queryKeys.ts`
- [ ] **Raw function**: Typed, uses `get`/`post`/`patch`/`put`/`del` from `api.ts`
- [ ] **Query hook**: `useGet<Entity>` with key factory key and `queryFn`
- [ ] **Mutation hook**: `useCreate/Update/Delete<Entity>` with `onSettled` invalidation
- [ ] **Component**: Destructures `data`, `isPending`, `error` (not `isLoading`)
- [ ] **Loading state**: `isPending` skeleton or `<Suspense>` boundary
- [ ] **Error state**: Fallback UI or `<ErrorBoundary>`
- [ ] **Mutation UX**: Button disabled during `mutation.isPending`
- [ ] **Toast/feedback**: At the `.mutate()` call site, not in the hook
- [ ] **No `any`**: Zero `any` in types, generics, params, or assertions — use proper interfaces or `unknown`
