# React Folder Structure

## Core Principles
- **Colocation**: Keep files close to where they are used. If a component is only used by one parent, put it in the same directory.
- **Feature-Driven Architecture**: Group by domain/feature (e.g., `auth`, `billing`) instead of technical role. This makes scaling easier as domains grow independently.
- **Public API via Barrel Files**: Use `index.ts` in feature folders to export only what other features need. Internal files remain strictly private.
- **Strict Boundaries**: A feature must never import from the internal files of another feature. If code is shared across features, it must be promoted to the global scope.
- **Shallow Nesting**: Avoid deep directory trees. Limit folder nesting to 3-4 levels maximum to maintain discoverability.

## Global Directory Structure (`src/`)

```text
src/
  assets/        # Static files (images, fonts, global CSS). Group by type (e.g., `assets/icons`).
  components/    # Generic, reusable UI components (Button, Modal, Input). Completely domain-agnostic.
  config/        # Global app config, env vars parsing, third-party library initialization.
  constants/     # Global static values, enums, magic strings used across multiple domains.
  features/      # Domain-specific modules (see Feature Structure below).
  hooks/         # Global custom hooks (e.g., useTheme, useDebounce). No domain logic here.
  layouts/       # Structural page layouts (Sidebar, Header wrapper).
  pages/         # Route entry points (mapping URLs to features). Should contain ZERO business logic.
  providers/     # Global React Context providers (Theme, Auth, QueryClient).
  services/      # Global API clients, external services (e.g., Axios instance, Firebase init).
  store/         # Global client state (Zustand, Redux). Use ONLY for truly global state (e.g., user session, UI theme).
  types/         # Global TypeScript definitions (e.g., generic API response types).
  utils/         # Global pure functions (formatters, parsers).
```

## Feature Structure (`src/features/*`)
Each feature acts as an isolated module containing everything it needs.

```text
src/features/auth/
  api/           # API requests and TanStack Query hooks (login.ts, useUser.ts).
  components/    # Feature-scoped components (LoginForm.tsx). They contain domain logic.
  hooks/         # Feature-scoped logic (useAuthFlow.ts).
  routes/        # Feature-specific route definitions (index.ts).
  schemas/       # Zod/Yup validation schemas for the feature's forms.
  stores/        # Feature-scoped state (Zustand slices specific to auth).
  types/         # Feature-scoped TypeScript definitions (User profile types).
  utils/         # Feature-scoped helpers.
  index.ts       # Public API: exports only what outside modules need.
```

## Structural Context & Guidelines

### Feature Components vs Global Components
- **Global Components (`src/components`)**: Must be "dumb" and domain-agnostic. They should not know about your business logic or fetch their own data. Example: A generic `Table` component.
- **Feature Components (`src/features/*/components`)**: "Smart" components connected to the domain. They can fetch data, contain business logic, and use domain-specific hooks. Example: A `UsersTable` component.
- **Promotion Rule**: If a feature component needs to be used by another feature, strip it of its domain knowledge and move it to `src/components`.

### Testing Colocation
- **Unit/Component Tests**: Place `.test.tsx` or `.spec.ts` files immediately next to the file they are testing. Do not use a separate `__tests__` directory, as it breaks colocation.
- **E2E Tests**: Keep End-to-End tests (Playwright, Cypress) in a root-level `e2e/` folder outside of `src/`, since they test the application as a whole.

### State Management: Store vs Providers
- **`src/store/`**: Use for global client state managers (Zustand, Redux). Ideal for rapidly changing data or data that needs to be accessed outside of React components.
- **`src/providers/`**: Use for React Context. Best for low-frequency updates like Theme, Locale, or Dependency Injection. Avoid using Context for complex, frequently updating state to prevent unnecessary re-renders.

## Component Colocation Pattern
For complex components, create a folder containing all related files.

```text
components/
  DataTable/
    DataTable.tsx        # Main component
    DataTable.module.css # Scoped styles
    DataTable.test.tsx   # Unit tests
    useDataTable.ts      # Component-specific logic / state
    index.ts             # Export { DataTable }
```

## Import Rules

### ✅ Do
- Use absolute paths/aliases for global directories (`@/components/Button`).
- Use relative paths within the same feature or folder (`./LoginForm`).
- Import from a feature's public API (`@/features/auth`).

### ❌ Don't
- Deep relative paths (`../../../../utils/format`).
- Cross-feature internal imports (`@/features/users/components/UserAvatar` -> Use `@/features/users` if exported, or move to global `components/` if shared).
- Circular dependencies between features.

## Naming Conventions
| Artifact | Convention | Example |
|---|---|---|
| Directories | `kebab-case` | `user-profile/` |
| Component directories | `PascalCase` | `Button/` |
| Feature directories | `kebab-case` | `billing-history/` |
| Page/Route directories| `kebab-case` | `dashboard/` |
| Barrel files | `index.ts` | `index.ts` |

## Anti-Patterns

| ❌ Don't | ✅ Do |
|---|---|
| Grouping all API calls in one massive `src/api/` folder | Slice API calls by feature (`src/features/*/api`) |
| Global `types/index.ts` for the whole app | Keep types close to where they are used (feature or component level) |
| Importing private components from another feature | Export shared components from `index.ts` or move to `src/components/` |
| Huge pages with business logic | Pages should only compose layout and feature components |
| Single massive `utils.ts` file | Create a `utils/` folder and split by domain/purpose (e.g. `date.ts`) |
