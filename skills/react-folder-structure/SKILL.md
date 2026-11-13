---
name: react-folder-structure
description: Organize React and Next.js application files with feature boundaries, colocation, and clear data flow. Use when creating, moving, or reviewing application folders, Turborepo packages, shared UI, contracts, and reusable logic placement.
---

# React Folder Structure

## Choose standalone or Turborepo ownership first

Inspect the workspace root, package-manager configuration, `turbo.json`, package manifests/exports, installed UI libraries, and generator configuration before creating files. Preserve established app layouts. For Next.js App Router use `app/` route conventions rather than creating the `pages/` example below alongside it.

In a Turborepo, read [Turborepo structure and extraction rules](references/turborepo.md) before adding or moving components, hooks, utilities, or API types. That reference defines the complete `apps/` and `packages/` structure and overrides app-local placement below for reusable capabilities.

- Put reusable primitives such as `Button`, `Input`, and `Dialog` into `packages/ui` from the first implementation, even with one app and one current consumer. Reusability is a responsibility, not an import-count threshold.
- Search workspace exports and existing implementations before creating or generating a component. A later app imports the same package primitive; it must not create a duplicate because its local folder is empty. Respect existing UI toolkit ownership and generator target paths.
- Put public API/WS contracts in `packages/contracts`; Fastify and React/Next both import its public exports. Put portable non-UI hooks/helpers into focused packages when appropriate; UI-private helpers stay in `packages/ui`.
- Keep routes, concrete forms, feature workflows, app state and configured clients in the app. Wrap shared primitives with app behavior instead of importing app dependencies into packages.
- Apps import packages by declared dependencies and public exports. Packages must not import app code. Move the existing reusable implementation with its dependencies and consumers rather than copying it.

## Core Principles
- **Colocation**: Keep implementation-specific files close to their owner. In a Turborepo, reusable components belong in their package even when only one parent currently uses them; app-specific children stay colocated with the app feature.
- **Feature-Driven Architecture**: Group by domain/feature (e.g., `auth`, `billing`) instead of technical role. This makes scaling easier as domains grow independently.
- **Public API via Barrel Files**: Use `index.ts` in feature folders to export only what other features need. Internal files remain strictly private.
- **Strict Boundaries**: Consume deliberate public APIs rather than another feature's private files. Keep shared domain behavior domain-owned; use an app-level owner for app-specific sharing and a package for reusable capabilities.
- **Shallow Nesting**: Avoid deep directory trees. Limit folder nesting to 3-4 levels maximum to maintain discoverability.

## App-local Directory Structure (`src/`)

The tree below describes app-owned code and standalone defaults. In a Turborepo, reusable UI/hooks/utils and public contracts use the package owners above; do not create app-local duplicates to fill this tree.

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
- **Ownership Rule**: Extract reusable presentation primitives to `packages/ui` in a Turborepo, or `src/components` in a standalone app. Keep domain workflows in features; share domain presentation through its owning public API or a focused domain package without erasing its meaning.

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
| Importing private components from another feature | Use its public API; reusable primitives belong in `packages/ui` for Turborepo or `src/components/` for standalone apps |
| Huge pages with business logic | Pages should only compose layout and feature components |
| Single massive `utils.ts` file | Create a `utils/` folder and split by domain/purpose (e.g. `date.ts`) |

## Verify package changes

For Turborepo changes, verify declared dependencies, public exports, absence of duplicate primitives/contracts, consumer imports, browser/server boundaries, and the affected packages and apps through their existing lint/typecheck/test/build tasks. Follow the migration checklist in [the Turborepo reference](references/turborepo.md).
