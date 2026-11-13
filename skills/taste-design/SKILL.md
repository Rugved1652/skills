---
name: taste-design
description: Design or refine Vayu UI screens with clear hierarchy, restrained typography, useful content, and intentional visual choices. Use for pages, dashboards, forms, and visual reviews that should avoid generic AI styling; preserve the user's brand and existing design direction.
---

# Taste Design

Start from the screen's task and the user's existing visual language. Identify its primary action, essential information, and density before choosing a layout. Keep a distinctive direction when the brief asks for one; restraint does not mean every product must look the same.

## Typography and hierarchy

Use Vayu `Typography.H1`–`H6`, `Typography.P`, `Typography.Label`, and `Typography.Link` for application text. Inspect the installed API or MCP metadata first. Keep native elements inside primitive implementations and preserve semantic native labels when form association requires them. Overlay titles/descriptions belong in their compound primitives so accessible IDs remain connected.

Choose heading levels by document structure, not visual size. Use a single page H1, H2 for sections, and lower levels only for nested sections. Vayu's `text-h1`–`text-h6` tokens provide the application scale: about 30/24/20/18/16/14px at the default base size. A card title or drawer title usually needs 16–20px. Do not turn every panel into a hero with `text-4xl`, `text-6xl`, or viewport-sized text. An explicitly requested marketing hero may use a deliberate larger scale through the supported Typography API.

Do not replace Typography with a locally styled `h4`/`p` component or blanket `unsized` overrides. Reuse the installed scale; change shared tokens only when the entire product needs a new scale.

## Make the composition serve the content

- Establish one main action per task; secondary actions should recede.
- Use spacing, alignment, and a modest number of type weights before adding containers or decoration. A list need not become a grid of cards.
- Use semantic canvas/surface/elevated layers and one purposeful accent. Preserve brand colors supplied by the user.
- Avoid automatic gradients, glowing borders, ornamental blobs, glass panels, excessive pill badges, and icons on every line. Use them only when they carry meaning or match an explicit art direction.
- Use concrete labels and realistic data shapes. Do not invent customer quotes, usage statistics, or claims for polish.
- Place dismiss controls in a header flex row with `min-w-0 flex-1` text and a nonshrinking close button. Use `Modal.Header` / `Drawer.Header` and their built-in close control; avoid floating an X over long titles.

## Review the result

Inspect the actual narrow and wide layouts, long titles, empty states, loading, errors, keyboard focus, and contrast. Ask whether a reader can identify the page purpose and next action without interpreting decoration. Remove elements that do not help. Verify real component APIs and imports, and run the project's relevant checks. If a browser is unavailable, say which visual behavior remains unverified.