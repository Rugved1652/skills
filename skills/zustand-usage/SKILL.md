# Zustand — State Management Skill

## 1. Installation

```bash
npm install zustand
```

> Works with React 18+, Next.js (App & Pages router), and Vite.

---

## 2. Folder Structure

### Naming Convention

| Type | Suffix | Example |
|---|---|---|
| Any Zustand store file | `Store.ts` | `useAuthStore.ts` |
| Store hook export | `use` prefix + `Store` suffix | `useAuthStore`, `useUserStore` |
| Store type definition | Same name as store + `State` | `AuthState`, `UserState` |

> **Rule:** Every store file is named `use<Feature>Store.ts` — always starts with `use`, ends with `Store`.

---

### Folder Split: `api/` vs `global/`

| Folder | What goes here | Changes when |
|---|---|---|
| `store/api/` | Data fetched from a backend API | A network call runs |
| `store/global/` | App-wide UI state (no API calls) | User interacts with UI |

---

### Full Structure — Route by Route

```
src/
└── store/
    │
    ├── index.ts                        # Barrel: re-exports every store
    │
    │── api/                            # ── SERVER / API STATE ──────────────
    │   │
    │   │   # Route: /users
    │   ├── useUserStore.ts             # holds: users[], isLoading, error, fetchUsers()
    │   │
    │   │   # Route: /products
    │   ├── useProductStore.ts          # holds: products[], filters, pagination, fetchProducts()
    │   │
    │   │   # Route: /orders
    │   ├── useOrderStore.ts            # holds: orders[], fetchOrders(), updateOrderStatus()
    │   │
    │   │   # Route: /dashboard (aggregated stats)
    │   └── useDashboardStore.ts        # holds: stats{}, fetchDashboard()
    │
    └── global/                         # ── UI / APP-WIDE STATE ─────────────
        │
        │   # Used everywhere (layout level)
        ├── useAuthStore.ts             # holds: user, token, setAuth(), clearAuth()
        │
        │   # Used in header / root layout
        ├── useThemeStore.ts            # holds: theme, toggleTheme()
        │
        │   # Used by any component that opens a modal
        ├── useModalStore.ts            # holds: modalId, data, openModal(), closeModal()
        │
        │   # Used by any component that shows a toast
        └── useToastStore.ts            # holds: toasts[], addToast(), removeToast()
```

---

### How to Decide: `api/` or `global/`?

Ask yourself:

- **Does it need a `fetch()` / API call?** → Put it in `store/api/`
- **Is it pure UI state (open/close, dark/light, logged-in user)?** → Put it in `store/global/`

---

## 3. Creating a Store

### Basic Global State Store

```ts
// src/store/global/useThemeStore.ts
import { create } from "zustand";

type ThemeStore = {
  theme: "light" | "dark";
  toggleTheme: () => void;
};

export const useThemeStore = create<ThemeStore>((set) => ({
  theme: "light",
  toggleTheme: () =>
    set((state) => ({ theme: state.theme === "light" ? "dark" : "light" })),
}));
```

### Auth Store (Global State)

```ts
// src/store/global/useAuthStore.ts
import { create } from "zustand";

type User = {
  id: string;
  name: string;
  email: string;
};

type AuthStore = {
  user: User | null;
  token: string | null;
  setAuth: (user: User, token: string) => void;
  clearAuth: () => void;
};

export const useAuthStore = create<AuthStore>((set) => ({
  user: null,
  token: null,
  setAuth: (user, token) => set({ user, token }),
  clearAuth: () => set({ user: null, token: null }),
}));
```

---

## 4. API State Store Pattern

Use Zustand to store fetched data alongside loading/error state:

```ts
// src/store/api/useUserStore.ts
import { create } from "zustand";

type User = {
  id: string;
  name: string;
  email: string;
};

type UserStore = {
  users: User[];
  isLoading: boolean;
  error: string | null;
  fetchUsers: () => Promise<void>;
};

export const useUserStore = create<UserStore>((set) => ({
  users: [],
  isLoading: false,
  error: null,

  fetchUsers: async () => {
    set({ isLoading: true, error: null });
    try {
      const res = await fetch("/api/users");
      const data = await res.json();
      set({ users: data, isLoading: false });
    } catch (err) {
      set({ error: "Failed to fetch users", isLoading: false });
    }
  },
}));
```

### Using it in a component:

```tsx
// src/components/UserList.tsx
import { useEffect } from "react";
import { useUserStore } from "@/store/api/useUserStore";

export const UserList = () => {
  const { users, isLoading, error, fetchUsers } = useUserStore();

  useEffect(() => {
    fetchUsers();
  }, []);

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;

  return (
    <ul>
      {users.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
};
```

---

## 5. Barrel Export (index.ts)

Re-export all stores from a single entry point for cleaner imports:

```ts
// src/store/index.ts
export { useAuthStore } from "./global/useAuthStore";
export { useThemeStore } from "./global/useThemeStore";
export { useModalStore } from "./global/useModalStore";
export { useToastStore } from "./global/useToastStore";

export { useUserStore } from "./api/useUserStore";
export { useProductStore } from "./api/useProductStore";
export { useOrderStore } from "./api/useOrderStore";
```

Usage:

```ts
import { useAuthStore, useUserStore } from "@/store";
```

---

## 6. Persist State (e.g. Auth Token)

Use the `persist` middleware to save state to `localStorage`:

```ts
// src/store/global/useAuthStore.ts
import { create } from "zustand";
import { persist } from "zustand/middleware";

type AuthStore = {
  token: string | null;
  setToken: (token: string) => void;
  clearToken: () => void;
};

export const useAuthStore = create<AuthStore>()(
  persist(
    (set) => ({
      token: null,
      setToken: (token) => set({ token }),
      clearToken: () => set({ token: null }),
    }),
    {
      name: "auth-storage", // key in localStorage
    }
  )
);
```

---

## 7. Key Rules / Best Practices

| Rule | Reason |
|---|---|
| Keep stores small and focused | Easier to maintain and test |
| Separate API state from UI state | API state changes often; UI state is app-wide |
| Use `immer` middleware for complex nested updates | Avoid manual spread for deep objects |
| Don't put derived data in the store | Compute it inside the component or with a selector |
| Use selectors to avoid unnecessary re-renders | `const name = useAuthStore(s => s.user?.name)` |

---

## 8. Selector Pattern (Avoid Re-renders)

```ts
// ✅ Good — only re-renders when `user` changes
const user = useAuthStore((state) => state.user);

// ❌ Bad — re-renders on ANY store change
const { user } = useAuthStore();
```
