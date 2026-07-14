# Book 4 — Frontend Experience & Design

Part of a four-book set. Start at `SKILL.md` if you haven't already. This book covers how the product should *look, move, and feel to operate* — design discovery, workflow-first screen design, layout systems (including bento grids), motion discipline, render correctness (no flicker, no flash, no layout disruption), and low-bandwidth resilience. For the big-picture methodology see `methodology.md`; for per-slice implementation discipline see `coding-standards.md`; for document templates see `templates.md`.

Load this book whenever a slice touches UI: designing a screen, choosing a layout system, adding animation, or verifying a UI slice. Pure backend/schema slices don't need it — that's the same context-management principle the rest of the workflow runs on.

This book plugs into the existing phase numbering rather than adding new phases: design discovery extends Phase 1, the Design System document is produced in Phase 3, workflow-first screen design informs Phases 2–6, and the UI verification checklist extends Phase 9.

---

## Purpose

AI-generated frontends have a recognizable house style: a centered container, a hero section, three feature cards in a row, uniform rounded-corner cards for everything, default component-library styling, and animations bolted on wherever they fit. It's not ugly — it's *generic*, and every AI-assisted product that skips design discovery converges on it. This book exists to prevent four specific failure modes:

- **Template convergence** — the UI looks like every other AI-built app because no deliberate design direction was ever chosen. The fix is a mandatory design-discovery step where the direction is *asked for and recorded* — and derived from the project's own subject matter, never defaulted or borrowed wholesale from a reference.
- **Workflow-hostile screens** — the UI mirrors the database (one entity, one page, one form) instead of the user's task, so completing one real-world job means bouncing across five pages. The fix is designing screens around tasks, with patterns like quick-add and bulk-add built in from the start.
- **Uncalculated geometry** — containers, gaps, and small recurring components (dropdowns, buttons, chips, icon-and-text pairs) placed by feel instead of arithmetic, so nothing quite lines up and small elements read as "off" even when no single rule is technically broken. The fix is treating every container's internal spacing and every small component's anatomy as a calculation, not a guess (see Geometry & Spatial Composition and Component Micro-Design below).
- **Janky, undirected motion** — animations that look fine on the developer's machine but stutter on mid-range phones and slow networks because they animate layout properties or depend on assets that haven't loaded yet; entrances that fire independently instead of as one composed sequence; a scroll effect or page intro bolted onto every project by reflex instead of chosen because the content earns it. The fix is a hard rule set about what may animate, an explicit choreography model for parent/child motion, and verification on throttled networks as a gate.
- **Broken-at-the-seams rendering** — each animation correct in isolation, but the page flickers and glitches where they meet reality: content flashes visible-then-hidden at first paint before the animation code takes over, an entrance from off-screen spawns a horizontal scrollbar and shifts the whole page, a hover state reflows its neighbors, a reveal never fires after a mid-page reload, an interrupted transition snaps or double-plays, text turns blurry or unreadable while it moves. These are correctness bugs with reproduction steps, not polish gaps — the fix is the Render & Motion Correctness rules and the mandatory Render QA pass below, run in a real browser before any UI slice is Done.

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

### The uniqueness guarantee: specificity, not randomness

The bar to hit: **two different people building the same *topic* with this same workflow should still end up with visibly different designs** — because their *projects* differ even when their topics don't. Uniqueness is not achieved by randomizing choices (random is as generic as default); it's achieved by making every major design decision a function of facts that are specific to *this* project — and no two projects share all their facts. The procedure:

1. **Mine the project's specific facts before designing anything.** Not "a bakery site" — *this* bakery: its name and what the name evokes, its city and that city's visual culture, its actual products (sourdough's blistered crust vs. patisserie's precision), its founding story, its customers, its tone of voice, its photography (rustic flour-dusted vs. clean studio), its price point. Ten minutes of asking produces a fact sheet no other project duplicates. For products without a business behind them yet, the facts come from the content itself: the data's shape, the domain's artifacts, the user's rituals.
2. **Every major design decision cites the fact that produced it.** The palette's base hue comes from *something* — the product's material, the city's light, the brand's history — not from "warm feels right for food." The type pairing, the layout system, the motion character, the signature interaction: each one traceable, in one line in `docs/DESIGN.md`, to a project fact. A decision that cites no fact is a default wearing a costume, and defaults converge.
3. **Derive a signature geometry motif from the project's own artifacts.** One shape, drawn from something real in the project — the diagonal scoring on a baguette becomes the project's chamfer angle; a vinyl record's concentric grooves become the loading spinner and the section-divider treatment; a legal firm's document folio becomes the card's folded corner. Carried consistently through radii, dividers, clip shapes, and icons (the shape-vocabulary sections above), one derived motif does more for uniqueness than fifty styling tweaks, because no other project has *this* artifact.
4. **Run the sameness audit before DESIGN.md is signed off.** Three questions, answered honestly: Would this palette + type + layout read at home in another AI-generated site? (If yes, find which decisions cite no fact, and re-derive them.) Is there any *one* thing here a visitor would describe to a friend? (If nothing is describable, there's no signature — go back to step 3.) If the topic were handed to a different team with this same workflow, would they plausibly land here? (If yes, the facts weren't specific enough — mine deeper.) Record the audit's answers in DESIGN.md; an unaudited design direction is not Ready.

This is the design-side twin of the anti-hallucination rule: "I derived it from this project's actual facts" and "I produced something plausible for this kind of project" are different confidence levels, and only the first is allowed to become the design.

### Extracting a reference site, not eyeballing it

When the user names a reference site, inspect it in a real browser (with browser tooling such as Claude in Chrome, or DevTools by hand) and extract **concrete measurements** — the difference between "inspired by" and "vaguely reminiscent of" is numbers:

- **Layout grid**: container max-width, number of columns, gutter width, outer margins at each breakpoint. Measure them; don't estimate from a screenshot.
- **Type scale**: the actual font sizes, weights, and line-heights in use for display, heading, body, and caption text — and the ratio between steps (see Typography System, below, for how to turn this into a stated ratio rather than a copied list of numbers).
- **Spacing rhythm**: the base spacing unit (commonly 4 or 8px) and the recurring gaps between sections, cards, and controls.
- **Shape language**: corner radii (uniform or mixed), border and divider treatment, shadow depth, any clipping/masking of images.
- **Color roles and distribution**: background layers, surface, text hierarchy, one or two accents — captured as roles, not just a list of hex codes — plus a rough read of how much of the page each occupies (see Color System, below, for turning that into a stated ratio and harmony rule).
- **Motion character**: what animates on the reference site (hover, scroll, page transitions), how fast, and how much. Match the *restraint* too — most admired sites animate far less than people remember.

Write the extracted values into `docs/DESIGN.md` as the project's tokens. This is the anti-hallucination discipline of Phase 13 applied to design: "I looked at the actual site and measured it" versus "I know roughly what that kind of site looks like" are different confidence levels, and only the first one is allowed to become a token.

If page-level layout math is involved (it usually is), do it explicitly: pick the container width, choose the column count and gutter, and compute what's left. For example, a 1,200px container with 12 columns and 24px gutters leaves (1200 − 11×24) ⁄ 12 = 78px per column — so a tile spanning 4 columns is 4×78 + 3×24 = 384px wide. Write the arithmetic into DESIGN.md once, and every session builds tiles that line up instead of re-guessing widths.

### Live inspiration research — browse the showcases like a human, during design

Reference extraction (above) fires when the user names a site. This procedure fires when they haven't — or when the design concept is set but its *execution* needs current, real-world craft to draw on. If browser tooling is available (Claude in Chrome or equivalent), do this during design discovery, before DESIGN.md is signed off; it's the design-side version of "check the installed version's docs before relying on an API":

1. **Derive the search brief from the design concept, not the topic.** The concept ("precision instrument for one expert user," "playful and forgiving," "editorial authority") plus the technique needs the project has already surfaced (a preloader? scroll-driven storytelling? a dense data console?) become the categories and tags to browse — on Awwwards (`awwwards.com/websites/<tag>/` — e.g. `loading-animations`, `scroll`, or an industry tag), and on the other showcases worth knowing: godly.website, land-book.com, siteinspire.com, minimal.gallery. Searching "best <topic> websites" is the lazy version that leads back to templates; searching by *concept and technique* finds transferable craft.
2. **Shortlist from thumbnails, then visit the actual live sites.** The showcase page is a menu, not the meal — thumbnails show layouts, never motion. Click through to 3–5 live sites. Practical craft for observing motion through screenshots: capture *multiple frames* (screenshot, wait a beat, screenshot again) to catch preloaders mid-sequence, entrance staggers, and scroll states; scroll and re-capture; expect and dismiss cookie banners and sign-in popups before judging the design under them.
3. **Extract techniques, not looks — with the same measurement discipline as reference extraction.** From each site, name the *transferable mechanism*: "preloader shares the hero's exact background color so resolve is a continuation, not a cut," "stagger carries only the outermost grouping," "nav hugs the true viewport edge at ~40px." Write it as a named technique with numbers, per the extraction checklist above — never as "make it look like this site."
4. **Synthesize across sources; never let one site dominate.** One site's motion restraint + another's preloader continuity + a third's chrome margins is inspiration; one site's everything is a clone with extra steps. Rule of thumb: no single source contributes more than one or two named techniques, and at least two sources contribute.
5. **Justify each adopted technique against a project fact** — the same rule as the uniqueness guarantee above. A technique enters DESIGN.md as "adopted because <project fact>," or it doesn't enter. The showcase's job is to expand the vocabulary of *how*; the project's own facts still decide *what*.
6. **Record the research in DESIGN.md**: sources visited (URLs, date), the technique each contributed, and the fact-based justification. The sameness audit still runs afterward — inspiration research that ends in a design indistinguishable from its sources failed the audit, no matter how good the sources were.

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
- **Clipped/shaped ("cliparted") bento**: the modern editorial variant — images or accent tiles clipped into non-rectangular shapes, oversized asymmetric corner radii (one corner 48px, the others 12px), or shapes that visually bridge adjacent tiles. Use it on *accent* tiles, not every tile — the contrast against calm rectangular neighbors is what makes it look designed. The full construction technique — cards whose outline is carved to receive the elements docked on them, not decoration floating on top — is its own subsection below.
- **Animated bento**: entrance as a short stagger (tiles fade/rise 20–30px, ~50–80ms apart, transform + opacity only); hover as a subtle lift or scale (1.01–1.03) on the tile; expanding/reordering tiles via FLIP-based layout animation (Framer Motion's `layout` prop, GSAP's Flip plugin) so tiles glide between positions instead of snapping. Everything in the motion section below applies with extra force here, because a bento animates many elements at once.

### Cut, docked, and shaped cards — calculated, not floating overlays

The clipped/shaped variant above has a more advanced form worth learning properly, because it's a small set of repeatable geometric moves, not a one-off illustration: instead of a plain rounded-rect card with a badge floating on top via absolute positioning and a drop shadow, **the card's own outline is carved to receive the element that sits on it** — the badge looks *docked into* the card, not stuck on top of it. This is exactly what separates a shaped card that reads as "designed as one object" from one that reads as "a rounded rectangle with stuff placed over it."

- **The docking notch.** Where a small control (an icon pair, a pill button, a label chip) needs to sit on a card's edge, cut a shallow concave scallop into that edge, sized to the control's own bounding box plus its clearance, built from two mirrored S-curve (cubic bézier) arcs meeting at the control's width — not a rectangular notch (reads as a slot) and not a single round bite (reads as an accident). The notch's width is the docked element's actual width plus one spacing-scale step of clearance on each side; the notch's depth is roughly the docked element's height. The two curves mirror each other exactly — that symmetry is what makes it read as "made for this," rather than "cut at a random spot."
- **The docking blister.** Where an element needs to sit on a card's *side* edge (a vertical rotated label, a small arrow control), bulge or recess the edge across the element's height with the same mirrored-S-curve technique, so the element nests into a receiving bump instead of overlapping the edge with a shadow. Done well, it reads as if the card's own material was formed around the control.
- **The corner recess.** The same move at a corner instead of an edge: an inward scoop sized to receive a circular badge (a logo mark, an icon button) so the badge's circle appears to sit inside a socket carved for it, tangent to both surrounding edges. Compute the scoop's radius from the badge's radius plus its clearance — the same arithmetic as the docking notch.
- **The chamfered corner.** A single corner cut at a diagonal (not rounded) reads as a folded or sliced corner — a strong signal of intentional shape design rather than a border-radius default. Size the chamfer proportionally to the card (a common ratio: chamfer run ≈ one grid gutter to 1.5 gutters, or roughly 10–15% of the card's shorter side) so it scales sensibly across breakpoints instead of looking oversized on a small card or invisible on a large one. Combine at most one chamfer with at most one notch/blister per card — stacking several cut features on one shape reads as busy, not designed.

**Building it, and keeping it responsive:**

- **Prefer an inline SVG shape layer behind plain DOM content over a raw CSS `clip-path`.** Author the card's cut outline as an SVG `<path>` in a representative `viewBox` (designed once at a representative size, e.g. 455×544 units), with `preserveAspectRatio="none"` and `width: 100%; height: 100%`, positioned behind an ordinarily-padded content layer. The browser's native non-uniform scaling stretches the notches and chamfer along with the box on every resize, with no JS and no recomputation required — which is what makes this genuinely flexible rather than a fixed-pixel graphic. The real content (text, images, controls) lives in normal DOM on top, positioned with the padding rules from Geometry & Spatial Composition, above; the SVG is purely the visual boundary.
- **`clip-path: path(...)` is the CSS-only alternative**, but its coordinates are absolute pixels evaluated post-layout — it does not stretch with the box on its own. Reach for it only when the shape's aspect ratio is effectively fixed (a small fixed-size avatar frame, an icon tile), and recompute it via a resize observer or container query when it must flex; for anything that needs to breathe across breakpoints, the inline-SVG-behind-content technique above is simpler and cheaper.
- **Keep notch/chamfer sizes as ratios of the card, not copy-pasted absolute numbers**, when the same cut shape is reused at meaningfully different sizes (a hero-sized panel vs. a grid tile) — a scallop calculated for a tall hero card looks subtly wrong stretched flat onto a short grid tile. Re-derive the construction at the new size rather than reusing the exact path.
- **The docked element's own position is computed from the same numbers that cut the notch.** If the notch is centered at 60% of the card's width and sized to the control's actual width, the control is positioned at that same coordinate, not eyeballed separately — the cut and the content that fills it come from one calculation, or they drift apart the first time either one's size changes.

### Decorative shape vocabulary: scatter patterns and organic accents

Two more repeatable moves round out this style's decorative layer, used sparingly (accent tiles only, per the anti-patterns below):

- **Confetti/scatter clusters** (small repeated shapes — diamonds, shields, dots — behind a headline or in a corner): build them on an implicit grid (a base unit and row/column spacing from the spacing scale), then break the grid's uniformity on purpose — offset alternate rows, vary each shape's opacity by a falloff function across the cluster (denser and more opaque near the anchor point, sparser and fainter toward the edge). A perfectly uniform grid of identical, identically-opaque shapes reads as a stamped repeat, not a considered decoration.
- **Organic accent blobs** (an asymmetric rounded shape behind an illustration or photo, for depth): construct from a small number of cubic-bézier lobes of clearly *unequal* size and curvature. A shape with any two lobes that mirror each other reads as generic clip-art; genuine asymmetry is what makes it read as designed rather than pulled from a "blob generator" default. Use one, maybe two, per screen, always behind content that needs the depth — never as standalone decoration with nothing to support.
- **A hand-drawn arrow/chevron glyph is part of the shape system, not an icon-font default** — a short shaft plus arrowhead at a stated weight, reused consistently for every "go to" affordance (a headline's call-to-action, a card's expand control) so it reads as belonging to this design, not borrowed from a generic icon set.

### Learning a supplied asset kit, without copying it verbatim

When a project supplies its own shape/asset library (a Figma export, an SVG folder of cards/decorations/icons), the job is to **learn its geometric grammar** — which notch widths pair with which control sizes, the chamfer ratio actually used, the scatter pattern's grid unit and opacity falloff, the blob lobes' proportions — and re-derive matching shapes for *this* project's own components at their own sizes, the same way a reference site gets measured rather than eyeballed (Design Discovery, above). Copying the supplied files unchanged into an unrelated project is template convergence with extra steps; extracting the technique and re-authoring it for the project's actual content, sizes, and breakpoints is the point.

### The alternatives, so the choice is a real one

Offer these alongside bento in discovery, each in one honest line: **editorial/typography-led** (large type, generous whitespace, few images — content-heavy products, brand-forward marketing); **dense console** (compact tables, minimal chrome, keyboard-first — heavy-use internal tools); **minimal utilitarian** (few elements, high contrast, near-zero decoration — single-purpose tools); **reference-derived** (extracted from a site the user loves — often the best answer when it exists).

---

## Cinematic Direction — the page as a directed scene, not a stack of boxes

The strongest rejection this workflow makes of the AI house style is compositional: **the default AI page is a vertical stack of uniform card containers, and a card-stack is inert** — it reads as a filing cabinet, not a living thing. A directed page works the way a film does: it tells one story, its elements share a world, and the viewer is *led* through it. This section is the director's half of Book 4; everything else (geometry, color, type, motion) is the crew.

### Storyboard before layout

Before any section is laid out, write the page's storyboard — three lines in DESIGN.md, like a film treatment:

- **Acts.** A page has an arc: the *hook* (the opening moment — the intro/hero, the one visual idea from the design concept landing in three seconds), the *development* (each section advances the story — capability, proof, depth — in a deliberate order, not "features, then testimonials, because that's the template order"), and the *resolution* (the call to action as the story's payoff, not a button bolted to the bottom). Name each section's job in the arc; a section with no narrative job gets cut, exactly like a scene that doesn't advance a film.
- **The scroll is the camera.** Decide the page's camera grammar once: scrolling as a *dolly* (continuous movement through one connected space — elements enter, pass, and exit like scenery), as *held shots with cuts* (pinned scenes that play out, then release — GSAP `pin`), or as *dissolves* (sections cross-fading through each other). Pick one grammar per page and let every section obey it — a page that dollies, then cuts, then parallaxes at random reads as edited by committee.
- **Blocking.** For each scene, decide where elements enter from and why — type rises from the fold line it will sit on; an image slides in from the side its content faces; nothing "fades in from nowhere, wherever it happens to sit." Entrances follow the reading path (the eye-line), so motion *leads* the viewer instead of decorating around them.

### The alive canvas — composition instead of containment

The rule that kills the card-stack: **an element gets a visible container only when containment means something** — a genuinely enumerable set of homogeneous items (products, plans, table rows), or an interactive surface that needs a boundary (a form, a dropdown panel). Everything else sits *in the scene*:

- **Compose in depth layers** — background (color fields, oversized type, texture, a 3D scene), midground (the primary content), foreground (small floating accents, the docked controls from the shaped-card section). Elements overlap across layers — a headline's descender tucks behind an image, a product breaks out of its color field — because overlap is what makes a composition read as one world instead of adjacent rectangles.
- **Type is scenery, not just labels.** Display type at architectural scale — cropped by the viewport edge, layered behind or in front of imagery, used as the section's structure itself (the noth.in reference earlier in this book is exactly this) — is the cheapest cinematic instrument there is: zero bytes of media, pure design.
- **Section boundaries are transitions, not gaps.** Two adjacent sections share an edge the way two film scenes share a cut: a color field that ends on a diagonal, a shape from the vocabulary bridging both, an element that starts in one section and finishes in the next. A page whose every section is a full-width stripe separated by 96px of nothing is paginated, not directed.
- **Whitespace is negative space in the composition** — deliberate, asymmetric, load-bearing — not the uniform padding a card template distributes evenly around everything.
- The bento grid (above) remains legitimate *within* this: a bento is a composed arrangement — the anti-pattern is not the grid, it's the reflexive wrapping of every piece of prose, every image, and every heading in its own bordered, shadowed rounded box.

### Presence of motion is the house default; its pattern is still derived

This workflow's standing position, recorded here once so sessions stop re-deciding it: **every public-facing project opens with a directed intro moment and navigates through designed transitions.** The craft rules from the Motion section still govern (real-content material, once per session, skippable, reduced-motion collapse, preloader gating on real readiness) and the *pattern* is still derived from the project's facts per the uniqueness guarantee — what's no longer optional is the presence of an opening moment and a transition grammar. The documented exception path: a daily-grind internal tool may opt down to minimal motion, recorded in DESIGN.md with its reason — silence is a directing choice too, but it has to be *chosen*, not defaulted into.

### 3D as a cinematic instrument — three.js, and the Blender pipeline

When the design concept earns real depth — a product worth orbiting, a spatial metaphor, a hero that is the story — 3D is the instrument, and it follows the same discipline as every other dependency:

- **Three.js (or react-three-fiber in React projects) is the delivery layer**; scroll-driven camera moves through a scene obey the scrubbed-moment rules from the Motion section — this is the one or two moments per page that are the point, not ambient decoration on every section.
- **Author custom assets instead of stock.** If the developer's machine has a Blender MCP connection configured (check for a `blender` MCP server; the addon requires Blender running with its socket server active), model the asset there — a custom object derived from the project's own artifacts (the signature-motif rule extended into 3D: *this* bakery's bread, *this* product's silhouette) is what separates a cinematic scene from a floating stock torus. Keep authored geometry deliberately low-poly-elegant; style comes from materials and lighting, not polygon count.
- **The delivery discipline is non-negotiable**: export GLB with Draco/meshopt compression; lazy-load the scene code and assets entirely outside the critical path (the page must be readable before the scene arrives — reserve its space, fade it in); cap device-pixel-ratio; pause the render loop when the scene is offscreen or the tab is hidden; `prefers-reduced-motion` and low-end devices get a rendered poster image, not a stuttering canvas. A 3D scene enters `PERFORMANCE_BASELINE.md` with its own bundle and frame-time budget, and the throttled-device verification pass applies to it doubly.

### Section-by-section design checkpoints — direct with the user, not for them

Cinematic ambition raises the cost of guessing wrong, so the feedback loop tightens: **each major UI surface gets its own checkpoint with the user as it's first built** — the nav, the hero/intro, each subsequent scene. The loop:

1. Build the section to a genuinely reviewable state (real content, real motion — not a gray-box sketch that forces the user to imagine the design).
2. Show it — screenshot at the key viewport, or the running app — and ask two focused questions: *is this direction right?* and *what would you change?* Not "do you like it?", which invites a mood, not a decision.
3. Adapt to the answer, and **carry the feedback forward as a standing decision**: if the user pulled the nav toward tighter spacing or calmer motion, that preference applies to every subsequent section without being re-asked — record it in the DESIGN.md checkpoint log, exactly like Phase 12's standing decisions.
4. An approved section is a settled scene: later work matches it, and revisiting it needs the same explicit sign-off as any source-of-truth change.

This replaces the failure mode of building the whole page and asking once at the end — by which point the accumulated guesses are too expensive to unwind. A UI slice covering a major surface is not **Done** (Phase 9) until its checkpoint has run.

---

## Color System

The AI-default palette is recognizable on sight: a blue or indigo primary, a white/light-gray surface, and a purple gradient hero — chosen because it's inoffensive, not because anything about the project asked for it. A color system prevents that the same way the layout system does: **a distribution rule and a harmony rule are chosen deliberately, from the design concept, and stated as a ratio and a role table** — not picked hex-by-hex until it "looks nice."

### The distribution rule — how much of each color, stated as a ratio

Pick one, explicitly, and record it as roughly-measurable percentages in `docs/DESIGN.md` — the point isn't which rule wins, it's that *a* rule is chosen instead of every component deciding its own amount of color:

- **60-30-10** (dominant / secondary / accent) — the safest default for most content-heavy or dashboard products: 60% a neutral dominant (background/surface), 30% a supporting tone (secondary surface, muted text, borders), 10% one accent reserved for what must draw the eye (primary actions, links, key data points). The *scarcity* of the 10% is what makes it work — spend it only on things that need to win the user's attention, not on every icon and border.
- **90-10 (or more extreme, 95-5)** — the right call when a minimal/utilitarian or restrained-editorial concept makes near-monochrome the brand statement; the accent should feel almost startling on arrival, used at one or two moments per screen, never spread thin.
- **40-30-20-10** (dominant / secondary / tertiary / accent) — for content with a genuine third visual tier before the true accent (a dense dashboard with background, card-surface, and a distinguishable "highlighted" tier).
- **70-20-10, brand-as-environment** — when the concept calls for the brand color as the field itself (a hero panel that *is* brand-orange, not just accented with it), backed by a calm near-neutral secondary for legibility surfaces. Only earns this when the concept explicitly wants that confidence — it's the loudest of these rules and wrong by default.

Tie the choice to the design concept question from Design Discovery: a domain's own visual vocabulary usually implies the rule (a trading tool's restraint implies 90-10 or a tight 60-30-10 with a single alert accent; a playful consumer brand can sustain 70-20-10 with itself as the field).

### The harmony rule — how the hues relate to each other

Construct the palette's hues from one named harmony, not by eye:

- **Monochromatic** (one hue, varying lightness/saturation) — the safest professional or dense-console choice; sophistication comes entirely from value/saturation steps, with one true accent hue breaking the family for actionable elements.
- **Analogous** (adjacent hues) — calm and cohesive; a good base for editorial, content-heavy, or lifestyle concepts; needs one deliberately different accent (often near the complement of the family) or it reads as flat.
- **Complementary** (opposite hues) — high contrast and energy; good for a single unmistakable CTA against a calm base; risky at large, equally-saturated areas (it vibrates) — use one side of the pair as the scarce accent only, never both at equal weight.
- **Split-complementary** (a base hue plus the two hues adjacent to its complement) — most of complementary's contrast with less visual tension; a good middle ground when the concept wants energy without a clash.
- **Triadic** (three hues evenly spaced) — vibrant and playful, but needs a firm distribution rule (60-30-10 or stricter) or it reads as a rainbow instead of a designed triad: one hue dominant, the other two used sparingly and *unevenly* — never as equal thirds.

### Roles, not hex codes

Define color the same way the reference-site extraction does: as a **role table**, one value per role per theme, not a loose list of hex codes reused ad hoc — Background/Base, Surface (cards/panels), Border/Divider, Text-primary, Text-secondary, Text-disabled, Accent-primary (the scarce color from the distribution rule), Accent-secondary (rare — a second-tier action), and Semantic (success/warning/danger/info). **Semantic colors sit outside the brand harmony** — chosen for universal recognizability (the green/amber/red family) regardless of what the harmony rule picked, because status legibility overrides brand aesthetics; never let "but it clashes with our purple" reassign what red means.

### Contrast is arithmetic, not a vibe check

- Every text/background pairing hits WCAG AA (4.5:1 body text, 3:1 large text and UI components) — check the actual computed pair the product uses, not the brand colors in isolation. This is already part of UI Slice Verification, below; the color system is where the pairs that need checking get decided.
- Never convey status by color alone — pair every semantic color with an icon or label for colorblind users.
- **Check the accent color's contrast first and specifically** — it usually sits on a button behind white or near-white text, which is exactly where a color chosen for hue alone (not for how it performs as a background) fails contrast.

### Dark mode is a derivation, not an inversion

Don't flip lightness values and call it done — recompute each role deliberately: dark surfaces usually want to keep a *trace* of the brand hue rather than going fully neutral (fully desaturated dark UI reads as muddy); the text hierarchy keeps its *relative* contrast steps, not its absolute values; and the accent color often needs its own lightness/saturation retuned for a dark background — a saturated accent that pops on white can vibrate or lose contrast on near-black. Record the dark-theme role table as its own set of values, not "same hex, dark background."

Record in `docs/DESIGN.md`: the chosen distribution ratio, the harmony type and base hue(s), the full role table for light (and dark, if supported) themes, and which pairings were contrast-checked.

---

## Typography System

The same content-decides-it principle applies to type: the typeface, the scale, and the pairing are derived from what the product actually contains and how it's read, not picked from a "good font pairings" list.

### Choosing the typeface(s) from the content

- **What's the dominant reading mode?** Scanning dense data (a console, a finance tool) wants strong numeral legibility and tabular figures (clear 1/l/I and 0/O distinction, `font-variant-numeric: tabular-nums` for anything in a column). Long-form reading (editorial, docs, a blog) wants a text-optimized serif or humanist sans with a generous x-height and real italics. A brand-forward marketing headline wants a distinctive display face for the headline tier *only*, paired with a plain, highly legible workhorse for body — never the display face at body size, where its personality becomes a legibility tax.
- **One display/heading family plus one body family is the usual ceiling.** A third family — often monospace — is justified only when the content includes real code or tabular data that benefits from fixed-width alignment. Every additional family beyond that is a dependency-weight decision like any other (Book 2, Phase 8): justify it, or don't add it.
- **Match shipped weights to what's actually used.** Don't ship nine weights of a variable font when the type scale below uses three — every unused weight is bytes on the wire for nothing.

### The type scale is a ratio, not a feeling

- Pick a scale ratio sized to the content's hierarchy depth: a **minor third (1.2)** or **major third (1.25)** for content with many close-together levels (dense UI, admin panels) where adjacent sizes should feel related, not violently different; a **perfect fourth (1.333)** or the **golden ratio (1.618)** for editorial/marketing content that wants a confident, dramatic jump from body to display headline.
- Compute the scale from a base (commonly 16px body), multiplying by the ratio per step and rounding to sensible values — this is arithmetic exactly like the spacing scale (Geometry & Spatial Composition, below), and belongs in `docs/DESIGN.md` as a table (display/h1/h2/h3/body/caption — size, weight, line-height), not re-guessed per screen.
- **Line-height tightens as size increases.** Large display type wants tighter line-height (~1.0–1.15) because its own letterforms already provide visual separation; body text wants generous line-height (~1.4–1.6) because dense small text needs the extra vertical air to stay readable. One line-height value reused at every size is a common tell that the scale wasn't actually designed.
- **Line length (the 45–75ch measure) is the type scale's real partner** — a well-built scale still produces unreadable paragraphs at any font size if the container isn't also capped to a measure; see Geometry & Spatial Composition, below.

### Pairing has a mechanical check, not just a look

Two typefaces pair well when they're either clearly **contrasting** in structure (a geometric sans display against a humanist serif body — different enough that the pairing reads as deliberate) or from the same **superfamily** (a sans and serif designed together to share metrics and proportions). Avoid pairing two faces that are almost-but-not-quite similar (two different humanist sans faces at similar weights) — that reads as an accident, not a choice.

### Performance is part of the type decision

Self-host or use `font-display: swap` with a metric-matched fallback font, so the fallback-to-webfont swap doesn't reflow the page (a layout-shift bug — see Motion & Performance Discipline, below); subset variable fonts to the weights and character sets actually used. A third or fourth typeface family is a bundle-size cost that has to justify itself the same as any other dependency.

Record in `docs/DESIGN.md`: the chosen families and why (tied to the content/reading-mode reasoning above), the scale ratio and base, the full scale table with line-heights, and the font-loading strategy.

---

## Geometry & Spatial Composition (extends Phase 3)

Generic-feeling UI is rarely one big mistake — it's a hundred small containers whose padding, gaps, and internal alignment were each eyeballed independently, so nothing shares a rhythm and small inconsistencies compound into a feeling of "not quite right" that's hard to point at. Treat every container's internal geometry as arithmetic, not feel — the same discipline the layout-grid math in Design Discovery applies at the page level, applied here at the level of every individual box.

### The container calculation procedure — run per container, not by feel

Every rule in this section and in Component Micro-Design (below) is an ingredient. This is the recipe that calls them, in order, for any container that repeats or serves as a template (a card, a button, a chip, a panel, a nav item, a modal) — a one-off decorative element doesn't need it, but anything the product will render more than once does. Work through these in order and **write the resulting numbers down** (in the component's micro-spec, in `docs/DESIGN.md`, or as a token comment) instead of nudging a value until it looks right:

1. **Identify the content extremes.** What's the shortest realistic content this container will hold, and the longest — a two-word label vs. a twelve-word one, an empty state vs. a five-line description? The container has to work at both ends; sizing it to whatever's currently in the mock is designing for a placeholder, not the product.
2. **Assign padding from the spacing scale**, choosing top/bottom independently from left/right per the optical rule below (text usually wants less top than bottom; buttons usually read better wider than tall). State which two scale steps were chosen and why — not "it looked balanced."
3. **Assign gaps to siblings and children from the same scale**, one step, chosen once and applied everywhere this container repeats. If it nests inside another container, confirm the compounding rhythm still reads as one system (outer step, inner step) rather than two unrelated scales colliding.
4. **Compute any icon/text/adornment relationship the container holds** — mirrored padding for a trailing icon, optical (not bounding-box) centering, cap-height alignment — using the exact formulas in Optical Alignment, below, and in Component Micro-Design. Write the pixel relationship down; don't nudge it by eye.
5. **Check the container against the viewport it will actually render in.** Does it fit at the smallest supported breakpoint without an unintended scroll, overlap, or truncation? If it's part of a fixed-height composition (a hero, a modal), it has to pass the "sum the real content height against the target viewport" check in Fitting the Viewport, below.
6. **Confirm it survives resize and reflow, not just its authored size.** A non-uniformly-scaled shape (an inline SVG, a `clip-path`), a fluid grid column, or a flex basis — check the container at both ends of its supported width range, not only at the width it happened to be designed at.

Skipping straight to step 2 without doing step 1 first is the single most common way a container "looks fine now" and reads as **stuck** — cramped, jammed, or awkwardly clipped — the moment real content or a real viewport hits it. That failure mode is the arithmetic version of this workflow's own Definition of Ready (Book 2, Phase 8): a recurring container isn't Ready to ship until this procedure has actually been run for it, not assumed to be fine because the happy-path mock looked right.

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

### Full-bleed chrome vs. the centered content well

A recognizable AI-default mistake: a nav bar, footer, or hero background gets wrapped in the same centered `max-width` container as the body copy, so on any screen wider than that container the nav's logo and links sit stranded in a narrow strip with large, empty margins on both sides — the page reads as "shrunk to the middle" instead of using the viewport it's actually given. Treat these as two different geometry decisions, not one:

- **Viewport-anchored chrome** (the nav bar, the footer, full-bleed section backgrounds and hero imagery) runs to the *true* edge of the viewport, with only a small, deliberate outer margin — one spacing-scale step, scaled by breakpoint (e.g. ~24px on mobile, ~40–64px on desktop), never the wide gutter a centered content column uses. A logo or a primary nav item sits close to the actual edge of the screen — the way a printed page's running header sits near the paper's edge — not floating in the middle of dead space because a `.container` class was applied to it out of habit.
- **The centered content well** (body copy, forms, article text) keeps its own `max-width` and centers for readability — that constraint exists for the *content*, not for the chrome that frames it. A page usually has both at once: full-width chrome, with a centered reading column inside it.
- **Compute the outer margin explicitly; don't inherit a component library's default container padding.** State it in `docs/DESIGN.md` as its own token, distinct from the content column's max-width, and audit it specifically at the desktop breakpoint — that's where the "shrunk to the middle" mistake is most visible and most often missed, because the authoring session was previewing at a narrower window.
- **This is a per-element hierarchy decision, the same as a bento tile's span.** Decide, for each element, whether it belongs to viewport-anchored chrome or the centered well — don't let one global container class make that call for everything placed inside it by default.

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

### When the list gets long: the dropdown becomes a searchable combobox

A plain scrolling option list stops working once the list is long — the failure mode is the user typing a letter and hoping, or scrolling and squinting. Decide the threshold explicitly, per this project's actual data, rather than defaulting either way: past roughly 7–10 options, the control gets a search/filter input, not a longer scrollbar.

- **The search input takes the top of the panel on open**, autofocused, full-width minus the panel's own padding (the same padding rule as the panel itself), with a small leading search icon aligned by the icon-and-text optical-centering rule above.
- **Filtering is live, and the matched substring is visually distinguished** (bold, or the accent color, on the matched characters) — a filtered list with no indication of *why* an item matched forces the user to re-read every result anyway.
- **Debounce only if filtering is expensive** (a remote search); a local filter over an in-memory list should feel instant, with no artificial delay bolted on.
- **An explicit empty-results state** — "No matches for '{query}'" — ideally with a way to clear the query or, if this is a quick-add-eligible dropdown (Workflow-First Screen Design, above), add a new record inline without leaving the search.
- **Keyboard behavior doesn't regress.** Arrow keys move through the *filtered* results, Enter selects the highlighted one, Escape clears the query before closing the panel entirely (two-stage escape, not one) — a search box that breaks the plain dropdown's keyboard model is a regression wearing an upgrade's clothes.
- **Whether search is worth it at all is a per-project decision, not a toggle applied everywhere.** A 6-item status dropdown never needs it; a country picker, a tag combobox, or a searchable user-assignment field almost always does. State the threshold and the reasoning in `docs/DESIGN.md`'s component micro-specs, the same as the plain dropdown's anatomy.

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

### Preloaders: gating on real readiness, not a decorative delay

A preloader and the first-visit intro (below) solve different problems and are easy to conflate. The intro choreographs content that's already loaded; a preloader gates the page *while something genuinely isn't ready yet* — a large hero video, a 3D scene, a custom font, an image sequence. The distinction matters because a preloader with nothing real to wait for is exactly the bolted-on delay users notice and resent.

- **Earn it with an actual readiness check.** Tie the preloader to a real signal — `document.fonts.ready`, decoded image/video promises, a 3D scene's own asset-load callback — never a fixed `setTimeout` standing in for work. If the real assets are ready in 200ms, the preloader shows for 200ms; artificially stretching it to "let the animation play out" is the dead-time anti-pattern, not a fix for it.
- **Continue the same visual world; don't cut to a blank slate.** A well-built preloader shares the destination page's background color and, where the design concept calls for it, a piece of its own decorative vocabulary (a shape from the project's own shape system, not a generic spinner) — so resolving it reads as a continuation into the hero, not a hard cut between two differently-styled screens. (Observed directly while researching this book: a recent Awwwards Site of the Day builds its preloader from small rotated icon shapes cycling in a ring, on the *exact* background color the hero uses, with the hero's own faint decorative line-work already visible behind it — the loading state and the destination state are visually one continuous thing, not two.)
- **A percentage counter is a legitimate pattern, but it's a promise.** If the design concept calls for a numeric counter (0→100%), it tracks *real* load progress, not an eased animation loosely timed to feel like loading — a counter that visibly doesn't correspond to anything real breaks the moment a user's connection is faster or slower than the animation assumed.
- **Not every project earns a preloader at all.** If there's nothing substantial to wait for — no heavy hero asset, a fast, mostly-text page — the honest move is no preloader, or one so brief it's never actually seen. Multiple well-regarded sites checked while researching this book have no visible gate whatsoever: content is ready fast enough that the page never needs one, and forcing a preloader onto a fast-loading page just to manufacture an "entrance moment" is decoration wearing a technical justification.
- **The handoff into the first-visit intro is one choreographed moment, not two animations bumping into each other.** Build the preloader's exit and the intro's entrance as a single timeline — the preloader doesn't fade to blank before the intro starts a beat later; the intro's opening state *is* the preloader's resolved state, continuous in time.
- **The same continuity principle extends to page-to-page route transitions in an SPA.** A curtain wipe, a shared-element morph (the clicked card becomes the next page's hero, via FLIP), or a brief cross-fade — chosen per the design concept the same way the signature interaction is chosen (above), never picked from a generic "page transition" list. Never a full white flash between routes, and never a heavier transition than the navigation frequency can afford: a transition that delights the first time and delays every subsequent one is a tax, not a feature — the same restraint math as the first-visit intro's once-per-session rule, applied to something that fires on every navigation instead of once.

Record the decision in `docs/DESIGN.md`: whether a preloader exists and what real signal it gates on, or "none, deliberately, because load is already fast" — and the page-transition pattern, if the project is an SPA.

### The first-visit intro (extends Phase 1's Design Discovery)

An intro sequence — a brief, deliberate animated moment before or as the main UI settles, picking up once any preloader gate above has lifted — is, per this workflow's house default (Cinematic Direction, above), *present* on every public-facing project; what remains a per-project decision is its **pattern**, derived from the design concept, never pulled from a generic effects library. The craft rules:

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

### One signature interaction, earned by the concept, not borrowed from a reference

Beyond the standard micro-interaction vocabulary (hover, focus, entrance, scroll-reveal), a project whose design concept has real personality can earn exactly one **signature interaction** — a custom-built, memorable moment that only this product does, tied to what the product actually is. This is the highest-risk, highest-reward move in this book, and the rules are correspondingly strict:

- **It's derived from the design concept, not copied from an admired reference.** A cursor-driven reveal, a physically-simulated material (liquid, cloth, glass), a custom cursor, a hover state that reshapes the whole layout — any of these can be the signature move, but the *choice* of which one comes from what this specific project is about, not from "that cool site did X." A creative studio's portfolio can support something playful and technically showy that a healthcare intake form never should — the same restraint principle as scroll-triggered motion applies at the concept level: not every project earns a signature interaction, and forcing one onto a project whose concept doesn't call for it is decoration, exactly like an unearned scroll effect.
- **Construction pattern A — a pointer-driven reveal** (one representative example, not a template to copy verbatim): a foreground layer masks a background layer — via `mask-image`/`clip-path` with a radial or freeform gradient, or an SVG mask — whose position tracks the pointer. Update the mask's position on `requestAnimationFrame` (or a library ticker, e.g. GSAP's `quickTo`) rather than on every raw `pointermove` event, and drive it through **transform or a CSS custom property**, never a layout-triggering property, per the animate-only-transform-and-opacity rule above. Smooth the tracked position with a **lerp or spring**, not a 1:1 snap to the raw cursor coordinate — the trailing lag is exactly what makes it feel fluid rather than glitchy.
- **Construction pattern B — a morphing mask that swaps material, not position.** A different, equally legitimate version of the same idea: instead of a mask that *moves*, an irregular mask (built from the same organic-blob technique as Decorative Shape Vocabulary, above — a small number of unequal bézier lobes, never a mirror-symmetric blob) that itself *deforms* over scroll or time, revealing a different rendered variant of the same content underneath — a different color, material, or texture treatment of the identical headline, so the piece reads as one word with two "skins" and a window that drifts between them. Build it as two stacked, identically-positioned copies of the same element (each with its own material/texture styling) with the top copy clipped by the animating blob mask — never as swapped image assets crossfading, which loses the "peeking through a window" quality that makes this pattern read as a single object rather than a slideshow. The mask's shape interpolates between a small number of keyframe blobs (again, deliberately asymmetric ones) rather than jumping between two fixed shapes.
- **The return-to-rest state is where the "fluid" feeling actually lives**, for pattern A specifically. On pointer leave (or after a period of no movement), the reveal doesn't just disappear — it eases back to its default/closed state through the same spring physics used for direct-interaction motion (Easing and the Physics of Smoothness, above), so releasing the interaction feels like a material settling, not a hard cut. This is a spring, explicitly — a fixed-duration authored curve reads as mechanical for this kind of continuous, physically-motivated motion. Pattern B has no "release" moment by nature (it's ambient, not interaction-triggered) — its equivalent discipline is a slow, continuous, never-quite-repeating cycle rather than a visibly looping animation, which is what keeps an ambient effect from reading as a canned loop on a second viewing.
- **It degrades honestly, not just gracefully.** No persistent pointer on touch devices — either substitute a tap-triggered variant with its own considered animation, or omit the effect entirely rather than leaving dead code that never fires; `prefers-reduced-motion` collapses it to a static or near-static state; and it never sits between the user and real content or blocks a real interaction underneath it.
- **Exactly one, ever, per project.** Two competing signature moments on the same product read as showing off, not designing — pick the one moment (usually the hero, or the single screen people will remember) and let the rest of the product be calm by comparison. Record the decision — what it is, why this project's concept earns it, and the fallback for reduced motion and touch — in `docs/DESIGN.md`, the same way the first-visit intro decision is recorded above.

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

## Render & Motion Correctness — flicker, flash, and layout disruption are bugs

Everything above decides what the motion *should be*; this section is about the glitch class that makes individually-correct animations read as broken on the real page. Every rule here has a reproduction step, which is what makes it a correctness bug rather than a taste question — and the Render QA pass at the end of this section is the frontend's twin of Phase 9's "actually run it and watch it": mandatory, empirical, and run in a real browser before any UI slice is Done.

### The first-paint contract: nothing ever flashes

The most common glitch in AI-built frontends: an element renders visible in its final position, JS loads and snaps it to its hidden pre-animation state, then it animates back in — the user watches content appear, vanish, and re-appear. The contract that kills the whole family: **the first painted frame already shows every element in its correct initial state, and no later frame ever shows an element in a state that precedes the one already shown.**

- **Elements that will animate in are hidden by CSS (or SSR-inlined styles) before the animation code ever runs** — never hidden *by* the animation code after a visible frame has painted. With GSAP that means the initial state lives in a stylesheet (or a synchronous `gsap.set()` before paint) and the tween animates *to* the settled state — a bare `gsap.from()` firing after hydration is the flash, mechanically. With CSS-driven entrances, the keyframe's `from` state is also the element's resting style until the animation starts, so there's no gap frame between them.
- **Every hidden-until-animated element has a no-JS failsafe.** Content invisible while waiting for an entrance must become visible even if the animation code never executes — gate the hidden state on a class the animation runtime adds (`html.js` / a "motion ready" attribute), so a failed bundle, a blocked script, or a crawler gets a fully visible page. A visitor on a bad connection staring at a blank hero because a stagger never fired is a total outage of the page, caused by a decoration.
- **Theme and font flashes obey the same contract.** Dark/light theme resolves *before* first paint (an inline head script reading storage — never an effect after hydration, which paints the wrong theme for a frame); fonts load with `font-display: swap` and a metric-matched fallback so the swap doesn't reflow (Typography System, above).
- **In SSR/hydrating frameworks, client-only state never participates in the initial render.** The sessionStorage "intro already played" flag, viewport width, `Date.now()`, randomness — reading any of these during render produces server/client mismatch, and the visible symptom is exactly the DOM flicker this contract bans (React warns; the user just sees the glitch). Gate on a mounted flag and reserve the space so the post-mount swap doesn't shift anything.
- **Measurements that decide what the first frame looks like happen before paint.** A position, size, or line-split read in a post-paint effect (`useEffect`) paints one wrong frame first, and that single wrong frame *is* the flicker — use the framework's before-paint hook (`useLayoutEffect` or equivalent) for anything whose result changes what's painted.

### Layout isolation: an animation may not move the page

The standing rule, stated once: **no animation changes the page's scroll height, its scrollbar state, or the position of any element outside itself — unless that movement is the animation's explicit purpose** (a FLIP reorder, an intentional accordion). Transform/opacity (above) covers *which properties* are cheap; this covers *containment* — a pure-transform animation can still wreck the layout around it:

- **Off-screen entrances are clipped at the section, or they widen the page.** An element translating in from beyond the viewport edge extends the document's layout while off-screen — a horizontal scrollbar appears, the vertical scrollbar changes, and the entire page shifts by the scrollbar's width, once on entrance and once back. Clip at the nearest section wrapper with `overflow-x: clip` — preferring `clip` over `hidden`, because `hidden` creates a scroll container that silently breaks `position: sticky` inside it and can itself be scrolled by focus/JS. Never solve it with a blanket `overflow-x: hidden` on `body` — that hides the symptom on every page while leaving the overflow (and its mobile-browser quirks) in place.
- **Decorative and floating elements live out of the document flow.** Anything that drifts, floats, or follows the pointer is absolutely positioned inside a positioned, size-reserved parent, so no frame of its motion can push a sibling. If removing the element would change where anything else sits, it isn't isolated yet.
- **Hover and focus states never change layout-affecting properties.** Font-weight on hover reflows the whole nav; a border appearing on hover nudges the content inside it by its own width. Reserve instead: a pre-existing transparent border that only changes color, an inset box-shadow, a width-locked label, `transform: scale()` instead of size changes. If toggling the state moves any pixel outside the element, it's a layout bug wearing a hover style.
- **Everything that arrives late has its space reserved before it arrives** — `aspect-ratio` on media, fixed-dimension skeletons, `min-height` on sections whose content mounts after JS or data. This is the slow-network rule (above) restated as a layout-isolation rule, because the symptom is the same: the page visibly reorganizing itself around content the user didn't ask to move. Conditionally-rendered elements either keep their space when absent or enter via transform/opacity *within* reserved space.
- **Scroll locking and modals account for the scrollbar they remove.** Opening a modal that sets `overflow: hidden` on the body removes the scrollbar and shifts the entire page by its width — set `scrollbar-gutter: stable` (or compensate with padding equal to the scrollbar width) so overlays open without a jolt.
- **Transforms create stacking contexts and containing blocks — structure around this, don't fight it.** A transformed (or filtered, or `will-change`d) ancestor traps `position: fixed` descendants and re-scopes z-index, which is where "the dropdown renders under the next card" and "the modal is stuck inside the section" come from. Rules: never leave a lingering `transform` on an ancestor of fixed/sticky content (clear it when the animation ends); render overlays — modals, dropdowns, tooltips, toasts — through a portal or the platform's top layer (`<dialog>`, popover API) instead of escalating z-index; and keep a documented z-index scale (a handful of named tiers in DESIGN.md), because an arms race of `z-index: 9999` is the symptom of structure being wrong, not the fix.

### Lifecycle correctness: animations that survive a real, messy session

Demos play an animation once, forward, on a fresh page. Real users interrupt it, reverse it, resize during it, and reload halfway down the page — and this is where "glitchy" actually lives:

- **Every interactive animation is interruptible and resumes from its current state.** Hover-out mid-hover-in, close mid-open, re-trigger mid-play: the transition continues from wherever the element currently is — never a snap to the start, never a second tween stacking on the first (visible as acceleration or a double-play). CSS transitions and Framer Motion do this natively; GSAP needs `overwrite: 'auto'` or an explicit kill of the previous tween; hand-rolled rAF animation almost never gets this right, which is a reason not to hand-roll it.
- **Everything an animation registers, it unregisters.** ScrollTriggers, timelines, listeners, and observers are cleaned up when their component unmounts (`gsap.context()`/`useGSAP`, effect cleanup functions). The failure is invisible until it isn't: navigations leak listeners animating dead elements, and React's StrictMode double-invocation duplicates every un-cleaned trigger — the classic "animations run twice as fast / jump twice" bug is exactly this.
- **Scroll-trigger positions are computed against the final layout, not the loading one.** Trigger points and pin distances measured before images and fonts finish loading are simply wrong — reveals fire at the wrong scroll position, pins jitter. The primary fix is the reserved-space rule above (if nothing shifts after load, the early measurement stays valid); the fallback is an explicit refresh (`ScrollTrigger.refresh()` or equivalent) after `document.fonts.ready` and hero-media load.
- **Scroll effects work in both directions, and from a mid-page start.** Browsers restore scroll position on reload, and users arrive at anchors — a page whose entrance states only resolve when scrolled from the top leaves everything above the restored position permanently invisible. Enter-once reveals must resolve (not re-hide) when scrolled back up past them; scrubbed effects must land on their correct value when the page loads already past them.
- **Pinned sections use the library's pinning, never a hand-rolled `position: fixed` swap** — the one-frame gap between unpinning and reflowing is the classic scroll jitter, and the library's pin-spacer exists precisely to remove it.
- **Route changes are a lifecycle event, not an interruption to ignore.** Navigating away mid-animation must not leave console errors, orphaned timelines, or a scroll position inherited by the next page; navigating back must not replay the intro (the once-per-session rule, above) or re-run entrance staggers on content the user has already seen.

### Text stays readable — at rest, in motion, and after motion

Motion never gets to spend legibility. The reading experience is the product; the animation is presentation:

- **Body text is never animated while it's being read.** No parallax on paragraphs, no continuous drift under running copy. Text that moves is display-scale, a few words, and decorative — and it comes to rest before the user needs to actually read it. Entrance fades on text finish fast (the 200–400ms entrance budget above); a slow, lingering fade on a paragraph is the user waiting to be allowed to read.
- **Transforms must not leave text blurry.** Scaling rendered text rasterizes it mid-tween and can leave it soft afterward: animate a wrapper rather than the text node, always end at exactly `scale(1)`, land translations on whole pixels (a resting `translateY(0.5px)` renders soft), and remove `will-change` when the animation completes so the browser returns the element to crisp synchronous rendering.
- **Split-text reveals reassemble into real text.** A headline split into per-word/per-line spans must remain selectable and accessible — the original string exposed to assistive tech (e.g. `aria-label` on the container, spans hidden from it) — and line-based splits are recalculated on resize, because stale line splits overlap and truncate, which is the glitchiest-looking text bug there is. After the reveal completes, the DOM ideally returns to the unsplit original.
- **Text over animated or image backgrounds keeps AA contrast on every frame, not on average.** A scrim or overlay is sized for the worst frame of the video/parallax behind it; if any frame of the background makes the headline unreadable, the composition fails, regardless of how the static mock looked.

### The Render QA pass — a reproduction script, run before Done

This is the checking step, and it's not optional: run it in a real browser on every UI slice, the way Phase 9 runs the app instead of asserting about it. Each step targets one glitch family from this section; watching for it explicitly is the point — none of these show up in a passing test suite or a static screenshot:

1. **Hard reload with cache disabled; watch the first two seconds.** Any element visible-then-hidden-then-animated? Theme flash? Font swap reflow? Skeletons matching final layout? (First-paint contract.)
2. **Scroll halfway down, reload, and scroll both ways.** Does everything above and below the restored position resolve visible? Do reveals behave scrolling *up*? Fast-scroll the whole page — anything that fires late, re-fires, or stays hidden? (Lifecycle.)
3. **Watch the scrollbars through every animation.** The vertical scrollbar must not change length during entrances; a horizontal scrollbar must never appear at any viewport, including mid-animation. (Layout isolation.)
4. **Hammer every interactive animation.** Rapid hover in/out, open/close a dropdown or modal five times fast, re-trigger entrances — everything transitions from its current state, nothing stacks, snaps, or double-plays. (Interruptibility.)
5. **Resize during and after animation; rotate at mobile widths.** Line splits recalculate, shaped/clipped cards stretch per their construction, pinned sections re-measure, nothing overlaps or strands. (Late layout.)
6. **Navigate away mid-animation and back.** No console errors, no orphaned motion, no intro replay, scroll position sane. In React dev mode, confirm StrictMode double-mount doesn't duplicate triggers. (Cleanup.)
7. **Read every piece of moving or revealed text out loud, literally.** If any of it can't comfortably be read at the moment the eye lands on it — too fast, still moving, low-contrast against its frame, blurry after settling — the text wins and the motion is retuned. (Legibility.)
8. **Open DevTools' rendering aids once per slice**: Layout Shift regions and Paint Flashing (or the Performance panel's CLS/long-frame view) to catch the shifts and full-page repaints that are visible to the metrics but easy for a human to rationalize away.

A slice that hasn't had this pass run — actually run, findings fixed or recorded — is not Done, exactly as an unverified backend slice is not Done. "The animation code looks correct" has the same evidentiary value as "the tests should pass": none.

---

## UI Slice Verification (extends Phase 9)

For any slice with a UI surface, Phase 9's end-to-end verification step expands to this checklist — same spirit: actually run it and watch it, don't assert it:

- [ ] Driven in a real browser at **three viewports minimum** (a small phone ~360px, a tablet ~768px, a desktop ~1280px+) — layout re-flows as designed at each, nothing overflows or overlaps
- [ ] **Empty, loading, error, and populated** states all reached and all look intentional
- [ ] Driven once with **network throttling (Slow 3G) and CPU throttling** — skeletons appear, nothing shifts when content lands, animations stay smooth or gracefully absent
- [ ] **No layout shift** on load (watch for it explicitly; images/fonts/embeds have reserved space)
- [ ] **The Render QA pass ran, all eight steps** (Render & Motion Correctness, above) — first-paint watched on a hard reload for any visible-then-hidden flash, mid-page reload resolved, both scroll directions driven, scrollbars stable through every animation, interactive animations hammered for interruptibility, resize during animation, navigation mid-animation clean, moving/revealed text actually read
- [ ] Animations follow the transform/opacity rule; nothing stutters during entrance or interaction
- [ ] **No animation changes the page's scroll height or any element outside itself** unless that movement is its explicit purpose — off-screen entrances clipped at their section, hover/focus states free of layout-affecting property changes, overlays portal/top-layer rendered rather than z-index-escalated under a transformed ancestor
- [ ] Composed entrances (card grids, staggered lists) are driven by one owned timeline/stagger, not independently-triggered animations that visibly drift under real load
- [ ] Small controls (dropdowns, buttons, chips, tooltips) checked against Component Micro-Design's anatomy — icon-to-text relationship, all states present, viewport-collision handling verified, not just the happy-path position
- [ ] If this slice includes a first-visit intro: verified it runs once per session, has a working skip path, and collapses correctly under reduced motion
- [ ] If this slice includes a preloader: verified it ties to a real readiness signal — runs shorter on a fast connection and longer on a throttled one, proportionally, rather than a fixed duration either way; and its exit continues into the intro/hero as one moment, not a visible cut
- [ ] If this project has SPA route transitions: verified they never flash blank/white and stay affordable at real navigation frequency, not just on the first click
- [ ] Shaped/cut cards (docking notches, blisters, chamfers): the cut and its docked content verified as computed from the same numbers, and the shape checked at least at the small-phone and desktop viewports to confirm it still stretches sensibly, not just at the size it was designed at
- [ ] Searchable dropdowns/comboboxes: filtering, the empty-results state, and full keyboard behavior (arrow keys through filtered results, Enter, two-stage Escape) verified, not just the mouse path
- [ ] If this slice includes the project's signature interaction: pointer, touch (or its substitute/omission), and `prefers-reduced-motion` fallbacks all verified, and it never blocks real content or interaction underneath it
- [ ] **`prefers-reduced-motion` verified** to actually reduce motion
- [ ] **Keyboard-only pass**: every interactive element reachable and operable, focus visible, tab order sane; dropdown quick-add and modals trap and return focus correctly
- [ ] Hover, focus, active, and disabled states present on interactive elements
- [ ] Text contrast meets WCAG AA; touch targets are comfortably tappable (~44px); the accent color's specific pairings (e.g. button text on button background) checked, not just body text
- [ ] If the project supports dark mode: verified against its own re-derived role table, not assumed to be "the same but inverted"
- [ ] Any form worth >30 seconds of input survives an accidental refresh/navigation (draft protection verified, not assumed)
- [ ] Viewport-anchored chrome (nav/footer) checked at a wide desktop viewport specifically — hugs the true edge with its own small margin, not stranded in the centered content well's gutter
- [ ] Every recurring container on the screen was run through the Container Calculation Procedure — checked at its shortest and longest realistic content, not just what's in the mock — and none of it reads as stuck, cramped, or awkwardly clipped
- [ ] For a major UI surface: its **design checkpoint ran** — the user saw the built section and the feedback was applied or recorded (Cinematic Direction, above); the slice is not Done on a surface the user never saw
- [ ] If the slice ships a 3D scene: GLB compressed (Draco/meshopt), loaded off the critical path with reserved space, DPR capped, render loop pauses offscreen/hidden-tab, reduced-motion/low-end fallback poster verified, and its frame-time/bundle budget in `PERFORMANCE_BASELINE.md` measured, not assumed
- [ ] The screen matches `docs/DESIGN.md` tokens — spacing, type, color roles, radii, motion, outer-margin, signature-interaction fallback — with no per-slice inventions

The workflow-level check, once per task rather than per slice: walk the *entire core task* end-to-end as a first-time user and count the steps and page hops. If the count exceeds what DESIGN.md promised, that's a plan-vs-reality conflict — handle it via Phase 14, don't quietly accept the worse workflow.

---

## Anti-Patterns (the house style to actively avoid)

- **The default AI template**: centered container, hero, three feature cards, uniform card grids for everything. If the delivered UI would look at home in any other AI-generated app, design discovery was skipped — go back and do it.
- **The card-stack page**: every section, paragraph, and image wrapped in its own bordered, shadowed, rounded container, stacked vertically with uniform gaps. Containment without meaning is the single strongest "AI-built" tell — content belongs *in the scene* (Cinematic Direction, above); visible containers are reserved for enumerable homogeneous items and bounded interactive surfaces.
- **A page with no narrative arc** — sections in template order (hero, features, testimonials, CTA) with no stated job in the story, no camera grammar, and entrances from nowhere. Directed pages are storyboarded first.
- **3D as a stock ornament** — a floating generic shape with no relationship to the project's artifacts, shipped uncompressed on the critical path, rendering at full DPR while offscreen.
- **The default AI palette**: an indigo/blue primary, white/light-gray surfaces, a purple gradient hero — chosen because it's inoffensive, not because a distribution or harmony rule was chosen for this project. If the palette would look at home in any other AI-generated app, the Color System section was skipped.
- **Color or type picked hex-by-hex or font-by-feel** instead of from a stated distribution ratio (60-30-10 or otherwise), a named harmony rule, and a scale ratio — the tell is that nobody could state *why* this accent, this ratio of type sizes, or this pairing, beyond "it looked nice."
- **One line-height reused across every type size**, or a type scale with no stated ratio — sizes that were each picked by eye rather than computed from a base and a ratio.
- **Semantic (success/warning/danger) colors reassigned to match the brand harmony** — status legibility is more important than palette consistency; don't make "danger" a brand-adjacent color because red clashes.
- **Dark mode as a naive inversion** of the light palette instead of a re-derived role table — muddy desaturated surfaces, an accent that vibrates or loses contrast on a dark background, or a text hierarchy that lost its relative contrast steps.
- **Component-library defaults shipped as the design.** Libraries are fine as *foundations*; unthemed defaults are a visible tell. The DESIGN.md tokens must be applied over whatever library is used.
- **A long option list left as a plain scrolling dropdown** past the point where a search/filter input was clearly needed — or, the reverse, a search box bolted onto a 4-item list that never needed one.
- **Chrome (nav/footer) wrapped in the same centered container as body content**, stranding it in a narrow strip with large dead margins on wide viewports instead of running to the true edge with its own small, deliberate margin.
- **A signature interaction borrowed wholesale from an admired reference** instead of derived from this project's own design concept — or more than one signature interaction competing on the same product.
- **A signature interaction with no touch/reduced-motion fallback**, or one that blocks real content or a real interaction underneath it.
- **One entity, one page.** Admin UIs that mirror the schema instead of the task — the quick-add/bulk-add section exists specifically to kill this.
- **Scroll-triggered animation on everything.** Reserve scroll animation for a few deliberate moments, if any; ambient constant motion reads as noise and costs performance exactly where it's least affordable.
- **A trailing icon or chevron placed by guesswork** — flush against the edge, floating with no stated relationship to the text baseline, or unsynced with the panel it controls opening.
- **Uniform fade-everything-in** — every element on a screen using the identical entrance regardless of visual hierarchy or grouping; a composition should read as arriving, not as a checklist being ticked off.
- **An intro sequence that replays on every navigation**, traps input, or hides real content behind an unrelated splash animation.
- **A preloader stretched with a fixed timeout to "let the animation finish"** instead of gating on real asset readiness — or a percentage counter that doesn't track anything real.
- **A preloader that cuts to a differently-colored or differently-styled page** instead of continuing the same visual world into the hero.
- **A preloader added to a page with nothing substantial to wait for**, manufacturing an entrance moment instead of gating real load.
- **A page-to-page transition heavy enough to become a tax on frequent navigation**, or one that flashes blank/white between routes.
- **Nested staggers that multiply** into a multi-second cascade instead of one owned, composition-level stagger.
- **Springs used where a fixed, synced duration was needed** (elements landing out of sync with a scroll-locked or timeline-driven sequence) **or authored curves used where physical responsiveness was needed** (a dragged element that doesn't feel connected to the cursor).
- **A docked notch/blister that doesn't match its docked element's actual size** — cut and content computed independently instead of from one calculation, so they drift the first time either one's size changes.
- **A perfectly uniform, identically-opaque decorative scatter pattern**, or a symmetric "blob" accent whose lobes mirror each other — both read as generated defaults, not designed shapes.
- **Shaped/clipped cards copied verbatim from a supplied asset kit into an unrelated project or size**, instead of re-deriving the same construction (notch width, chamfer ratio, scatter falloff) for this project's own components and sizes.
- **The first-paint flash**: content rendered visible, snapped hidden by late-arriving animation code, then animated back in — or a theme/font flash from resolving client state after hydration. The initial state belongs in CSS before paint, with a no-JS failsafe (Render & Motion Correctness, above).
- **An entrance that moves the page**: an element translating in from off-screen without a clipping section wrapper, spawning a horizontal scrollbar and shifting the whole layout by the scrollbar's width — or the blanket `overflow-x: hidden` on `body` that hides that symptom site-wide while breaking sticky positioning.
- **Hover states that reflow**: font-weight, borders, or size changes on hover that nudge every sibling — reserve the space (transparent border, width lock, `scale()`) instead.
- **Non-interruptible animations**: re-triggering mid-play snaps to the start or stacks a second tween (double-speed, double-play) — every interactive transition continues from the element's current state.
- **Leaked animation lifecycles**: ScrollTriggers, timelines, and listeners never cleaned up on unmount — dead elements animating, StrictMode double-mounts running everything twice, the next route inheriting orphaned motion.
- **Scroll effects that only work top-down on a fresh load**: reveals that never resolve after a mid-page reload or anchor arrival, or that re-hide content when the user scrolls back up.
- **Text spent on motion**: paragraphs under parallax, headlines left blurry by a scale tween that didn't land on `scale(1)`, split-text spans that break selection and screen readers or overlap after a resize, text over an animated background that loses contrast on its worst frame.
- **Spinner-only loading** and **animations that gate content** (users wait for the fade-in to finish before they can act).
- **Confirmation dialogs as a safety blanket** on reversible actions, instead of undo.
- **Per-slice style invention** — a new gray, a new gap value, a new duration "just this once." That's source-of-truth drift; the tokens live in DESIGN.md.
- **A container sized or padded by feel** — "it looked balanced" instead of a stated spacing-scale step and a check against real content extremes (the Container Calculation Procedure, above). This is the single biggest source of the generic, slightly-stuck feeling users notice without being able to name.
- **Designing only the happy, populated, fast-network state.** Real first impressions are empty states on slow connections.
