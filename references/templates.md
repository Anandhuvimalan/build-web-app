# Book 3 — Reference Templates

Part of a four-book set. Start at `SKILL.md` if you haven't already. This book is copy-pasteable templates for every document the other books reference. For the big-picture methodology (discovery, planning, architecture, context management), see `methodology.md`. For day-to-day implementation discipline (how a slice gets built, verified, committed), see `coding-standards.md`. For frontend experience and design, see `frontend-design.md`.

This is the book to load when actually creating one of these files for the first time in a project — otherwise it's reference material, not something to read start to finish.

---

## `NEXT_SESSION.md`

Referenced in `methodology.md` Phase 12 (Context & Memory Management).

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

Referenced in `methodology.md` Phase 12's two-tier documentation pattern.

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

## Design System (`docs/DESIGN.md`)

Referenced throughout `frontend-design.md` (Book 4) and produced in Phase 3 for any project with a UI. Set once in design discovery, revised rarely, only with explicit sign-off — every UI slice follows it without re-deciding it.

```markdown
# Design System

The approved design direction. UI slices follow this without re-deciding it;
per-slice style inventions are source-of-truth drift (see frontend-design.md).

## Direction
- Chosen direction: <bento grid / editorial / dense console / minimal / reference-derived>
- Chosen by: <who>, on <date> — from options offered in design discovery, not defaulted
- Reference site(s), if any: <URL> — measured in a real browser on <date>, values below
- Users of this UI: <who, how often, skill level>
- Device/network floor: <e.g. mid-range Android on 3G — this is a design input>

## Layout system
- Container max-width: <px>
- Grid: <columns> columns, <px> gutter, <px> outer margins
- Column math: (container − (columns−1)×gutter) / columns = <px> per column
- Breakpoints: <list> — and the re-flow decision at each (what the composition
  becomes, and the deliberate stack order on mobile)

## Tokens
- Spacing scale: <base unit and steps, e.g. 4 / 8 / 16 / 24 / 32 / 48 / 64>
- Type scale: <display / heading / body / caption — size, weight, line-height>
- Color roles: <background layers, surface, text hierarchy, accents — as roles>
- Radii: <values, and where each applies>
- Motion tokens: <2–3 duration+easing pairs, e.g. micro 150ms ease-out;
  entrance 250ms ease-out> — and the standing rules: transform/opacity only,
  prefers-reduced-motion respected
- Animation library: <CSS only / Framer Motion / GSAP> — per the Decision Log
  entry of <date>; lazy-loaded, never in the first-paint path

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
- [ ] Security Checklist delta is written (or "N/A beyond standard")
- [ ] Performance Checklist delta is written (or "N/A beyond standard")
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
