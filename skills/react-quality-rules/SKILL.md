---
name: code-quality
description: Enforce strict React/Next.js TypeScript code quality for Vayu UI consumer apps. Use when writing, reviewing, refactoring, or formatting code; creating pages, containers, components, hooks, API hooks/services, WebSocket/Socket.io hooks and services, forms, tables, or primitives; checking TypeScript safety; preventing hardcoded values; enforcing arrow functions, naming, no any, semantic HTML, stable React keys, memoization discipline, safe data rendering, and alignment with folder-structure and design-system skills.
---

# Code Quality

Apply these rules to all app code. Follow `folder-structure` for file placement and `design-system` for tokens, styling, and compound component patterns.

## Hard Gates

| Rule | Requirement |
| --- | --- |
| Page size | No `page.tsx` over 400 lines; split into `containers/Sections/`. |
| Container size | No container file over 400 lines; split into smaller containers/sections. |
| Functions | Use arrow functions only: `const fn = (...) => {}`. Components are arrow functions too. |
| Types | Zero tolerance for `any`; use explicit types, generics, `unknown`, or narrowed unions. |
| Variables | Use camelCase for variables/functions; PascalCase for components/types/enums. |
| Hardcoded values | Do not hardcode business values, UI copy reused across screens, API paths, statuses, colors, spacing, radii, shadows, or magic numbers. Use constants, config, enums, schemas, or design tokens. |
| Architecture | Keep containers logic-focused and presentational components UI-focused. |
| Design | Follow `design-system` strictly; use Vayu tokens and compound components for primitives or when composition is needed. |
| Structure | Follow `folder-structure` strictly for pages, containers, `ui/`, `api/`, `ws/`, hooks, types, schemas, and columns. |

## Formatting

Use Prettier consistently:

- Single quotes
- 2-space indentation
- 100 character line width
- Trailing commas, ES5 compatible
- Semicolons enabled

## React Rules

- Use unique, stable IDs for list `key` props. Avoid array indexes when a list can add, remove, reorder, filter, or update.
- Use `useMemo` for expensive calculations and `useCallback` for callbacks whose identity causes needless child re-renders; do not memoize trivial code.
- Keep components pure: render JSX from props/state only; do not mutate objects, arrays, globals, props, or external variables during render.
- Never call component functions directly. Render with JSX: `<MyComponent />`, not `MyComponent()`.
- Separate container and presentational responsibilities: containers manage state, data fetching, mutations, effects, and orchestration; presentational components receive props and render UI.
- Use semantic HTML first: `button`, `form`, `label`, `nav`, `main`, `section`, `header`, `table`, etc. Add ARIA only when native semantics are insufficient.
- Avoid `dangerouslySetInnerHTML`. If unavoidable, sanitize before rendering and isolate the usage in a small reviewed component.
- Avoid inline styles. Use CSS modules, styled components, Tailwind/design-token classes, or the project styling pattern.

## TypeScript Rules

- Do not use `any` in source, tests, utilities, or type helpers.
- Prefer precise domain/API types from `types/` and inferred Zod types from validation schemas.
- Use `unknown` for untrusted input, then narrow before use.
- Type exported values, hook returns, API responses, event handlers, and component props.
- Avoid non-null assertions unless the invariant is proven nearby.

## Component Rules

- Keep primitives in `ui/components/`; use compound components in primitives or where API ergonomics require composition.
- Keep domain forms, modals, drawers, popovers, cards, and sections in `containers/`.
- Keep reusable non-domain app UI in `components/`.
- Extract repeated logic into hooks/utilities only when reuse or clarity is real.
- Prefer clear prop names over boolean clusters; use variants/enums when states are mutually exclusive.

## Review Checklist

- File is in the right folder and under the 400-line cap.
- All functions are arrow functions.
- No `any`, hardcoded design values, inline styles, unsafe HTML, or unstable keys.
- Page/container/presentational boundaries are respected.
- Formatting matches Prettier config.
- Semantic HTML and accessibility are handled.
- Design tokens and Vayu composition rules are used.
- Data, API, schema, and table logic follow the folder-structure skill.

## Anti-Patterns

- Named function declarations or directly invoked components.
- `any`, index keys for mutable lists, magic numbers, hardcoded strings shared across features, raw colors, custom shadows, or arbitrary radii.
- Fetching data or mutating state inside presentational components.
- `page.tsx` or container files growing beyond 400 lines.
- Inline table columns, inline schemas, inline API calls, or business logic inside JSX.
- Styling with the `style` prop when the project has CSS modules, styled components, Tailwind, or token classes.
- Putting WS hooks in root `hooks/` instead of `ws/hooks/`.
