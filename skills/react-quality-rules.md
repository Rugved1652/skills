# React Quality Rules

## Component Design
- One component per file — file name matches component name (`PascalCase.tsx`)
- Prefer functional components — no class components
- Keep components under ~150 lines. If longer, extract sub-components or hooks
- Single responsibility — a component does one thing well
- Co-locate related files: `Component.tsx`, `Component.module.css`, `Component.test.tsx`, `useComponent.ts`
- Export named components — avoid `export default` (harder to refactor, inconsistent imports)
- Never define components inside other components — causes remount on every render

```tsx
// ❌ Component defined inside another
const Parent = () => {
  const Child = () => <div>remounts every render</div>;
  return <Child />;
};

// ✅ Separate component
const Child = () => <div>stable</div>;
const Parent = () => <Child />;
```

## Props
- Destructure props in the function signature
- Use descriptive prop names — `onSubmit` not `handler`, `isLoading` not `flag`
- Boolean props: prefix with `is`, `has`, `should`, `can` → `isDisabled`, `hasError`
- Event handler props: prefix with `on` → `onClick`, `onSubmit`, `onChange`
- Provide sensible defaults via destructuring, not `defaultProps`
- Avoid prop drilling > 2 levels — use composition, context, or state management
- Spread `...rest` only on the root DOM element, never on child components

```tsx
// ✅ Clean prop destructuring with defaults
const Button = ({ variant = "primary", size = "md", isLoading, children, ...rest }) => (
  <button className={`btn-${variant} btn-${size}`} disabled={isLoading} {...rest}>
    {isLoading ? <Spinner /> : children}
  </button>
);
```

## Hooks
- Custom hooks start with `use` — `useAuth`, `useDebounce`, `usePagination`
- Extract reusable logic into custom hooks — keep components declarative
- Never call hooks conditionally or inside loops
- One concern per hook — `useAuth` handles auth, not auth + routing + analytics
- Return stable references: wrap callbacks in `useCallback`, derived objects in `useMemo`
- Custom hooks go in `hooks/` dir (global) or co-located with the feature

## State Management
- Lift state only as high as needed — not higher
- Prefer derived/computed values over synced state:

```tsx
// ❌ Synced state — stale bugs
const [items, setItems] = useState([]);
const [count, setCount] = useState(0);
useEffect(() => setCount(items.length), [items]); // unnecessary

// ✅ Derived
const count = items.length;
```

- Use `useReducer` for complex state with multiple related fields
- URL state (search params) for filterable/shareable UI state
- Server state via TanStack Query / SWR — never `useEffect` + `useState` for fetching
- Global client state via Zustand / Context — choose one per project, not both
- Never store derived data in state — compute it

## useEffect Rules
- Every `useEffect` must have a clear purpose comment
- Cleanup mandatory for subscriptions, timers, listeners, AbortControllers
- Never use `useEffect` for:
  - Transforming data for rendering (compute inline or `useMemo`)
  - Responding to events (use event handlers)
  - Fetching data (use TanStack Query / SWR)
  - Syncing state from props (derive it)
- Minimize dependencies — extract stable values outside the effect
- Avoid `// eslint-disable-next-line react-hooks/exhaustive-deps` — fix the deps instead

```tsx
// ❌ useEffect for data transform
useEffect(() => {
  setFilteredItems(items.filter((i) => i.active));
}, [items]);

// ✅ Derive inline
const filteredItems = useMemo(() => items.filter((i) => i.active), [items]);
```

## Rendering
- Never mutate state — always create new objects/arrays
- Use `key` on lists — stable, unique IDs, never array index (unless list is static and never reordered)
- Conditional rendering: ternary for two branches, `&&` for single (guard with `Boolean()` for numbers)
- Extract repeated JSX blocks into components — no copy-paste
- Avoid inline object/array literals in JSX props (causes re-renders):

```tsx
// ❌ New object every render
<Map center={{ lat: 0, lng: 0 }} />

// ✅ Stable reference
const center = useMemo(() => ({ lat: 0, lng: 0 }), []);
<Map center={center} />
```

## Performance
- `React.memo()` — wrap components that receive stable props but re-render due to parent
- `useCallback` — for functions passed as props to memoized children or used in dependency arrays
- `useMemo` — for expensive computations, not every variable
- Lazy load routes and heavy components: `React.lazy()` + `Suspense`
- Virtualize long lists (1000+ items) — use `react-virtuoso` or `@tanstack/react-virtual`
- Avoid premature optimization — profile first with React DevTools Profiler

## Event Handlers
- Define handlers inside the component — name them `handleX` (internal) or receive as `onX` (prop)
- Never create arrow functions inside JSX for non-trivial logic:

```tsx
// ❌ Inline complex logic
<button onClick={() => { validate(); submit(); track(); }}>Go</button>

// ✅ Named handler
const handleSubmit = () => { validate(); submit(); track(); };
<button onClick={handleSubmit}>Go</button>
```

- For lists, use `data-*` attributes or closures — avoid creating N functions in a `.map()`:

```tsx
// ✅ Single handler with data attribute
const handleClick = (e) => {
  const id = e.currentTarget.dataset.id;
  selectItem(id);
};
{items.map((item) => (
  <button key={item.id} data-id={item.id} onClick={handleClick}>{item.name}</button>
))}
```

## Error Handling
- Wrap route-level or feature-level trees with `ErrorBoundary`
- Show fallback UI — never a blank screen
- Handle async errors in try/catch within mutation callbacks
- Use `error` state from TanStack Query for server errors — display user-friendly messages

## Project Structure

```
src/
  components/          # Shared/reusable UI components
    Button/
      Button.tsx
      Button.module.css
  features/            # Feature-based modules
    auth/
      components/      # Feature-specific components
      hooks/           # Feature-specific hooks
      services/        # API calls
      schemas/         # Zod schemas (forms)
      forms/           # Form components
      utils/           # Feature-specific utilities
  hooks/               # Global custom hooks
  utils/               # Global utilities
  services/            # Global API layer
  constants/           # App-wide constants
  providers/           # Context providers
  routes/              # Route definitions
```

## Naming Conventions

| Artifact | Convention | Example |
|---|---|---|
| Component file | `PascalCase.tsx` | `UserCard.tsx` |
| Hook file | `camelCase.ts` | `useAuth.ts` |
| Utility file | `camelCase.ts` | `formatDate.ts` |
| Constant file | `camelCase.ts` | `apiEndpoints.ts` |
| Service file | `kebab.service.ts` | `auth.service.ts` |
| Style file | `PascalCase.module.css` | `UserCard.module.css` |
| Test file | `PascalCase.test.tsx` | `UserCard.test.tsx` |
| Constants | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT` |
| Booleans | `is/has/should/can` prefix | `isOpen`, `hasAccess` |
| Handlers (internal) | `handle` prefix | `handleClick` |
| Handlers (prop) | `on` prefix | `onClick` |

## Imports
- Order: React → third-party → aliases (`@/`) → relative → styles
- Use path aliases (`@/components`, `@/hooks`) — no deep relative paths (`../../../`)
- Barrel exports (`index.ts`) only for public API of a feature — not every folder

## Conditional Rendering Patterns

```tsx
// Single condition
{isLoggedIn && <Dashboard />}

// Two branches
{isLoggedIn ? <Dashboard /> : <Login />}

// Multiple branches — extract to map or component
const statusComponent = {
  loading: <Spinner />,
  error: <ErrorView />,
  success: <DataView />,
};
return statusComponent[status] ?? null;
```

## Anti-Patterns

| ❌ Don't | ✅ Do |
|---|---|
| `useEffect` for data fetching | TanStack Query / SWR |
| `useEffect` to sync derived state | Compute inline / `useMemo` |
| `useState` + `useEffect` as state machine | `useReducer` or state lib |
| Components inside components | Separate file/declaration |
| `index` as list key | Stable unique ID |
| Prop drilling > 2 levels | Composition / Context |
| `any` / type suppression | Proper types or `unknown` |
| Giant monolith components | Extract hooks + sub-components |
| Catching errors nowhere | `ErrorBoundary` at route level |
| `export default` | Named exports |
| `// eslint-disable` for hook deps | Fix the dependency array |
| `console.log` in committed code | Remove or use proper logging |
| Inline styles for theming | CSS modules / design tokens |
| Magic strings/numbers | Named constants |
