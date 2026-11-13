# Design Review and Scenario Checks

Use the relevant checks for the actual change. This is a review aid, not an instruction to add features, write mirrored tests, or redesign neighboring screens. A pattern count or checklist cannot prove taste, accessibility, or usability.

## Assess the chosen direction

For substantial work, compare three compositions by the following questions. Keep only a short decision summary; no lengthy design report or user approval stop is required.

| Question | Evidence to look for |
|---|---|
| Does it serve the actual task? | The user can identify the purpose, important information, and next action without decoding decoration. |
| Does the composition fit the content? | Information relationships explain the regions, proportions, reading order, and disclosure. The layout survives real text/data. |
| Does it preserve product identity? | Existing fonts, theme, components, navigation conventions, and neighboring screens remain coherent. |
| Is the difference meaningful? | Alternatives change organization or interaction, not just color, shadow, or corner radius. The choice has a task-specific reason. |
| Can it work within the design system? | Each treatment maps to an actual token, primitive, supported variant, or properly owned extension. |
| Does it remain usable across contexts? | Narrow layout, keyboard, focus, zoom, long text, state changes, and motion preferences have viable behavior. |
| Is the complexity earned? | The design adds neither dependencies nor elaborate interaction merely to look original. A familiar simpler option remains acceptable. |

Useful axes for alternatives include **what leads** (task, content, evidence, image, exception), **what stays visible** (navigation, context, controls, selected object), **how users compare** (aligned rows, paired detail, sequence, spatial groups), and **how much is revealed** (overview, detail-on-demand, full working surface). Do not mechanically combine all axes into one screen.

## Check the result in the product

- **Scope and continuity:** Compare the affected screen to its prior state and neighboring screens. An improvement should not silently change the brand, application shell, public API behavior, or workflow.
- **Hierarchy:** Check actual headings, supporting text, units, dates, labels, and action priority. Remove unexplained badges/icons, miniature decorative copy, and containers that add no grouping value. Keep a quiet section when it reads better.
- **System conformance:** Inspect the real tokens, imports, compound primitives, variants, and CSS owner. Check light/dark behavior when supported. Repeated visual treatments must not be implemented through scattered call-site overrides.
- **Content honesty:** Confirm supplied assets, real labels, claims, counts, and status rules. Sample data must be identified as such. Unknown, missing, zero, loading, empty, error, and stale must not be conflated.
- **Responsive composition:** Inspect at the project's narrow/wide target sizes and at intermediate widths where content starts to fail. Test long headings, translated labels if applicable, zoom, and overflowing data. Content order and access to actions matter more than preserving the desktop silhouette.
- **Interaction:** Exercise the changed controls using keyboard and pointer/touch where available. Verify focus visibility/order/return, usable close paths, selection and save state, and access to detail without hover or dragging alone. Do not silently change a component's established semantics.
- **Motion:** Check reduced-motion settings against both CSS and scripted behavior. Ensure content/actions remain available without animation, automatic movement can be controlled, and transitions do not steal focus or block work.
- **Implementation:** Run the consumer project's available relevant lint/typecheck/tests/build for code changes. Use existing checks and investigate actual failures. A passing build does not verify visual quality, contrast, or keyboard behavior.

Report what was changed, why that composition fits, which actual checks ran, and remaining limitations. If no browser/device or runnable app is available, state which visual/interactive checks remain unverified. Do not call a screenshot an interaction test or report a speculative score as proof.

## Scenario exercises for this skill

These are author/reviewer exercises with expected decisions, not claims that a production interface has been tested. Use them after revising the skill. Compare resulting decisions with these boundaries; do not merely match keywords. The named patterns are helpful candidates, not mandated outputs.

### 1. Existing branded application

**Request:** "Improve this billing overview; keep our current design system." Existing theme, font, shared components, and surrounding screens are available.

**Expected decision:** First inspect the current task and pain point. If a substantial composition change is justified, compare an actionable invoice queue, a period-comparison view, and a concise account summary against real requirements. Preserve the product shell and visual vocabulary. Select the one serving the user's actual billing work; do not implement all three.

**Reject:** A fresh palette, new font, gradient hero, changed navigation, or synthetic revenue data justified only as "less generic." If the problem is just spacing, use the small-edit path instead of researching a new direction.

### 2. Hospital operational dashboard

**Request:** "Design the dashboard for staff reviewing today's work." Known fields, permissions, and status rules are provided.

**Expected decision:** Research comparable operational tasks, then compare an exception queue, time schedule, and worklist/detail composition. Select based on how staff actually prioritize and act. Use the existing tokens; keep identity, timing, status, and next action clear. A quiet composition can be distinctive through excellent grouping and progressive detail.

**Reject:** Invented clinical urgency thresholds, fake patient information, automatic hospital colors, hero-sized headings, decorative gradients, or a dashboard of unrelated metrics. Design must not change medical or permission policy.

### 3. Restaurant discovery and booking

**Request:** "Make a restaurant website with our supplied menu, photos, hours, address, and reservation link." Existing brand/system rules apply.

**Expected decision:** Compare booking-first, image-led with practical secondary information, and menu/collection-led compositions. Choose based on visitor task, available assets, and booking capability. Make menu, location, hours, and reservation easy to reach; preserve a coherent small-screen reading order.

**Reject:** Cosmic software imagery, invented awards/reviews, default beige/black or monospace treatment, fabricated booking controls, or copying a gallery restaurant's identity. A supplied expressive dark brand is still valid.

### 4. Service carousel within an existing page

**Request:** "Present our services with the anchored-title carousel relationship in the supplied example." Existing page theme and service content are available.

**Expected decision:** Preserve the anchored introduction and separate content track where space permits; adapt CAR-01 using existing primitives and tokens. On narrow screens put the introduction before readable service content. Verify continuation cues, manual controls, reachable item actions, and focus/visibility handling.

**Reject:** Copying the reference palette/fonts, replacing the whole page, hiding the only critical service link, drag-only navigation, or introducing autoplay as decoration. The user already chose the relationship, so do not force three unrelated alternatives.

### 5. Dense operational table

**Request:** "Make this orders table clearer without losing comparison speed." Real columns, sorting, and API capabilities are available.

**Expected decision:** Inspect the scanning/decision problem, preserve aligned comparison, and improve identity, meaningful grouping, column priority, units, and action placement. Consider an inspector or supporting row detail only if it helps the task. Verify long values, narrow screens, loading/failure, and selection scope where applicable.

**Reject:** Replacing every row with a large marketing card, inventing server filtering, adding decorative sparklines, hiding comparison fields without a path, or weakening data labels to create whitespace.

### 6. Sidebar transition adjustment

**Request:** "Make collapse/expand feel smoother." Existing sidebar, animation tokens, and motion preferences are known.

**Expected decision:** Treat this as a focused interaction change. Use MOT-01 and the installed primitive/animation rules. Preserve navigation, selected item, focus, label access, and user control; provide a usable immediate reduced-motion state. No new reference hunt, palette, shell redesign, or library is needed.

**Reject:** A page-wide entrance sequence, hover-only icon labels, delayed navigation, focus loss, or a claim that disabling CSS animations alone handles all scripted movement.

### 7. New page without brand-specific assets

**Request:** "Create a service page in this new app." Only the project's base design system and factual service text exist.

**Expected decision:** Use the base system as the visual constraint. Research appropriate task/layout references and compare an editorial opening, task-first entry, and evidence-led structure using actual available content. Choose a content-first option if no real media/evidence exists. Ask only if a consequential decision cannot be reasonably inferred.

**Reject:** Importing a new font or style library, inventing social proof, manufacturing a hero illustration dependency, demanding approval for every visual choice, or treating an unknown brand as permission to impose a favorite palette.

### 8. Small button adjustment

**Request:** "Fix the spacing between this button's label and icon." Existing Button API and tokens are available.

**Expected decision:** Inspect the primitive and this usage. Fix the layout using the actual API and appropriate style owner. If the defect affects every Button, fix the primitive; if it is a local composition issue, keep the correction local. Verify long labels/focus and any relevant component checks.

**Reject:** Researching fresh CTA styles, generating three design directions, changing the global theme, adding an unnecessary new variant, or copying the primitive into the app.

## Maintain the pattern library

The library has five collections, each with four categories of ten patterns. IDs are unique and stable: NAV, SID, SRCH, FTR; HERO, SEC, CAR, GRID; CTA, CARD, DISC, OVR; DASH, TBL, STAT, FORM; TYPE, MEDIA, STATE, MOT. Each prefix runs from `01` to `10`.

When updating it, verify valid skill metadata, resolvable local links/anchors, all 200 entries and category coverage, and a mechanism/fit/misuse/adaptation for each entry. Review entries for genuine distinctions: a color swap or a second name for the same mechanism is not a new idea. Do not inflate the count with duplicated text. Keep source observations separate from original suggestions and revisit scenario decisions after substantive rule changes.
