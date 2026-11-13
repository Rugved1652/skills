---
name: taste-design
description: Design or refine product-specific interfaces while preserving their design system and existing visual identity. Use for screen composition, dashboards, marketing pages, component presentation, interaction design, visual reviews, and removing generic AI styling; not for unrelated business logic.
---

# Taste Design

Make the interface feel considered for its audience, content, and task. Distinctiveness comes from composition, information hierarchy, useful imagery, and interaction. A new palette, decorative effect, or unusual font is not a substitute for those decisions.

## Authority and continuity

- Read and follow the applicable `design-system` skill before visual changes. Its tokens, typography, semantic layers, component APIs, variants, and CSS ownership rules govern every pattern here. Inspect the actual configuration, theme files, shared primitives, and neighboring screens; a gallery never overrides them.
- Preserve the user's current theme, fonts, structure, content, and interaction conventions unless the requested change requires revising them. A component fix is not permission to redesign the page. Do not infer a rebrand from "make this better."
- In Vayu projects, use the installed Typography and compound component APIs, including overlay titles and close controls. Reusable styles belong in their primitive/variant or token owner; local layout adjustments use existing utilities. Do not invent component props, token names, or imports. Follow the actual design system in other projects rather than introducing Vayu.
- If the design-system skill or tooling is unavailable, inspect the project's existing system directly and state the limitation. Do not invent a competing theme. A genuine missing reusable visual concept must be implemented through the system's documented extension mechanism within the authorized scope.
- Reuse the existing UI library and workspace packages before creating primitives. Do not add fonts, animation libraries, or dependencies merely to look different. Keep app workflows separate from reusable presentation according to the project's folder rules.

## Choose the amount of design work

| Request | Response |
|---|---|
| Small component, alignment, or interaction fix | Follow the current direction; inspect the relevant component and pattern only. No fresh reference hunt or three-direction exercise. |
| New screen inside an established product | Preserve its visual language. Research the unfamiliar task/composition, not a replacement brand. |
| New product page or substantial redesign | Research relevant references and compare three structurally different compositions before choosing one. |
| User supplied a specific composition or reference | Adapt that direction faithfully to the product and system; compare alternatives only to resolve a real conflict, not to reopen the user's choice. |
| Review only | Identify specific hierarchy, usability, continuity, or system violations. Explain their effect; do not implement unsolicited changes. |

## Compose from the task

1. **Establish the brief from evidence.** Identify audience, primary task/action, content shape, density, decision urgency, available assets, and the existing product character. Inspect real content/data and current screens first. Ask only when a consequential unknown cannot be resolved from the project; otherwise state a reasonable assumption and continue.
2. **Lock what must persist.** Note the theme, typography, component vocabulary, navigation model, and working layout conventions. Find the actual presentation problem: an unclear priority, poor grouping, weak imagery, excessive wrapping, hidden action, or mismatched density.
3. **Research when warranted.** Use [reference research](references/research.md) to find examples for the task and component. Inspect a small, contrasting set rather than collect endless links. Extract the spatial or behavioral mechanism, not someone else's brand. Distinguish visual observation, documented behavior, and your inference.
4. **Compare directions when called for above.** For substantial work without a user-specified composition, compare three directions that change organization, emphasis, reading sequence, or interaction. Three palettes on the same split hero are one direction. Judge task clarity, content fit, continuity, accessibility, responsiveness, implementation cost, and distinctiveness. Choose the best fit autonomously; briefly explain what it emphasizes and why the alternatives fit less well. Do not create an approval stop unless a real user decision is required.
5. **Map the choice to the system.** Reuse components and variants, assign semantic layers, retain the type scale, and determine where the change belongs. Decide narrow-screen reading order and key interaction states before decorative detail. If the idea only works by violating the system, adapt the idea or choose another.
6. **Implement and inspect.** Work at the requested scope. Check the rendered result and real interactions with long/short content, narrow/wide screens, and relevant states. Use [review and scenarios](references/review.md); report actual checks and remaining limitations without claiming visual verification from a build or text output.

Keep user-facing explanation short: the task served, the chosen composition, what remains consistent, and the evidence checked. The comparison can be a working note; do not expose lengthy internal deliberation or build extra demo pages to show the alternatives unless requested.

## Find useful patterns without loading the whole library

The library contains **200 original design suggestions in 20 categories**. These are options to adapt, not templates, mandatory features, or claims that a source proved them optimal. Each entry names a mechanism, fit, misuse, and adaptation. Read the relevant section and a contrasting option, not every file. Search by ID/category when tools allow it; each category has its own heading.

| Need | Reference and categories |
|---|---|
| Find and evaluate outside references | [Research](references/research.md): sources, access limitations, evidence notes, supplied examples |
| Navigation, sidebar, search/filter, footer | [Navigation](references/navigation.md): NAV, SID, SRCH, FTR; 40 patterns |
| Hero, section sequence, carousel, modular/bento composition | [Marketing composition](references/marketing.md): HERO, SEC, CAR, GRID; 40 patterns |
| Buttons/CTAs, cards, accordion/tabs, dialog/drawer | [Components and actions](references/components.md): CTA, CARD, DISC, OVR; 40 patterns |
| Dashboard, table, statistics/charts, forms | [Product interfaces](references/product.md): DASH, TBL, STAT, FORM; 40 patterns |
| Typography/content, media, onboarding/empty states, motion/responsive transitions | [Content and behavior](references/content-and-behavior.md): TYPE, MEDIA, STATE, MOT; 40 patterns |
| Audit the result or check the skill's behavior | [Review and scenarios](references/review.md) |

## Avoid replacing one stereotype with another

- Do not default to gradient blobs, glowing panels, beige/black palettes, decorative monospace, tilted mockups, pill labels, tiny gray copy, or three equal feature cards beneath every hero. Each needs a purpose or an established art direction. They are not universally forbidden: keep them when intentional and appropriate.
- Do not replace an existing Inter-based system merely because Inter is common. Improve grouping, proportion, line length, action placement, and content emphasis inside the existing typography. Avoid making operational app headings look like marketing displays.
- Do not treat every section as a card grid, every dashboard as KPI cards plus charts, or every hero as text beside a rotated rectangle. An open list, table, workflow, comparison, image-led composition, or anchored content track may fit better. Familiar layouts are still valid when they serve the task.
- Domain fit is about users and tasks, not costume. A clinical work queue needs unambiguous priorities and status; a restaurant page may emphasize food, menu, location, and reservation. Do not assign automatic hospital colors, restaurant textures, or futuristic software imagery.
- Use available, relevant assets and concrete labels. Do not invent testimonials, achievements, statistical deltas, patient details, business claims, or fake live activity for polish. Clearly distinguish illustrative fixture data from verified product facts.
- Compose coherent pages. Do not paste one unrelated gallery idea into each section. Establish a small set of spatial and interaction rules and vary emphasis within them; leave quiet sections quiet. Do not impose a gimmick quota.

## Make interaction part of the design

Use the applicable animation guidance when motion is involved; retain the design system's timing, effects, and ownership rules. Motion should explain a state change, preserve orientation, or give feedback. It must not delay work, steal focus, or become the content.

Preserve semantic controls, keyboard access, visible focus, meaningful reading order, sufficient contrast, and reduced-motion behavior. Do not rely on hover, dragging, color, or animation alone. Keep layouts functional with long text, zoom, and narrow viewports. Prefer manual carousels; automatic movement needs user control. See the focused interaction notes in the relevant reference rather than inventing another motion system.
