---
name: design-system
description: Enforce Vayu UI design tokens, semantic layers, component variants, compound composition, and correct CSS ownership. Use when styling UI, changing colors, radius, shadows, blur, typography, animations, Tailwind classes, component variants, global tokens, or deciding whether CSS belongs in global.css, a primitive component, a variant map, a container, or one-off className usage. Do not use for unrelated CSS questions outside Vayu UI design-system decisions.
---

# Design System

Use Vayu UI's semantic tokens and component APIs before writing custom CSS. First decide where the visual change belongs, then apply the smallest durable change.

## Where CSS Changes Belong

| Change Type                                                | Change Here                                                                               | Do Not                                                   |
| ---------------------------------------------------------- | ----------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| Brand/color/theme changes used across app                  | `global.css`: update `:root`, `:root.dark`, and `@theme` token mapping                    | Scatter `text-blue-*`, `bg-[#...]`, or per-use overrides |
| Token shipped by Vayu CLI/template                         | Also sync `packages/cli/src/templates/tokens.ts`                                          | Update docs CSS only                                     |
| Docs preview token behavior                                | `apps/docs/src/app/global.css`                                                            | Patch random demo classNames                             |
| New semantic color/radius/shadow/blur/font/animation token | `global.css` `@theme`; add light/dark values when color-based                             | Hide it inside component class strings                   |
| New keyframe/animation                                     | `animations.css` plus `@theme --animate-*` mapping                                        | Inline ad hoc animation CSS                              |
| New reusable visual kind for any component                 | Add a typed variant/size/state to that component's primitive API and variant map/classes  | Write the recurring styling in every usage `className`   |
| Existing component style should change everywhere          | Update the component implementation in `ui/components/` or `packages/ui/src/components/*` | Override it at page/container/call-site level            |
| One-off layout/spacing tweak for one screen                | Usage `className`, using tokens/utilities                                                 | Add global token or component variant for a single use   |
| Repeated feature-specific composition                      | Extract container/presentational component                                                | Keep growing page-level class strings                    |

Default rule: if a style appears in more than one place, or represents a named design concept, promote it to a token, variant, or component. If it is truly one place only, use local `className` with existing tokens.

## Token Rules

- Use semantic Tailwind classes generated from tokens: `bg-brand`, `text-brand-content`, `rounded-control`, `shadow-surface`, `backdrop-blur-overlay`, `text-h3`.
- Pair color backgrounds with their content token: `bg-[name] text-[name]-content`.
- Use raw values only when no token exists and the need is truly one-off; otherwise add/adjust the token in `global.css`.
- Keep light/dark values paired in `:root` and `:root.dark`; keep Tailwind names mapped in `@theme`.
- Use `mcp-usage`/`get_design_tokens` when unsure which token exists.

## Semantic Layers

| Layer    | Use For                                              | Classes                             |
| -------- | ---------------------------------------------------- | ----------------------------------- |
| Canvas   | App/page background                                  | `bg-canvas text-canvas-content`     |
| Surface  | Cards, panels, forms, tables, list items, inputs     | `bg-surface text-surface-content`   |
| Sidebar  | Side nav, drawers, rails, mobile menus               | `bg-sidebar text-sidebar-content`   |
| Elevated | Modals, popovers, dropdowns, tooltips, notifications | `bg-elevated text-elevated-content` |

Stack from canvas -> surface/sidebar -> elevated. Do not fake layer separation with random grays or opacity hacks.

## Semantic Effects

| Token Family                                                                          | Use                                                                   |
| ------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| `rounded-control`, `rounded-surface`, `rounded-overlay`, `rounded-full`               | Buttons/inputs, cards/panels, modals/popovers, pills/avatars          |
| `shadow-control`, `shadow-surface`, `shadow-elevated`, `shadow-focus`, `shadow-inner` | Interactive lift, surface separation, overlays, focus, pressed states |
| `drop-shadow-subtle`, `drop-shadow-elevated`                                          | Icons/illustrations                                                   |
| `text-shadow-subtle`, `text-shadow-surface`, `text-shadow-overlay`                    | Text over images/backgrounds                                          |
| `blur-control`, `blur-surface`, `blur-overlay`                                        | Subtle effects, glass surfaces, modal backdrops                       |
| `transition-fast`, `transition-medium`, `transition-slow`, `animate-*`                | Motion timing and named animations                                    |

Avoid nested blur layers. Respect reduced motion; global CSS already disables animations under `prefers-reduced-motion`.

## Component Variants

- Add reusable visual choices as typed variants/sizes/states on the relevant primitive component, not only Button.
- If any component already has variants and the new style will be reused in multiple places, add another variant to that component.
- Update the component's type union, variant class map/classes, defaults, docs/registry examples, and MCP metadata when applicable.
- Keep usage code declarative: `<Component variant="destructive" />`, not `<Component className="bg-red-500 ..." />`.
- If an existing component style should change everywhere, update the component itself; do not patch pages or containers.
- Use compound components for primitive APIs or complex composition: `Button.Icon`, `Button.Text`, `Button.Badge`.
- Use `className` at call sites for layout, width, margin, or a true one-off tweak, not for inventing a recurring variant.

## Decision Examples

- "Make all primary buttons blue": change `--brand`/`--brand-content` in `global.css` light and dark roots, ensure `@theme --color-brand` still maps to them, sync CLI token template if shipping.
- "Add a soft Button kind used in many places": add a `soft` Button variant in the Button type and `variantClasses`; update examples/registry. Same rule applies to Badge, Card, Alert, Modal, Tooltip, and every other component with reusable variants.
- "Make all Alerts less rounded": update the Alert primitive style or the semantic radius token it uses, not every page using Alert.
- "This one modal needs extra max width": use local `className="max-w-..."` on that modal/container.
- "Cards need more radius everywhere": adjust `--radius-surface` or base radius tokens, not every card className.
- "A page hero needs special image text shadow once": use existing `text-shadow-overlay`; only add a token if it becomes a named repeated pattern.

## Anti-Patterns

- Hardcoded hex/rgb, Tailwind palette colors, arbitrary radii/shadows/blur, or magic CSS values when semantic tokens exist.
- New component variants implemented only in usage `className`.
- Global tokens added for a single page tweak.
- Color backgrounds without matching `text-*-content`.
- Changing docs `global.css` but forgetting CLI token template for shipped tokens.
- Mixing layer tokens incorrectly, such as elevated content on surface background.
- Custom shadows/radius scales that bypass `@theme`.

## Typography and proportion

Use `Typography.H1`–`H6`, `Typography.P`, `Typography.Label`, and `Typography.Link` for application headings and text. Keep `Modal.Title`, `Drawer.Title`, and other compound titles for their accessible relationships. Native elements inside primitive implementations are appropriate; avoid replacing application Typography with custom styled heading/paragraph wrappers.

Use semantic heading levels with the default `text-h1`–`text-h6` tokens (30/24/20/18/16/14px at the default base). Page headings should not inherit marketing hero sizes. Card and overlay titles usually need 16–20px. Large display type is an explicit design decision, not an automatic responsive enlargement. Read the optional `taste-design` skill when composing or reviewing a whole screen.

Read `vayu-ui.config.json` before installation. Pass `--cwd` for the intended app or use a workspace-root config with `packagePath` and explicit `paths`. Request `get_install_guide` with those paths/aliases and use imports for copied sources. Use package imports only when the `vayu-ui` package is installed.