# Book 3 — Reference Templates

Part of a four-book set. Start at `SKILL.md` if you haven't already. This book is copy-pasteable templates for every document the other books reference. For the big-picture methodology (discovery, planning, architecture, context management), see `methodology.md`. For day-to-day implementation discipline (how a slice gets built, verified, committed), see `coding-standards.md`. For frontend experience and design, see `frontend-design.md`.

This is the book to load when actually creating one of these files for the first time in a project — otherwise it's reference material, not something to read start to finish.

---

## `NEXT_SESSION.md`

Referenced in `methodology.md` Phase 12 (Context & Memory Management). Written in the working-doc register (Phase 12): compressed wording, every fact/path/why kept.

```markdown
# Next Session — Start Here

Overwritten every slice, not appended to. For history, see CHANGELOG.md and
ROADMAP.md's Progress Log.

## Cold-start reading order
[The specific, narrow list of documents to read for the next slice — not the whole doc set.]

## Last completed slice
[Slice ID] — [one line]. See docs/features/<slice-id>.md.

## Next slice to implement
[Slice ID] — [objective, files likely to change, dependencies]
- [Any necessary-but-unlisted infrastructure it will likely need, flagged now.]
- [Any gap between this slice's own spec and something already decided, flagged now.]

## Standing decisions this session should not re-litigate
- [Decision] — [why, and what it implies for future work]

## Open questions / gaps flagged, not yet resolved
[Only genuinely open items — not the ones above with a clear resolution path.]
```

---

## Per-slice note (`docs/features/<slice-id>.md`)

Referenced in `methodology.md` Phase 12's two-tier documentation pattern. Written in the working-doc register (Phase 12).

```markdown
# <Slice ID>: <Title>

## Objective
[One or two sentences.]

## Files Changed
[Grouped by layer/role.]

## Necessary additions beyond this slice's literal scope (disclosed)
[Anything added that wasn't explicitly listed, and why it was unavoidable.]

## Bugs caught during self-review (disclosed)
[Real issues found and fixed before this was called done — not hidden.]

## Testing Completed
[What was actually run, including any end-to-end/real-environment verification.]

## Known Issues
[Honest — "None known" if true.]

## Next Dependencies
[What this unblocks.]
```

---

## Per-module Feature Summary (`docs/features/<module-name>.md`)

Referenced in `methodology.md` Phase 12's two-tier documentation pattern.

```markdown
# <Module Name>

## Objective
## Files Changed (by layer)
## Database Changes
## APIs Added
## UI Changes
## Business Rules Implemented
## Testing Completed
## Security Review
## Performance Notes
## Known Issues
## Next Dependencies
```

---

## Project Map (`docs/PROJECT_MAP.md`)

Referenced in `methodology.md` Phase 12 (The Project Map) and updated incrementally in `coding-standards.md` Phase 10. Written in the working-doc register — this file is read more often than any other, so terseness matters most here. `[E]` = entry written/updated with the file open; `[I]` = inferred, verify before relying on it. The example entries below show the register in action:

```markdown
# Project Map

Index only — code wins on conflict. Update touched entries per slice (Phase 10),
never regenerate whole map. Repo scan only on map miss; backfill what scan finds.

## Load-bearing (know these before touching anything near them)
- `src/lib/db.ts` — DB client + transaction helper. All data access flows through it. [E]
- `src/lib/api/envelope.ts` — response envelope + error codes. Every route wraps through it. [E]

## Module: auth
- `src/modules/auth/session.ts` — session create/verify/refresh. Exports `createSession`, `requireUser`. Depends: `db.ts`, `crypto.ts`. [E]
- `src/modules/auth/middleware.ts` — route guard, wraps `requireUser`. Used by every authed route. [E]

## Module: billing
- `src/modules/billing/invoice.ts` — invoice CRUD + totals calc. Depends: `db.ts`, `money.ts`. [I]
```

---

## ADR (`docs/adr/<n>-<title>.md`)

Referenced in `coding-standards.md` Phase 10 (Documentation Update).

```markdown
# ADR-<n>: <Decision Title>

## Status
Accepted — <date>

## Decision
[What was decided, stated as a single clear sentence.]

## Reason
[Why, tied to a specific requirement or constraint — not just "it's popular."]

## Alternatives Considered
[What else was evaluated, and why it lost.]

## Trade-offs
[What is being given up by choosing this path — stated honestly.]
```

---

## Decision Log (`docs/DECISIONS.md`)

Referenced in `coding-standards.md` Phase 10 (Documentation Update) — the lightweight tier between an ADR and a Risk Register entry.

```markdown
# Decision Log

Small decisions that don't rise to ADR weight, logged so they don't get
re-litigated later. One or two lines each, dated, with the reason.

## <YYYY-MM-DD>
- <What was decided>.
  Reason: <why, in one line>.

## <YYYY-MM-DD>
- <What was decided>.
  Reason: <why, in one line>.
```

---

## Roadmap Progress Log entry

Referenced in `methodology.md` Phase 12 (Context & Memory Management).

```markdown
| Slice | Status | Notes |
|---|---|---|
| <ID> | ✅ Done | <one line + link to its per-slice note> |
```

---

## Risk Register (`docs/RISKS.md`)

Referenced in `coding-standards.md` Phase 10 (Documentation Update) and `methodology.md` Phase 16 (Project Completion).

```markdown
# Risk Register

Updated only when a meaningful technical or business risk surfaces — not a
restatement of ADRs (which record decisions already made). Entries are closed
out in place, not deleted, so the history of what was once a risk is kept.

| ID | Risk | Discovered | Likelihood | Impact | Mitigation / Owner | Status |
|---|---|---|---|---|---|---|
| R-1 | <concrete uncertainty, stated so it's falsifiable> | <slice/date> | Low/Med/High | Low/Med/High | <what's being done about it, and who owns that> | Open / Mitigated / Accepted / Closed |
```

---

## Final Quality Audit (`docs/QUALITY_AUDIT.md`)

Referenced in `methodology.md` Phase 16. Run once per release tier, after the last module closes, before the Release Readiness Checklist. Distinct from both: roadmap tracking asks "did we build the slices," this asks "is the *whole* now correct, clean, and what was actually asked for."

```markdown
# Final Quality Audit — <release/version>

## Does it do what was planned (traceability)
- [ ] Every PRD functional requirement traces to a shipped, verified slice —
      or is explicitly descoped, with sign-off recorded here
- [ ] Every business rule from Phase 2 has a named test that proves it
- [ ] Each core user workflow walked end-to-end as the user, against the
      PRD's own description of it (list the workflows and who walked them)
- [ ] Open assumptions recorded during discovery revisited — each confirmed,
      corrected, or consciously accepted

## Bugs
- [ ] Full regression suite green, run fresh (not from cache/memory of green)
- [ ] Every "Known Issues" entry across all module Feature Summaries either
      fixed, ticketed for a version, or explicitly accepted — none forgotten
- [ ] Error paths exercised, not just happy paths: invalid input, permission
      denied, provider down, empty data

## Security (whole-system, beyond per-slice reviews)
- [ ] Full authorization matrix verified: every role × every sensitive action
- [ ] Secrets audit: nothing in source history, bundles, or logs
- [ ] Security Baseline re-checked against the system as deployed
- [ ] Every external integration re-checked against the integration rules
      (signatures, idempotency, timeouts)

## Naming & consistency (cross-module)
- [ ] Names introduced early vs. late reconciled — one term per concept,
      per REPOSITORY_STANDARDS and the domain vocabulary (no user/member/
      account meaning the same thing in three modules)
- [ ] API surface consistent with docs/CONVENTIONS.md end to end
- [ ] UI conforms to docs/DESIGN.md tokens everywhere (no per-module drift)

## Redundancy & dead code (cross-module)
- [ ] Dead-code / unused-export tooling run project-wide; findings resolved
- [ ] Unused dependencies pruned from the tree
- [ ] Logic duplicated *between* modules consolidated (module-close passes
      only see one module at a time — this is where cross-module twins die)
- [ ] Zero TODO/FIXME remaining: each became a slice, a Risk entry, or was
      deleted deliberately
- [ ] Verification gates re-run green after all cleanup

Findings that can't be fixed now go to docs/RISKS.md or the roadmap — 
nothing discovered here is allowed to just evaporate.
```

---

## Release Readiness Checklist (`docs/RELEASE_CHECKLIST.md`)

Referenced in `methodology.md` Phase 16 (Project Completion).

```markdown
# Release Readiness Checklist

Checked in full before every production deployment. Distinct from roadmap
completion — this covers deployment-specific risk, not feature completeness.

## Configuration
- [ ] All required environment variables/secrets are set in the target environment
- [ ] No secret is present in source control or client-visible bundles

## Data
- [ ] Migrations tested against a production-like copy of the data
- [ ] Backup taken immediately before deployment; restore procedure tested, not just documented

## Reliability
- [ ] Rollback plan written down and tested at least once
- [ ] Monitoring/alerting is live and verified to actually fire, before real traffic arrives
- [ ] On-call/support path defined for the first 24–48 hours post-launch

## Security & Compliance (mandatory for Regulated/Enterprise-grade, per Phase 0)
- [ ] Security review/sign-off recorded, by name, not assumed
- [ ] `docs/RISKS.md` reviewed — nothing open and unaddressed that should block launch
- [ ] Dependency licenses and maintenance status re-checked for anything added since the last release

## Post-deploy verification
- [ ] Real payment/email/error-monitoring integrations verified against production, not just staging
- [ ] A smoke test of the critical path (the thing the business actually depends on) run against production itself
```

---

## Repository Standards (`docs/REPOSITORY_STANDARDS.md`)

Referenced in `methodology.md` Phase 3 (Product Planning) and `methodology.md` Phase 12 (Context & Memory Management).

```markdown
# Repository Standards

Set early, revised rarely, followed by every session without re-deciding it
each time.

## Naming
- Variables/functions: <convention, e.g. camelCase>
- Types/classes: <convention, e.g. PascalCase>
- Constants: <convention, e.g. SCREAMING_SNAKE_CASE>
- Database tables/columns: <convention, e.g. snake_case, plural table names>

## File & folder conventions
- File naming: <convention>
- Folder structure: <where domain logic lives vs. framework-mandated locations>
- Import ordering: <e.g. external packages, then internal aliases, then relative>

## Code style
- Comments policy: <e.g. explain why, not what; no restating the code in prose>
- Error message style: <e.g. always actionable, never a bare stack trace to the user>
- Logging style: <levels used, what must never be logged (secrets, PII)>

## Git
- Commit message format: <e.g. "<SLICE-ID>: summary" + body explaining why>
- Branch naming: <convention, if used>
```

---

## System Conventions (`docs/CONVENTIONS.md`)

Referenced in `methodology.md` Phase 3. Repository Standards covers how code *looks*; this covers how the system *behaves* — the things that silently drift between sessions unless decided once. Set in Phase 3, revised rarely, with sign-off.

```markdown
# System Conventions

How this system behaves, everywhere, decided once. A slice that needs to
deviate flags it (Phase 14) — it doesn't quietly invent a variant.

## API responses
- Success envelope: <exact shape, e.g. { data, meta }>
- Error envelope: <exact shape, e.g. { error: { code, message, fields? } }>
- Error codes: <the named codes and when each is used>
- Validation errors: <shape, e.g. per-field messages under fields>
- Pagination: <style (cursor/offset), parameter names, response metadata>
- Versioning: <strategy, or "none until first external consumer — recorded here">

## Data representation
- Dates/times: stored and transmitted as <e.g. UTC ISO-8601>; rendered in
  <user's timezone / fixed business timezone> at the edge only
- Money: <e.g. integer minor units (cents) + ISO currency code — never floats>
- IDs: <format and generation strategy>
- Null vs. absent: <the convention, so clients aren't guessing>

## Logging
- Format: <structured (JSON) with these standard fields: timestamp, level,
  request id, actor, event>
- Levels: <when each is used>
- Never logged: secrets, credentials, tokens, full PII, payment data —
  <plus this project's specific additions>
- Request correlation: <how a request is traced across layers>

## Background work & retries (if applicable)
- Queue/job conventions: <idempotency, retry policy, dead-letter handling>
```

---

## Security Baseline (`docs/SECURITY_BASELINE.md`)

Referenced in `methodology.md` Phases 3 and 6, and `coding-standards.md` Phase 9. This is the standing checklist per-slice "Security Checklist deltas" are deltas *against*. Seed it from Phase 1's quality/security bar; tighten per the Phase 0 grade.

```markdown
# Security Baseline

Every slice's security review checks these. A slice's "Security Checklist
delta" adds slice-specific items; "N/A beyond baseline" still means this
baseline was checked.

## Every slice
- [ ] Every mutation re-checks authorization server-side (never trust the
      client having hidden a button)
- [ ] All input crossing a trust boundary is validated server-side (type,
      length, range, allowed values)
- [ ] Queries are parameterized / ORM-safe — no string-built SQL
- [ ] Output is encoded for its context (XSS); rich text is sanitized
- [ ] No secret, credential, or key in source control, client bundles, or logs
- [ ] Errors shown to users leak no internals (stack traces, SQL, paths)
- [ ] New config/secrets follow the env-handling rules (Phase 8)

## Auth & session slices (additionally)
- [ ] Passwords hashed with a current, salted algorithm
- [ ] Auth endpoints rate-limited; lockout thresholds per the PRD's NFRs
- [ ] Session/token expiry, rotation, and revocation per the PRD's NFRs
- [ ] Privilege changes and sensitive actions audit-logged (per the Phase 1
      bar — launch-blocking for regulated grades)

## Integration slices (additionally)
- [ ] Incoming webhooks verify signatures; handlers are idempotent
- [ ] Third-party credentials are least-privilege and environment-separated

## Project-specific additions
- [ ] <from Phase 1's threat model and compliance answers>
```

---

## Performance Baseline & Budgets (`docs/PERFORMANCE_BASELINE.md`)

Referenced in `methodology.md` Phases 3 and 6, and `coding-standards.md` Phase 9. Budgets are numbers so the Phase 9 performance pass is a comparison, not an impression. Set them realistically in Phase 3 (informed by the Phase 1 device/network floor) and revisit deliberately, not per-slice.

```markdown
# Performance Baseline & Budgets

## Standing checks (every slice)
- [ ] No N+1 query patterns introduced (verified by inspecting actual queries,
      not assumed)
- [ ] Queries on hot paths hit indexes; each new index's reason is recorded
- [ ] Unbounded lists are paginated — no "fetch all" that grows with the data
- [ ] Payloads carry what the consumer needs, not entire entities by habit
- [ ] Heavy work off the request path (jobs/queues) where latency matters
- [ ] UI slices: skeletons reserve space; images sized; animation rules per
      frontend-design.md

## Budgets (numbers, checked when a slice plausibly moves them)
| Metric | Budget | Measured how |
|---|---|---|
| Page load (LCP) on the floor device/network | < <e.g. 2.5s throttled 4G> | <tool> |
| Initial JS bundle (gzipped) | < <e.g. 200KB> | <build output> |
| API latency, p95, hot endpoints | < <e.g. 300ms> | <tool/logs> |
| Slowest acceptable query | < <e.g. 100ms, or recorded reason> | <EXPLAIN/logs> |
| <project-specific: search, report generation, import throughput…> | | |

Budget misses are Phase 14 conflicts: flagged and decided (optimize, or
consciously revise the budget) — never silently absorbed.
```

---

## Design System (`docs/DESIGN.md`)

Referenced throughout `frontend-design.md` (Book 4) and produced in Phase 3 for any project with a UI. Set once in design discovery, revised rarely, only with explicit sign-off — every UI slice follows it without re-deciding it.

```markdown
# Design System

The approved design direction. UI slices follow this without re-deciding it;
per-slice style inventions are source-of-truth drift (see frontend-design.md).

## Direction
- Design concept: <one or two sentences — the dominant content shape, the
  domain's own visual vocabulary, and the one visual idea a first-time viewer
  should get in three seconds. Derived from the project itself, not a style
  preference; see frontend-design.md's Design Discovery>
- Project fact sheet: <the specific facts mined in discovery that design
  decisions cite — name/place/products/story/tone/artifacts; see the
  uniqueness guarantee in frontend-design.md>
- Signature geometry motif: <the one shape derived from a project artifact,
  and where it's carried — radii, dividers, clips, icons>
- Chosen direction: <bento grid / editorial / dense console / minimal / reference-derived>
- Chosen by: <who>, on <date> — from options offered in design discovery, not defaulted
- Reference site(s), if any: <URL> — measured in a real browser on <date>, values below
- Users of this UI: <who, how often, skill level>
- Device/network floor: <e.g. mid-range Android on 3G — this is a design input>

## Inspiration research (if live showcase browsing was run)
- Sources visited: <URLs + date, e.g. awwwards tag pages and the live sites
  behind them>
- Techniques adopted: <one line each — technique, source, and the project
  fact that justifies adopting it; no single source dominating>

## Uniqueness audit (run before sign-off; see frontend-design.md)
- Would this read at home in another AI-generated site? <no, because …>
- The one thing a visitor would describe to a friend: <…>
- Would a different team with this workflow land here? <no, because these
  decisions trace to these project facts: …>

## Cinematic direction (see frontend-design.md's Cinematic Direction)
- Storyboard: <the page arc in three lines — hook / development / resolution,
  and each section's named job in the story>
- Camera grammar: <dolly / held shots with cuts / dissolves — one per page>
- Blocking notes: <where key elements enter from, and the reading path that
  motion follows>
- Depth layers: <what lives in background / midground / foreground; where
  elements overlap across layers>
- Containment rule applied: <which content is genuinely enumerable and gets
  visible containers; everything else composed on the canvas>

## 3D pipeline (only if the concept earns a 3D scene)
- Scene and its narrative job: <what it shows and why it's a "moment">
- Asset source: <authored in Blender (via Blender MCP if configured) from
  <project artifact> / other — never a stock ornament>
- Delivery: <GLB + Draco/meshopt, lazy-loaded off critical path, DPR cap,
  offscreen pause, reduced-motion poster fallback>
- Budgets: <bundle size and frame time, in PERFORMANCE_BASELINE.md>

## Design checkpoint log (append per major surface; see frontend-design.md)
- <surface> — shown <date>, feedback: <what the user said>, adaptation:
  <what changed>, standing preference carried forward: <e.g. tighter spacing,
  calmer motion — applies to all later sections without re-asking>

## Layout system
- Container max-width (centered content well): <px>
- Grid: <columns> columns, <px> gutter, <px> outer margins
- Column math: (container − (columns−1)×gutter) / columns = <px> per column
- Outer margin (viewport-anchored chrome — nav/footer/full-bleed sections):
  <px per breakpoint, e.g. 24px mobile / 40–64px desktop> — distinct from the
  content well's max-width; audited at the desktop breakpoint specifically
  (see frontend-design.md's Full-bleed chrome vs. the centered content well)
- Breakpoints: <list> — and the re-flow decision at each (what the composition
  becomes, and the deliberate stack order on mobile)

## Color system
- Distribution rule: <e.g. 60-30-10 / 90-10 / 40-30-20-10 / 70-20-10> — chosen
  because <reasoning tied to the design concept, see frontend-design.md's
  Color System>
- Harmony rule: <monochromatic / analogous / complementary / split-complementary
  / triadic> — base hue(s): <values>
- Role table (light theme): Background, Surface, Border/Divider, Text-primary,
  Text-secondary, Text-disabled, Accent-primary, Accent-secondary (if any),
  Semantic (success/warning/danger/info) — <values per role>
- Role table (dark theme, if supported): <re-derived values per role, not a
  naive inversion — see frontend-design.md>
- Contrast pairs checked: <which pairings, against WCAG AA, and result>

## Typography
- Families: <display/heading family, body family, monospace if used> — chosen
  because <reasoning tied to dominant reading mode / domain, see
  frontend-design.md's Typography System>
- Scale ratio: <e.g. minor third 1.2 / major third 1.25 / perfect fourth 1.333
  / golden ratio 1.618> from a <px> base
- Scale table: <display / h1 / h2 / h3 / body / caption — size, weight,
  line-height per tier, line-height tightening at larger sizes>
- Pairing rationale: <contrasting structure / superfamily — why these two
  faces work together>
- Loading strategy: <self-hosted / font-display: swap + metric-matched
  fallback, subsetted weights>

## Tokens
- Spacing scale: <base unit and steps, e.g. 4 / 8 / 16 / 24 / 32 / 48 / 64> —
  every container's padding/gap is one of these steps, no invented values
  (see frontend-design.md's Geometry & Spatial Composition)
- Radii: <values, and where each applies>
- Motion tokens: <named easing curves, e.g. ease-out-standard:
  cubic-bezier(0.16, 1, 0.3, 1) for entrances, ease-in-out-standard:
  cubic-bezier(0.65, 0, 0.35, 1) for moves; plus one spring config for
  interactive elements, e.g. stiffness 300 / damping 30 / mass 1> — standing
  rules: transform/opacity only, prefers-reduced-motion respected
- Animation library: <CSS only / Framer Motion / GSAP> — per the Decision Log
  entry of <date>; lazy-loaded, never in the first-paint path

## Preloader
- Decision: <exists, and what real readiness signal it gates on (fonts, image/
  video decode, 3D asset load) — or "none, deliberately, because load is
  already fast">
- Visual continuity: <shares the hero's background color/decorative
  vocabulary, per frontend-design.md's Preloaders section>
- Handoff: <how the preloader's exit and the intro's entrance are one timeline>

## Page transitions (if this project is an SPA)
- Pattern: <curtain wipe / shared-element FLIP morph / cross-fade / none> —
  chosen because <reasoning tied to the design concept>
- Cost check: <confirmed affordable at real navigation frequency, not just
  on first click>

## First-visit intro
- Decision: <pattern chosen (e.g. split-text headline build, masked hero
  reveal, structural frame-in) and why it fits the design concept above — or
  "none, deliberately, because <reason>">
- Total duration: <ms, ceiling ~600–1200ms>
- Runs once per session: <mechanism, e.g. sessionStorage flag>
- Skip path: <what input short-circuits it>
- Reduced-motion behavior: <collapses straight to settled state>

## Component micro-specs (running list)
- Dropdown/select: <link to the anatomy in frontend-design.md's Component
  Micro-Design, plus any project-specific deviation and why>
- Searchable comboboxes: <which dropdowns cross the search threshold, the
  threshold used, and why — see frontend-design.md's "When the list gets long">
- <other recurring small component, if its spec deviates from the book's
  default anatomy — most won't need an entry here>

## Signature interaction (if any)
- Decision: <what it is, one or two sentences — or "none, deliberately,
  because <reason>">
- Why this project's concept earns it: <tie to the design concept above>
- Construction approach: <pointer-driven reveal / morphing material mask /
  other — see frontend-design.md's One signature interaction>
- Touch fallback: <substitute interaction, or "omitted on touch, deliberately">
- Reduced-motion fallback: <collapses to state>

## Shape vocabulary (only if this project uses cut/clipped card shapes)
- Docking notch/blister construction: <controls it's sized for, clearance used>
- Chamfer ratio: <e.g. ~12% of card's shorter side>
- Scatter pattern grid unit + opacity falloff: <values>
- Organic blob lobe proportions: <values, or "not used">
- Source: <derived from a supplied asset kit at <path/URL>, re-authored at this
  project's own sizes — not copied verbatim, per frontend-design.md>
- Responsive technique: <inline SVG behind content (default) / clip-path with
  resize recompute — and why>

## Workflow patterns in force
- Quick-add on reference dropdowns: <where it applies>
- Bulk add: <which entry screens offer it, and via what (inline table / CSV)>
- Page-hops budget: <each core task and the max screens it may take>
- Draft protection: <which forms, via what mechanism>

## Screens designed (running list)
- <screen> — <one line: its task, its layout, link to slice notes>
```

---

## Definition of Ready (quick checklist)

Referenced in `coding-standards.md` Phase 8 (Implementation).

```markdown
A slice is Ready when all of the following are checked:

- [ ] Dependencies are actually complete, not "should be fine"
- [ ] Acceptance criteria are written down
- [ ] Named test cases are defined
- [ ] Security Checklist delta against docs/SECURITY_BASELINE.md is written (or "N/A beyond baseline")
- [ ] Performance Checklist delta against docs/PERFORMANCE_BASELINE.md is written (or "N/A beyond baseline")
- [ ] Files likely to change are identified
- [ ] Required agent role(s)/context are identified

If anything above is unchecked, resolve it before writing code.
```

---

## Project Bootstrap Checklist (`PROJECT_BOOTSTRAP.md`)

Referenced in `coding-standards.md`'s Project Bootstrap Checklist section.

```markdown
# Project Bootstrap Checklist

Worked through once, when the repository is created, before Phase 0.

- [ ] Git initialized
- [ ] .gitignore configured
- [ ] LICENSE created
- [ ] README created
- [ ] Formatter configured
- [ ] Linter configured
- [ ] Editor settings configured (e.g. .editorconfig)
- [ ] Commit hooks configured (lint/format/test gates)
- [ ] CI configured (build, lint, type-check, test on every push/PR)
- [ ] Dead-code & unused-dependency detection configured (fits the stack; run at every module-close hygiene pass and in the Final Quality Audit)
- [ ] Issue templates configured
- [ ] Pull request template configured
- [ ] Automated dependency updates configured (Dependabot/Renovate or equivalent)
- [ ] CODEOWNERS configured
- [ ] Security Policy configured (how to report a vulnerability)
- [ ] CONTRIBUTING guidance configured
- [ ] docs/ folder structure created

For a Prototype (Phase 0 classification), CODEOWNERS/Security Policy/CONTRIBUTING
may be skipped if there's no team yet — but still configure the linter/formatter
and initialize Git properly. For Regulated/Enterprise-Grade, treat every item as
mandatory.
```
