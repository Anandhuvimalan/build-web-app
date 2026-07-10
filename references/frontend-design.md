# Book 4 — Frontend Experience & Design

Part of a four-book set. Start at `SKILL.md` if you haven't already. This book covers how the product should *look, move, and feel to operate* — design discovery, workflow-first screen design, layout systems (including bento grids), motion discipline, and low-bandwidth resilience. For the big-picture methodology see `methodology.md`; for per-slice implementation discipline see `coding-standards.md`; for document templates see `templates.md`.

Load this book whenever a slice touches UI: designing a screen, choosing a layout system, adding animation, or verifying a UI slice. Pure backend/schema slices don't need it — that's the same context-management principle the rest of the workflow runs on.

This book plugs into the existing phase numbering rather than adding new phases: design discovery extends Phase 1, the Design System document is produced in Phase 3, workflow-first screen design informs Phases 2–6, and the UI verification checklist extends Phase 9.

---

## Purpose

AI-generated frontends have a recognizable house style: a centered container, a hero section, three feature cards in a row, uniform rounded-corner cards for everything, default component-library styling, and animations bolted on wherever they fit. It's not ugly — it's *generic*, and every AI-assisted product that skips design discovery converges on it. This book exists to prevent four specific failure modes:

- **Template convergence** — the UI looks like every other AI-built app because no deliberate design direction was ever chosen. The fix is a mandatory design-discovery step where the direction is *asked for and recorded* — and derived from the project's own subject matter, never defaulted or borrowed wholesale from a reference.
- **Workflow-hostile screens** — the UI mirrors the database (one entity, one page, one form) instead of the user's task, so completing one real-world job means bouncing across five pages. The fix is designing screens around tasks, with patterns like quick-add and bulk-add built in from the start.
- **Uncalculated geometry** — containers, gaps, and small recurring components (dropdowns, buttons, chips, icon-and-text pairs) placed by feel instead of arithmetic, so nothing quite lines up and small elements read as "off" even when no single rule is technically broken. The fix is treating every container's internal spacing and every small component's anatomy as a calculation, not a guess (see Geometry & Spatial Composition and Component Micro-Design below).
- **Janky, undirected motion** — animations that look fine on the developer's machine but stutter on mid-range phones and slow networks because they animate layout properties or depend on assets that haven't loaded yet; entrances that fire independently instead of as one composed sequence; a scroll effect or page intro bolted onto every project by reflex instead of chosen because the content earns it. The fix is a hard rule set about what may animate, an explicit choreography model for parent/child motion, and verification on throttled networks as a gate.

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

### The design concept is derived from the project, not chosen from a menu

A bento grid, an editorial layout, a dense console — those are *systems*, not concepts. The concept is the specific idea that makes *this* project's execution of whichever system unmistakably itself, and it comes from the project's own subject matter, not from a style preference layered on top afterward. Two products can both choose "editorial" and still look nothing alike if one derives its concept from a rare-book archive and the other from a running club — that difference is the whole point, and it's what separates a designed product from a themed template.

Before drawing anything, answer three questions about the content itself:

1. **What is the dominant content shape?** Long-form text, dense tabular data, a timeline of events, a small number of high-stakes numbers, a visual/image-heavy catalog, a real-time feed. The dominant shape should be visible in the layout's proportions before a single color is chosen — a product whose core content is six KPIs doesn't get a fifty-row dense table as its home screen, and a product whose core content is a reading list doesn't get a bento grid of equal-weight tiles.
2. **What is the domain's own visual vocabulary?** Not a mood board of unrelated inspiration — the vocabulary the domain already has. A trading tool inherits the visual precision of instrument panels (tabular figures, hairline rules, no wasted motion). A children's education product inherits playfulness and a forgiving, soft-edged feel. A legal or medical record system inherits restraint and unambiguous hierarchy, because misreading a UI has a real cost there. Name the domain's own visual language before importing someone else's.
3. **What is the one idea a first-time viewer should get in the first three seconds?** Not a tagline — a *visual* idea: "this is fast," "this is exhaustive and trustworthy," "this is built for one specific expert user," "this is playful and forgiving of mistakes." Every layout, type, and motion decision downstream should be traceable to this idea. If a decision isn't traceable to it, it's decoration, not design.

Record the answer as the project's **design concept** in `docs/DESIGN.md`, in one or two sentences, before the Direction section. Everything else in this book — layout system, spacing, motion character — is downstream of this concept, which is what keeps two different projects that both chose, say, a bento grid, from looking like reskins of each other.

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

## Geometry & Spatial Composition (extends Phase 3)

Generic-feeling UI is rarely one big mistake — it's a hundred small containers whose padding, gaps, and internal alignment were each eyeballed independently, so nothing shares a rhythm and small inconsistencies compound into a feeling of "not quite right" that's hard to point at. Treat every container's internal geometry as arithmetic, not feel — the same discipline the layout-grid math in Design Discovery applies at the page level, applied here at the level of every individual box.

### Every container is a calculation

For any box that holds content — a card, a button, a chip, a modal, a table cell — its dimensions are a function of its content plus the spacing scale, not an arbitrary fixed value:

- **Internal padding comes from the spacing scale, and top/bottom don't have to equal left/right.** Text sits optically better with slightly less top padding than bottom (ascenders and cap-height already read as "up" to the eye), and buttons usually read better a touch wider than tall. `16px` vertical / `16px` horizontal around a single word often reads as cramped and squarer than `12px` vertical / `20px` horizontal around the same word — check both, don't default to a square value out of habit.
- **A container's minimum size is derived from its shortest *and* longest realistic content, not its current placeholder content.** A card designed around a two-word label breaks the day real data brings a twelve-word label. Verify each recurring container against both extremes, not just what's in the mock.
- **Gaps between siblings are one step of the spacing scale, not a feeling.** If the scale is `8/16/24/32/48`, every gap in the product is one of those five numbers — never a `13px` that "looked right" in one spot. A spacing value invented per-container is source-of-truth drift, exactly like an invented API shape.
- **Nested containers inherit rhythm; they don't reset it.** A card inside a section should compound the spacing scale predictably (e.g., outer section `48px`, card padding `24px`, card-internal element gap `8px`) rather than each level choosing its own scale independently. If the nesting is deep enough that this gets hard to track, that's a signal the composition itself is too deep, not just a spacing problem.

### Optical alignment beats mathematical alignment

Pixel-identical coordinates and *visually apparent* alignment are not always the same thing, and the eye is the ground truth:

- **Icon-and-text pairs need optical, not bounding-box, centering.** A geometric icon (a circle, a square glyph) centered by its bounding box next to text often reads as sitting slightly high; nudge it 1–2px to compensate. This is exactly the kind of "minor stuck-looking" imprecision that's invisible in a spec but obvious on a real screen.
- **A trailing icon inside a control — a dropdown's chevron, a button's arrow — aligns to the control's own padding, not to the control's edge and not to its horizontal center.** It sits the same distance from the right edge as the text sits from the left edge (mirrored padding). An icon jammed against the border reads as a rendering bug; an icon floating with no stated relationship to the text reads as unfinished. Full anatomy in Component Micro-Design, below.
- **Rounded shapes need larger optical margins than sharp ones** to read as equally spaced — a circular avatar next to a square card edge needs a few extra pixels of clearance to look as close as two square edges at the mathematically identical gap.
- **Text blocks keep a measure of 45–75 characters per line** regardless of how wide the container is; a full-bleed paragraph in a wide container is a container-sizing bug, not a typography detail — cap the text element's own width even inside a wider parent.

### Fitting the viewport on purpose

- **Compute what fits before deciding what's shown**, the same way the layout-grid math computes column widths: for any fixed-height composition (a hero, a dashboard's above-the-fold section, a modal), sum the actual heights of its real content — type line-heights, image aspect ratios, spacing — against the target viewport height at the smallest supported viewport, and confirm it fits or make an explicit scroll/truncation decision. Don't discover the overflow live.
- **Safe-area and scrollbar-width awareness are arithmetic, not an afterthought.** A composition centered against the *window* width instead of the *content* width (window minus a scrollbar that may or may not be present) visibly jumps depending on whether a scrollbar happens to be showing. Account for it explicitly.
- **A container's aspect ratio is declared, not incidental** — `aspect-ratio` (or an equivalent reserved-space technique) on anything that will hold async content, so the calculated geometry survives content arriving late, not just the empty-state mock.

This section pairs with the layout-grid math in Design Discovery — that math computes the page-level grid; this section computes what happens *inside* every cell of it.

---

## Component Micro-Design: Small Controls

Small, frequently-repeated components — selects/dropdowns, buttons, chips, badges, inputs, tooltips, menus — do more damage to a product's perceived quality than any single hero section, because the user touches them constantly and any imprecision compounds across dozens of instances on one screen. Each of these gets a deliberate anatomy, not a browser default or an unthemed component-library default.

### Anatomy of a select / dropdown

This is the component named most often as a symptom of generic AI output, and it earns a full spec:

- **Trigger control**: padding follows the container-padding rule above (slightly more horizontal than vertical). The trailing chevron sits at a fixed distance from the right edge equal to the control's own horizontal padding — not flush against the border, not centered in leftover whitespace unrelated to the text. The chevron is vertically centered against the *text's* cap-height, not the control's full box height including padding, which is where "the arrow looks slightly off" usually comes from.
- **Icon behavior on open**: the chevron rotates 180° (or morphs to a caret-up) over the same duration as the panel's entrance, never as a separate, unsynced tween — a chevron that snaps instantly while the panel eases open reads as two components that don't know about each other.
- **Panel entrance**: opens from the trigger's edge (`transform-origin` set to the trigger, e.g. `top center` for a downward panel) with a combined transform (scale ~0.96→1 and/or translateY 4–8px) plus opacity, 120–180ms, ease-out — never a hard cut, never a slide from a direction unrelated to the trigger.
- **Option list**: options may stagger in on first open only (~15–20ms apart, capped at roughly 8 items' worth of stagger before the rest appear together — staggering fifty rows looks slow, not smooth); re-opening the same panel should not re-run the stagger every time, which reads as the UI performing rather than responding.
- **Viewport collision**: the panel flips above the trigger, or clamps/shifts horizontally, when it would otherwise overflow the viewport — computed at open time against actual available space, not assumed to always have room below.
- **Selected state**: the current selection is visually distinct in the list (not just a checkmark that requires scanning every row), and on open the list scrolls so the current selection is already in view — not always pinned to the top.
- **States**: default, hover, focus-visible (keyboard), open, disabled, and — if async — loading, each styled from a DESIGN.md token, not an unstyled browser default peeking through in one of them.

### The same discipline, generalized

Every other small control follows the same method — derive the anatomy from first principles, don't accept a default:

- **Buttons**: icon-plus-label buttons align the icon's optical center against the label's cap-height, with a gap from the spacing scale (commonly its smallest step) between them — never an icon touching the label or floating with a gap larger than the button's own padding.
- **Chips/tags**: internal padding scales with the chip's role (a removable filter chip needs room for its own close icon without the label crowding it); a row of chips wraps with the same gap value on both axes.
- **Tooltips**: appear after a short delay (~300–500ms, so they don't flash on incidental hover), position against the trigger with a small offset and a directional arrow pointing at the trigger's actual center, and flip when they'd overflow — the same collision rule as the dropdown.
- **Inputs with adornments** (currency prefix, unit suffix, a clear button): the adornment's vertical center matches the input *text's* vertical center, not the input box's border-to-border center, which differs once padding is asymmetric.

The underlying rule for all of these: **an icon or control placed "near" text without a stated relationship to that text's padding, baseline, or cap-height is a guess, and guesses are what read as slightly wrong.** State the relationship, then implement the number.

---

## Motion & Performance Discipline

Animation is the most common place AI-built frontends feel cheap: technically present, physically wrong. These rules are hard gates, not taste.

### What may animate

- **Animate only `transform` and `opacity`.** These run on the compositor thread and stay smooth even while JavaScript is busy. Animating `width`, `height`, `top`, `left`, `margin`, or `padding` forces layout recalculation every frame — that is where "sticky" stuttering comes from, and it gets worse precisely on the low-end devices and busy main threads where users will notice.
- When something genuinely must move between layout positions or sizes, use the **FLIP technique** — measure First and Last positions, apply the Inverted transform, Play it back as a pure transform animation. Framer Motion's `layout` prop and GSAP's Flip plugin do this correctly out of the box; hand-rolled layout animation usually doesn't.
- Use `will-change` sparingly and only on elements about to animate — blanket `will-change` consumes memory and can make things slower.
- **Timing**: micro-interactions (hover, press, toggle) 100–200ms; element entrances 200–400ms; anything over 500ms had better be a deliberate moment. Ease-out for things appearing, ease-in-out for things moving. Define two or three duration/easing tokens in DESIGN.md and reuse them everywhere — uniform motion is what makes a UI feel coherent.
- **Respect `prefers-reduced-motion`** — collapse non-essential animation to simple fades or nothing. This is an accessibility requirement, not a preference.

### Easing and the physics of smoothness

Two animations can both hit 60fps and still feel completely different — one smooth, one "stucky" (the exact word for motion that starts or stops abruptly, or moves at a constant rate; it reads as mechanical no matter how technically performant it is). The difference is almost entirely in the easing curve and duration, not the frame rate:

- **Never use `linear` for anything perceived as an object moving** — linear is correct only for continuous, mechanical motion like a progress bar or a spinner. Everything else needs acceleration: things starting from rest ease in, things coming to rest ease out, things that move and then stop (most UI motion) ease in-out.
- **Prefer spring/physics-based easing for anything triggered by direct interaction** — a dragged panel settling, a toggle, a card responding to a click. Hand-authored `cubic-bezier` curves can look close to physical but never quite match how a real object with mass and friction decelerates; a spring model (mass/stiffness/damping — Framer Motion's `type: "spring"`, GSAP's `elastic`/`back` eases) computes the actual physics and is why interactions built on it feel *right* without hand-tuning numbers by trial and error.
- **Prefer authored cubic-bezier curves for entrances/exits and anything on a fixed timeline** — page intro sequences, scroll-triggered reveals — where a predictable, repeatable duration matters more than physical responsiveness. Springs don't have a fixed duration by nature, which is wrong for something that has to land in sync with a timeline of other elements.
- **Define the actual curves as named tokens**, not just the words, in `docs/DESIGN.md` — e.g. `ease-out-standard: cubic-bezier(0.16, 1, 0.3, 1)` for entrances, `ease-in-out-standard: cubic-bezier(0.65, 0, 0.35, 1)` for moves, plus one spring config (e.g. `stiffness: 300, damping: 30, mass: 1`) for interactive elements — three to four tokens total, reused everywhere. "Ease-out" as a word is not a token; the curve is.

### Parent/child choreography

A composition of many elements — a card grid, a nav with children, a form with fields — reads as smooth only when the parent and its children move as one coordinated system, not as independent elements that happen to animate at the same time:

- **The parent's timeline conducts the children's; it doesn't just contain them.** Build one timeline per composed entrance (a GSAP timeline, or a Framer Motion parent using `staggerChildren`/`variants`), not N independent animations started together by coincidence of mount order. Independently-triggered animations drift out of sync under real-world jank (a slow paint on one element) in a way a single owned timeline does not.
- **Stagger by proximity and hierarchy, not just index.** Elements that are visually grouped (a card's title and its body) enter together or nearly together; elements that are visually separate (card 1 and card 4 in a grid) get the visible stagger gap. A flat, uniform stagger across everything on the page regardless of grouping is what makes an entrance feel like a checklist being ticked off rather than a composition arriving.
- **A child never starts before its parent's own geometry has resolved.** If a parent container itself animates size or position (rare — most parents should only fade/translate, per the transform/opacity rule), children wait for that to complete, or use FLIP, so they don't visibly fight the parent's still-settling layout.
- **Exits reverse the hierarchy, not just the animation.** Children typically exit first (fast, subtle) before or as the parent container that held them exits — a parent that vanishes while its children are still mid-exit looks orphaned for one frame, exactly the kind of glitch users can't name but do notice.
- **One coordinated stagger per composition, not nested staggers multiplying each other.** A card grid staggering in, where each card *also* staggers its own internal elements on entrance, easily produces a multi-second cascade that feels slow rather than smooth. Cap total composition entrance time (roughly 400–700ms even for a busy grid) and choose *one* level of the hierarchy to carry the visible stagger — usually the outermost meaningful grouping.

### The first-visit intro (extends Phase 1's Design Discovery)

An intro sequence — a brief, deliberate animated moment before or as the main UI settles — is a legitimate design decision for a project whose three-second idea (from the design-concept question in Design Discovery) benefits from one, and a mistake when bolted onto every project by reflex. Decide it explicitly, in discovery, per project:

- **The pattern is derived from the design concept and the dominant content shape, not chosen from a generic "website intro" effects library.** A data-dense internal tool used fifty times a day should almost never get a decorative intro — the cost is paid by the same user every single time, and speed *is* the design statement for that product. A portfolio, a landing page, or a brand-forward consumer product visited occasionally can afford a few hundred milliseconds of a considered opening, because the cost is paid once and the payoff is the first impression. Decide which situation this project is before adding one.
- **The intro composes the same content the page needs anyway — it doesn't hide behind an unrelated splash.** A mask/clip-path reveal of the actual hero content, a headline that builds itself (a split-text reveal, one word or line at a time), or a structural build (frame lines draw in, then real content populates them) all use real content as the animation's material. A generic logo spinner or unrelated loading animation the user sits through before the real page appears is dead time, not design — exactly the "bolted-on" feeling to avoid.
- **It runs once per session, not once per navigation.** Store that the intro has played (session storage or equivalent) and never replay it on route changes or re-visits within the same session — an intro that repeats every time reads as a nag, not a moment.
- **It has a real skip path.** Interacting with the page (a click, a keypress, a scroll) short-circuits straight to the settled state — an intro that traps input for its full duration is a usability bug wearing a design costume.
- **It respects `prefers-reduced-motion`** by collapsing straight to the settled state, and it never blocks the page from being usable if its assets load slowly — build it so a slow-loading font or image degrades to *no intro*, not a broken, half-finished one.
- **Total duration is short and stated as a token** in `docs/DESIGN.md` — a reasonable ceiling is ~600–1200ms for the whole sequence before the page is fully interactive. An intro users have to sit through more than once becomes the thing they resent about the product.

Record the decision — including "this project deliberately has no intro, because X" — in `docs/DESIGN.md`; like the visual direction itself, this is a decision made once and followed, not re-litigated per session.

### Scroll-triggered motion, done sparingly

The Anti-Patterns list below still holds — scroll animation on every element is noise. But the fix for noise is *restraint*, not *absence*: a small number of deliberate scroll moments, chosen because the content shape earns them, is a legitimate and often-underused tool:

- **Pick moments where scroll position is meaningfully tied to content, not decoration for its own sake.** A step-by-step process, a before/after comparison, a data visualization that builds as its section enters view, or a long narrative page with a small number of section transitions are content shapes that earn scroll-linked motion. A marketing page where every third paragraph fades up on scroll for no content-driven reason is the anti-pattern, not a milder version of the technique.
- **Prefer enter-once reveals** (GSAP's ScrollTrigger with `once: true`, or an Intersection Observer one-shot) **over continuously scrubbed effects for most content** — a section that fades/rises into place as it enters the viewport, once, is far cheaper and far less likely to feel gimmicky than a value tied continuously to scroll position.
- **Reserve scrubbed/pinned effects** (opacity or transform tied continuously to scroll offset via `scrub: true`, or a section held in place with `pin: true` while an internal sequence plays) **for the one or two moments per page that are actually the point** — a hero that's meant to be a moment, a data story's key chart. Not the default for every section on the page.
- **Depth via parallax is a light touch, not a gimmick**: a background layer moving at 0.5–0.8× the foreground's scroll speed reads as depth; multiple layers at wildly different speeds, or parallax applied to text (which becomes hard to read while moving), reads as a mistake.
- **Scroll-triggered animation still obeys the transform/opacity rule** — a scrubbed effect driving `width` or `top` on scroll is the single worst place to break that rule, because it runs on every scroll frame, continuously, on the exact interaction users notice jank in fastest.
- **Verify with the same throttled-CPU pass as everything else.** Scroll listeners and scrub calculations are exactly the kind of work that's invisible on a dev machine and stutters on a mid-range phone.

### Library choice, stated neutrally

Plain CSS transitions handle most micro-interactions with zero bundle cost — prefer them for hover/focus/press states. Reach for a library when you need what it's actually for: **Framer Motion** (React) for declarative component animation, presence (mount/unmount) animation, `staggerChildren`/`variants` parent-child choreography, and FLIP layout animation; **GSAP** for complex timelines, scroll-driven sequences (the ScrollTrigger plugin), FLIP layout (the Flip plugin), and framework-independent control. Both are legitimate defaults; pick per the project's framework and the animation's complexity, record the choice as a Decision Log entry, and don't ship both without a stated reason. Build the easing/spring tokens above as the actual constants those APIs are called with — not re-authored per component. Whichever is chosen, **lazy-load animation code** with the routes that use it — animation libraries must never be in the critical path of first paint. And per Phase 8's dependency rules, an animation library is a dependency like any other: justify it, check maintenance, isolate usage behind the project's own motion tokens/components so it can be swapped.

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
- [ ] Composed entrances (card grids, staggered lists) are driven by one owned timeline/stagger, not independently-triggered animations that visibly drift under real load
- [ ] Small controls (dropdowns, buttons, chips, tooltips) checked against Component Micro-Design's anatomy — icon-to-text relationship, all states present, viewport-collision handling verified, not just the happy-path position
- [ ] If this slice includes a first-visit intro: verified it runs once per session, has a working skip path, and collapses correctly under reduced motion
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
- **A trailing icon or chevron placed by guesswork** — flush against the edge, floating with no stated relationship to the text baseline, or unsynced with the panel it controls opening.
- **Uniform fade-everything-in** — every element on a screen using the identical entrance regardless of visual hierarchy or grouping; a composition should read as arriving, not as a checklist being ticked off.
- **An intro sequence that replays on every navigation**, traps input, or hides real content behind an unrelated splash animation.
- **Nested staggers that multiply** into a multi-second cascade instead of one owned, composition-level stagger.
- **Springs used where a fixed, synced duration was needed** (elements landing out of sync with a scroll-locked or timeline-driven sequence) **or authored curves used where physical responsiveness was needed** (a dragged element that doesn't feel connected to the cursor).
- **Spinner-only loading** and **animations that gate content** (users wait for the fade-in to finish before they can act).
- **Confirmation dialogs as a safety blanket** on reversible actions, instead of undo.
- **Per-slice style invention** — a new gray, a new gap value, a new duration "just this once." That's source-of-truth drift; the tokens live in DESIGN.md.
- **Designing only the happy, populated, fast-network state.** Real first impressions are empty states on slow connections.
