# Reference Research

Read this for new screens, substantial redesigns, or unfamiliar interaction problems. Small edits should follow the current direction. Research informs a choice; it is not an approval ceremony or permission to replace the design system.

## Route by the problem

| Problem | Source | What to investigate |
|---|---|---|
| Navbar, dropdown, mega menu, sidebar, breadcrumbs | [Navbar Gallery](https://www.navbar.gallery/) | Navigation hierarchy, placement, disclosure, and desktop/mobile relationships. Choose the relevant category rather than browsing only popular homepages. |
| Button, purchase, download, form, newsletter, pricing CTA | [CTA.gallery](https://www.cta.gallery/) | Where the action appears, what context accompanies it, and how commitment is communicated. Look at the matching task/industry. |
| Modular layout or unequal content emphasis | [Bento Grids](https://bentogrids.com/) | Grouping and span choices. Check whether unequal modules communicate different importance or merely fill a rectangle. |
| An already-dark product or an explicit dark brief | [Dark Design](https://www.dark.design/) | Contrast, surface hierarchy, imagery, and navigation within a dark environment. Not a reason to turn a light product dark. |
| Closing navigation, contact, resources, final action | [Footer Design](https://www.footer.design/) | Information structure and transition from the last section; filter by relevant footer type. |
| Dashboard/workflow examples from products | [Mobbin dashboard study](https://mobbin.com/blog/what-good-dashboard-design-looks-like) | Real screen/task context; inspect underlying examples where accessible. Editorial popularity is not proof of end-user usability. |
| Data-heavy operational interfaces | [Pencil & Paper dashboard patterns](https://www.pencilandpaper.io/articles/ux-pattern-analysis-data-dashboards) | Orientation, investigation, drill-down, and task completion, rather than a collection of attractive graphs. |
| Different dashboard organizations | [Dashboard Design Patterns](https://dashboarddesignpatterns.github.io/patterns.html) | Structure, screen space, representation, and interaction tradeoffs. The accompanying [research paper](https://arxiv.org/abs/2205.00757) provides the study context. |
| Table behavior and presentation | [Carbon data table guidance](https://www.carbondesignsystem.com/components/data-table/usage/) | Comparison, row actions, selection, expansion, and density. Borrow interaction principles, not Carbon styling or dependencies. |
| Hierarchy, alignment, content emphasis | [NN/g visual design principles](https://www.nngroup.com/articles/good-visual-design/) | How visual priority follows information priority. Retain the project's typography and spacing system. |
| Carousel keyboard/rotation behavior | [WAI carousel pattern](https://www.w3.org/WAI/ARIA/apg/patterns/carousel/) and [animation tutorial](https://www.w3.org/WAI/tutorials/carousels/animations/) | Controls, focus, access to slides, and user control of automatic movement. |
| Reduced motion | [WAI animation from interactions](https://www.w3.org/WAI/WCAG22/Understanding/animation-from-interactions) | Avoid making optional movement necessary to understand or use the interface. |

These sources were consulted during authoring on 2026-11-13. Gallery indexes and text were accessible; that does not establish the live behavior of every featured site. Bento Grids did not expose readable content to the text browser. Do not claim it was visually studied. Access and collections may change; follow working redirects and record what was actually inspected.

## A bounded research pass

1. Name the task and constraints first: e.g. "compare service categories without changing the current brand" or "triage overdue orders in a dense workspace."
2. Inspect two to four relevant examples with contrasting mechanisms, preferably including a live product or interaction guideline when behavior matters. Browse only the source categories needed. Stop when the alternatives and their tradeoffs are clear, rather than filling a moodboard quota.
3. For visual judgments, inspect actual screenshots or rendered pages using available tools. Check a narrow layout and the key interaction if accessible. A gallery category label, HTML description, or search snippet is not a screenshot review; a still image cannot prove hover, scrolling, keyboard, or animation behavior.
4. Keep compact working notes: **URL; observed mechanism; useful task fit; limitation; adaptation to current tokens/components**. Mark documented behavior and inference separately. Do not send private app data or screenshots to third parties just to obtain inspiration.
5. Compare directions and select one. Carry over relationships such as an anchored heading, unequal content weights, a secondary rail, or persistent controls. Do not copy logos, proprietary illustrations, text, photos, code, or a site's exact visual identity.

If a site is blocked, login-only, unavailable, or cannot be visually inspected, state the access limit. Use another accessible source, supplied images, or the project's existing patterns. Do not invent evidence, bypass access controls, require paid access, or block an ordinary edit on a gallery. When external access is prohibited, use local evidence and the pattern library with that limitation stated.

## Read the supplied examples as relationships

The following observations come from the user's screenshots, not a claim of testing those sites live. The structural adaptations are original suggestions. No screenshot assets need to be bundled with this skill.

### Anchored introduction beside a service track

The service example gives the heading and paired controls a stable left-hand region. Content cards move through a separate right-hand track; a partial next item indicates continuation. This is a hierarchy decision, not simply "make three cards."

Adapt it when readers should explore categories without losing the section's context. Keep the heading, card text, and actions in existing tokens/components. On narrow screens put the introduction before the track, allow touch and keyboard access, and prevent offscreen actions from becoming invisible focus targets. Important services must remain discoverable; do not hide the only path to a critical action in a slide. See CAR-01 in [marketing composition](marketing.md#carousels-car).

### CoLabs integrated composition

The supplied CoLabs image places navigation over a dominant media area, uses shaped boundaries to make room for a headline, and gives secondary content a separate rail. Its visual identity emerges from how regions meet and share space. The [CoLabs site](https://colabs.com.au/) is a source for further observation, not an instruction to clone its image treatment.

Adapt the spatial relationship only when assets, content, and brand support it. Build a readable linear mobile version. Focus outlines, hit areas, and real content must survive any mask or notch. Use existing system radii/effects or a permitted local composition; do not modify global radius tokens to force a reference's silhouette. A conventional rectangle is preferable to clipping meaningful content. See HERO-03 and HERO-04 in [marketing composition](marketing.md#heroes-hero).

### Tilted panel plus small labels and three features

The other supplied hero and lower section show a repeated composition the user wants to avoid as an automatic answer: a split headline/tilted graphic followed by three equal text blocks. The issue is reflexive reuse, weak supporting-text emphasis, and limited task-specific reasoning. It is not a universal prohibition on split layouts, green, cards, supporting copy, or a particular font. Redesign the reading sequence and content priorities before changing visual ingredients.

## Provenance and adaptation

The 200 numbered library entries are independently written options, not reproductions of a gallery's proprietary catalog or findings that each option is universally best. Source links support exploration and specific behavioral guidance. Do not attribute an original pattern suggestion to a source without inspecting an example that actually demonstrates it.

Choose relevant imagery already available or requested by the user. If a concept depends on an unavailable asset, use an honest content-first alternative or explain the dependency; do not silently add fabricated social proof or unrelated stock imagery.
