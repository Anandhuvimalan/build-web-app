# Book 4 — Frontend Experience & Design

Part of a four-book set. Start at `SKILL.md` if you haven't already. This book covers how the product should *look, move, and feel to operate* — design discovery, workflow-first screen design, layout systems (including bento grids), motion discipline, and low-bandwidth resilience. For the big-picture methodology see `methodology.md`; for per-slice implementation discipline see `coding-standards.md`; for document templates see `templates.md`.

Load this book whenever a slice touches UI: designing a screen, choosing a layout system, adding animation, or verifying a UI slice. Pure backend/schema slices don't need it — that's the same context-management principle the rest of the workflow runs on.

This book plugs into the existing phase numbering rather than adding new phases: design discovery extends Phase 1, the Design System document is produced in Phase 3, workflow-first screen design informs Phases 2–6, and the UI verification checklist extends Phase 9.

---

## Purpose

AI-generated frontends have a recognizable house style: a centered container, a hero section, three feature cards in a row, uniform rounded-corner cards for everything, default component-library styling, and animations bolted on wherever they fit. It's not ugly — it's *generic*, and every AI-assisted product that skips design discovery converges on it. This book exists to prevent three specific failure modes:

- **Template convergence** — the UI looks like every other AI-built app because no deliberate design direction was ever chosen. The fix is a mandatory design-discovery step where the direction is *asked for and recorded*, never defaulted.
- **Workflow-hostile screens** — the UI mirrors the database (one entity, one page, one form) instead of the user's task, so completing one real-world job means bouncing across five pages. The fix is designing screens around tasks, with patterns like quick-add and bulk-add built in from the start.
- **Janky motion** — animations that look fine on the developer's machine but stutter on mid-range phones and slow networks, because they animate layout properties or depend on assets that haven't loaded yet. The fix is a hard rule set about what may be animated, and verification on throttled networks as a gate.

The same one-source-of-truth principle applies here as everywhere else in this workflow: the design direction lives in `docs/DESIGN.md` (template in `templates.md`), gets approved once, and every UI slice follows it without re-deciding it. A session that invents a new spacing value or animation style mid-slice is drifting from the source of truth exactly the way a session that invents a new API shape is.

---

## Design Discovery (extends Phase 1)

Before any UI is designed or built, run design discovery as part of — not after — Phase 1's discovery loop. The core rule: **the visual direction is the user's decision to make, not the assistant's default to fill in.** Do not assume a style, and do not silently fall back to the familiar card-grid template. Ask, in roughly this order:

1. **Who operates this UI, and how often?** A back-office tool used 8 hours a day by trained staff wants density and keyboard speed; a public-facing product used once a month by anyone wants large targets, plain words, and zero learning curve. The design bar to state explicitly: *a first-time user should be able to complete the core task without training or a manual.*
2. **What visual direction do they want?** Offer concrete options rather than an open-ended "what style?": a bento-grid layout (see the layout section below), an editorial/typography-led look, a dense data-console look, a minimal utilitarian look — or something modeled on a site they admire. Present these neutrally; don't steer toward a favorite.
3. **Is there a reference website they like?** This is often the highest-signal answer. If they name one, *go look at it properly* (next section) instead of working from a guess about what it probably looks like.
4. **What are the brand constraints?** Existing logo, colors, fonts, tone. If none exist, that's a recorded fact ("no brand constraints; direction chosen fresh in DESIGN.md"), not an invitation to improvise per-slice.
5. **What's the realistic device and network floor?** If a meaningful share of users are on mid-range Android phones or unreliable connections, that's a *design input*, not a deployment detail — it changes how much motion, imagery, and JavaScript the design can afford. Ask; don't assume everyone has the developer's hardware.

Record every answer in `docs/DESIGN.md`. Like all Phase 1 output, an unanswered question is recorded as an open assumption, not silently resolved by picking something.

### Extracting a reference site, not eyeballing it

When the user names a reference site, inspect it in a real browser (with browser tooling such as Claude in Chrome, or DevTools by hand) and extract **concrete measurements** — the difference between "inspired by" and "vaguely reminiscent of" is numbers:

- **Layout grid**: container max-width, number of columns, gutter width, outer margins at each breakpoint. Measure them; don't estimate from a screenshot.
- **Type scale**: the actual font sizes, weights, and line-heights in use for display, heading, body, and caption text — and the ratio between steps.
- **Spacing rhythm**: the base spacing unit (commonly 4 or 8px) and the recurring gaps between sections, cards, and controls.
- **Shape language**: corner radii (uniform or mixed), border and divider treatment, shadow depth, any clipping/masking of images.
- **Color roles**: background layers, surface, text hierarchy, one or two accents — captured as roles, not just a list of hex codes.
- **Motion character**: what animates on the reference site (hover, scroll, page transitions), how fast, and how much. Match the *restraint* too — most admired sites animate far less than people remember.

Write the extracted values into `docs/DESIGN.md` as the project's tokens. This is the anti-hallucination discipline of Phase 13 applied to design: "I looked at the actual site and measured it" versus "I know roughly what that kind of site looks like" are different confidence levels, and only the first one is allowed to become a token.

If page-level layout math is involved (it usually is), do it explicitly: pick the container width, choose the column count and gutter, and compute what's left. For example, a 1,200px container with 12 columns and 24px gutters leaves (1200 − 11×24) ⁄ 12 = 78px per column — so a tile spanning 4 columns is 4×78 + 3×24 = 384px wide. Write the arithmetic into DESIGN.md once, and every session builds tiles that line up instead of re-guessing widths.

---

## Workflow-First Screen Design (informs Phases 2–6)

Design screens around the **user's task**, not around the data model. A schema with `products`, `brands`, and `categories` tables does not imply three separate admin pages — it implies one *"get products into the system"* workflow, and the screen count follows from the workflow.

### The core rule

For every core task identified in Phase 1, count the page navigations required to complete it end-to-end. **More than one screen per task needs an explicit justification** recorded in DESIGN.md — "the form was long so we split it" is not a justification; "step 2 legally requires review of step 1's output" is.

### Patterns that keep one task on one screen

- **Quick-add inside dropdowns.** Any select/dropdown whose options are user-manageable records (brand, category, supplier, tag…) gets an "+ Add new…" affordance that opens an inline popover or small modal, creates the record with the minimum viable fields, and **auto-selects it** in the dropdown on save. The user never leaves the form, never loses entered data, and never has to know that brands live on some other page. This single pattern removes the most common admin-workflow failure: "to add a product I first had to go add a brand, and when I came back my form was empty."
- **Bulk add beside single add.** Wherever records are entered one at a time, ask in discovery whether they also arrive in batches (they usually do: initial catalog import, supplier price lists, event attendee lists). Offer bulk entry *on the same screen* — an inline editable table for a handful of rows, and CSV/paste import for hundreds, with a validation preview (which rows will fail and why) *before* committing anything. Never make bulk import an afterthought page discovered three months post-launch.
- **One place to do the whole job.** An admin "add product" screen contains the product form, the bulk-add entry point, and quick-add on every reference dropdown — everything the task needs, in one place. The same consolidation thinking applies to any workflow: a support agent's case screen shows the case, the customer's history, and the reply box together, because that's one task.
- **Inline edit where the data lives.** If users will routinely correct a field (a price, a status, a quantity), let them edit it in the table/list where they see it, rather than navigating to a detail page, editing, saving, and navigating back.
- **Undo over confirmation, where reversible.** "Are you sure?" dialogs train users to click through them. For reversible actions, act immediately and offer a brief undo. Reserve confirmation dialogs for genuinely destructive, irreversible operations — and make those spell out what's about to happen.
- **Protect half-done work.** Any form worth more than 30 seconds of typing must survive an accidental navigation, a dropped connection, or a closed tab — draft state, local persistence, or at minimum a leave-warning. Losing entered data is the fastest way to make users distrust the whole product.
- **Sensible defaults and remembered context.** Default every field that has a predictable answer (today's date, the user's own branch, the last-used category). The measure of a good form is how few fields the user must actually touch in the common case.
- **Keyboard flow for repeat users.** If someone will do this task fifty times a day, tab order, Enter-to-submit, and type-ahead in dropdowns are core features, not polish.

### Empty, loading, and error states are part of the design

Every screen gets designed in four states, not one: **empty** (first use — what tells the user what to do here?), **loading** (skeletons that match the final layout, not spinners in a void), **error** (what went wrong, in plain words, with a next step), and **populated**. A screen designed only in its populated state is half-designed, and the missing states are exactly what real users hit first. Slice specs for UI slices should name these states in their acceptance criteria.

---

## Layout Systems — Bento Grids and the Alternatives

The layout system is chosen once, in design discovery, and recorded in DESIGN.md. This section covers bento grids in depth because they're a strong modern default for mixed-density content — but the choice is still the user's, made from options, not a house default.

### What a bento grid is, and when it fits

A bento grid is a composed arrangement of unequal tiles — different spans, different heights — inside one deliberate grid, like a bento box. Its strength is **hierarchy**: the important thing is literally bigger. It fits dashboards, landing/feature pages, portfolio and showcase layouts, and admin overview screens. It does *not* fit long forms, dense record tables, or reading-heavy pages — forcing those into tiles hurts the task. A product can (and usually should) mix systems: a bento overview screen leading into conventional full-width working screens.

What separates a real bento from "cards in a grid": intentional asymmetry (a 2×2 anchor tile among 1×1s, a full-width strip, a tall 1×2), tiles whose *content shape* matches their tile shape (a chart gets a wide tile; a single KPI gets a small one), and a single consistent gutter creating one visual object out of many tiles.

### Building it: explicit spans, explicit breakpoints

Implement bento with CSS Grid and **explicit span decisions**, not auto-flowing equal cards:

- Define the grid once: `grid-template-columns: repeat(12, 1fr)` (or 4/6/8 columns for simpler compositions) with a single `gap` token from DESIGN.md.
- Give each tile a deliberate span (`grid-column: span 4`, `grid-row: span 2`). Use `aspect-ratio` or fixed row units (`grid-auto-rows`) to keep tile heights composed rather than content-driven chaos.
- **Responsiveness is a re-flow decision per breakpoint, not shrinking.** At each breakpoint, decide the new composition: the 12-column desktop bento becomes perhaps a 4-column tablet bento, and on mobile a deliberate *stack order* — which tile comes first when everything is one column? That's an information-hierarchy decision; make it explicitly (with `grid-template-areas` per breakpoint or reordered spans), don't let source order decide by accident.
- **Flexible bento**: when tile count is dynamic (user-configurable dashboards), define span *rules* per tile type (a KPI is always 3×1, a chart is always 6×2) plus `grid-auto-flow: dense` to fill gaps — the composition stays intentional even when the contents vary. Container queries let a tile adapt its internal layout to the tile's own size rather than the viewport's.
- **Clipped/shaped ("cliparted") bento**: the modern editorial variant — images or accent tiles clipped into non-rectangular shapes with `clip-path`, oversized asymmetric corner radii (one corner 48px, the others 12px), or shapes that visually bridge adjacent tiles. Use it on *accent* tiles, not every tile — the contrast against calm rectangular neighbors is what makes it look designed. Keep `clip-path` static or animate only its container's transform; animating clip-path geometry itself is expensive.
- **Animated bento**: entrance as a short stagger (tiles fade/rise 20–30px, ~50–80ms apart, transform + opacity only); hover as a subtle lift or scale (1.01–1.03) on the tile; expanding/reordering tiles via FLIP-based layout animation (Framer Motion's `layout` prop, GSAP's Flip plugin) so tiles glide between positions instead of snapping. Everything in the motion section below applies with extra force here, because a bento animates many elements at once.

### The alternatives, so the choice is a real one

Offer these alongside bento in discovery, each in one honest line: **editorial/typography-led** (large type, generous whitespace, few images — content-heavy products, brand-forward marketing); **dense console** (compact tables, minimal chrome, keyboard-first — heavy-use internal tools); **minimal utilitarian** (few elements, high contrast, near-zero decoration — single-purpose tools); **reference-derived** (extracted from a site the user loves — often the best answer when it exists).

---

## Motion & Performance Discipline

Animation is the most common place AI-built frontends feel cheap: technically present, physically wrong. These rules are hard gates, not taste.

### What may animate

- **Animate only `transform` and `opacity`.** These run on the compositor thread and stay smooth even while JavaScript is busy. Animating `width`, `height`, `top`, `left`, `margin`, or `padding` forces layout recalculation every frame — that is where "sticky" stuttering comes from, and it gets worse precisely on the low-end devices and busy main threads where users will notice.
- When something genuinely must move between layout positions or sizes, use the **FLIP technique** — measure First and Last positions, apply the Inverted transform, Play it back as a pure transform animation. Framer Motion's `layout` prop and GSAP's Flip plugin do this correctly out of the box; hand-rolled layout animation usually doesn't.
- Use `will-change` sparingly and only on elements about to animate — blanket `will-change` consumes memory and can make things slower.
- **Timing**: micro-interactions (hover, press, toggle) 100–200ms; element entrances 200–400ms; anything over 500ms had better be a deliberate moment. Ease-out for things appearing, ease-in-out for things moving. Define two or three duration/easing tokens in DESIGN.md and reuse them everywhere — uniform motion is what makes a UI feel coherent.
- **Respect `prefers-reduced-motion`** — collapse non-essential animation to simple fades or nothing. This is an accessibility requirement, not a preference.

### Library choice, stated neutrally

Plain CSS transitions handle most micro-interactions with zero bundle cost — prefer them for hover/focus/press states. Reach for a library when you need what it's actually for: **Framer Motion** (React) for declarative component animation, presence (mount/unmount) animation, and FLIP layout animation; **GSAP** for complex timelines, scroll-driven sequences, and framework-independent control. Both are legitimate defaults; pick per the project's framework and the animation's complexity, record the choice as a Decision Log entry, and don't ship both without a stated reason. Whichever is chosen, **lazy-load animation code** with the routes that use it — animation libraries must never be in the critical path of first paint. And per Phase 8's dependency rules, an animation library is a dependency like any other: justify it, check maintenance, isolate usage behind the project's own motion tokens/components so it can be swapped.

### Designing for the slow network, not the demo

The user's phrase for the failure is exact: on low internet, things get "stucky." That happens when the design *assumes* assets and data arrive instantly. Build the opposite assumption in:

- **Never animate while waiting on the network.** Data-dependent content gets a **skeleton that matches the final layout's exact dimensions**; when data arrives it fades in (opacity only). An entrance animation that starts when data lands looks broken when data lands late.
- **Zero layout shift by construction.** Every image gets explicit dimensions or `aspect-ratio`; embeds and dynamic sections get reserved space; fonts load with `font-display: swap` and metric-compatible fallbacks. Cumulative layout shift is jank users can't articulate but always feel.
- **Ship less on the critical path**: code-split by route, compress and appropriately size images (modern formats, responsive `srcset`), lazy-load below-the-fold media, keep the animation library out of the initial bundle.
- **Optimistic UI for the workflow patterns above** — quick-add and inline edits should reflect immediately and reconcile with the server in the background, with a visible, recoverable error path if the server disagrees. On a slow connection this is the difference between "fast app" and "frozen app."
- **Test throttled, on purpose.** Part of UI verification (below) is driving the feature with DevTools network throttling (Slow/Fast 3G) and CPU throttling (4–6×). If the design only feels good on the developer's machine and fiber connection, it isn't verified.

---

## UI Slice Verification (extends Phase 9)

For any slice with a UI surface, Phase 9's end-to-end verification step expands to this checklist — same spirit: actually run it and watch it, don't assert it:

- [ ] Driven in a real browser at **three viewports minimum** (a small phone ~360px, a tablet ~768px, a desktop ~1280px+) — layout re-flows as designed at each, nothing overflows or overlaps
- [ ] **Empty, loading, error, and populated** states all reached and all look intentional
- [ ] Driven once with **network throttling (Slow 3G) and CPU throttling** — skeletons appear, nothing shifts when content lands, animations stay smooth or gracefully absent
- [ ] **No layout shift** on load (watch for it explicitly; images/fonts/embeds have reserved space)
- [ ] Animations follow the transform/opacity rule; nothing stutters during entrance or interaction
- [ ] **`prefers-reduced-motion` verified** to actually reduce motion
- [ ] **Keyboard-only pass**: every interactive element reachable and operable, focus visible, tab order sane; dropdown quick-add and modals trap and return focus correctly
- [ ] Hover, focus, active, and disabled states present on interactive elements
- [ ] Text contrast meets WCAG AA; touch targets are comfortably tappable (~44px)
- [ ] Any form worth >30 seconds of input survives an accidental refresh/navigation (draft protection verified, not assumed)
- [ ] The screen matches `docs/DESIGN.md` tokens — spacing, type, radii, motion — with no per-slice inventions

The workflow-level check, once per task rather than per slice: walk the *entire core task* end-to-end as a first-time user and count the steps and page hops. If the count exceeds what DESIGN.md promised, that's a plan-vs-reality conflict — handle it via Phase 14, don't quietly accept the worse workflow.

---

## Anti-Patterns (the house style to actively avoid)

- **The default AI template**: centered container, hero, three feature cards, uniform card grids for everything. If the delivered UI would look at home in any other AI-generated app, design discovery was skipped — go back and do it.
- **Component-library defaults shipped as the design.** Libraries are fine as *foundations*; unthemed defaults are a visible tell. The DESIGN.md tokens must be applied over whatever library is used.
- **One entity, one page.** Admin UIs that mirror the schema instead of the task — the quick-add/bulk-add section exists specifically to kill this.
- **Scroll-triggered animation on everything.** Reserve scroll animation for a few deliberate moments, if any; ambient constant motion reads as noise and costs performance exactly where it's least affordable.
- **Spinner-only loading** and **animations that gate content** (users wait for the fade-in to finish before they can act).
- **Confirmation dialogs as a safety blanket** on reversible actions, instead of undo.
- **Per-slice style invention** — a new gray, a new gap value, a new duration "just this once." That's source-of-truth drift; the tokens live in DESIGN.md.
- **Designing only the happy, populated, fast-network state.** Real first impressions are empty states on slow connections.
