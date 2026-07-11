---
name: build-web-app
description: Technology-agnostic, documentation-driven workflow for building any web application (or other software project) with an AI coding assistant, from initial idea through production release. Covers project classification, discovery, requirements, architecture, module/slice decomposition, the per-slice implement-verify-document-commit loop, context/memory management across sessions, anti-hallucination discipline, recovery from stuck implementations, release readiness, multi-agent orchestration (when to fan out to parallel subagents, worktree isolation, reconciliation), frontend experience design (design discovery that derives a project-specific design concept instead of a generic template, a uniqueness guarantee — every design decision cites a project-specific fact and a sameness audit gates sign-off, live inspiration research browsing awwwards-style showcases by concept and extracting techniques from multiple live sites, workflow-first screens, bento grids and other layout systems including shaped/clipped/docked card construction, color systems — distribution rules like 60-30-10 and named harmony rules chosen per project rather than a default palette, typography systems — typeface and type-scale-ratio choices derived from the content's reading mode, calculated container/spacing geometry, small-component anatomy like dropdowns and buttons, choreographed motion — easing/spring tokens, parent/child stagger, first-visit intros, restrained scroll-triggered animation — and low-bandwidth performance, UI verification), and system design & data-structure selection (access-pattern-first engineering — matching indexes, caches, queues, bloom filters, sorted sets, and fan-out/caching/rate-limiting patterns to the question the code asks, plus the industrial caching and resilience tier — the browser→CDN→edge→app→DB caching ladder, per-entity invalidation, stampede protection, stale-while-revalidate, TTL jitter, hot-key handling, read-your-own-writes, dataloader batching, connection pooling, backpressure, load shedding, graceful degradation — all calibrated to project grade). Use when starting a new app/project, planning or architecting a system, deciding what to build next, implementing or verifying a development slice, designing or building UI screens/layouts/animations/components, deciding whether to parallelize work across agents, choosing how a feature stores/queries/caches data, diagnosing a slow query or page, scaling a feature, writing project documentation (PRD, architecture doc, roadmap, ADRs, design system doc), or preparing for a production deployment.
---

# Build Web App — AI-First Development Workflow

A complete, technology-agnostic methodology for building software (web apps, SaaS, internal
tools, regulated systems) with an AI coding assistant. It exists to prevent five recurring
failure modes: **context loss** across sessions, **hallucinated** API/library behavior,
**scope drift**, **silent rework** of already-decided questions, and **unverified completion**
(tests pass but nobody actually ran the thing).

Core idea: **documentation is not an end-of-project artifact — it's the mechanism that makes
long, multi-session AI-assisted work possible.** The repo itself should let a session with zero
memory of prior conversations pick up exactly where the last one left off.

This skill is a navigator over five detailed reference books. Read this file first, then load
only the reference file(s) the current moment actually needs — loading all five for a small
task defeats the point of the workflow's own context-management principle.

| Reference | Load it when… |
|---|---|
| `references/methodology.md` (Book 1, Phases 0–6, 12, 14, 16) | Starting a new project, running discovery, writing planning docs (PRD/Architecture/Modules/Slices/Roadmap), reviewing architecture, deciding what's next, resolving a plan-vs-reality conflict, or closing out the project. |
| `references/coding-standards.md` (Book 2, Phases 7–11, 13, 15) | Implementing, verifying, documenting, or committing one specific slice — the file loaded on essentially every working session once planning is done. |
| `references/frontend-design.md` (Book 4, extends Phases 1, 3, 9) | Anything UI: running design discovery (including deriving the project's own design concept and the uniqueness guarantee — every decision cites a project fact, audited before sign-off — never a template default), running live inspiration research on showcase sites (Awwwards etc.) matched to the concept, choosing a layout system (bento grid or otherwise, including shaped/docked card construction), picking a color system (a stated distribution ratio like 60-30-10 and a named harmony rule, never hex-by-hex) and a typography system (typeface/scale ratio derived from the content's reading mode), extracting a reference site the user likes, computing container/spacing geometry, designing a small component (dropdowns, buttons, tooltips), designing a screen or admin workflow (quick-add, bulk-add), choreographing motion (parent/child stagger, a first-visit intro, scroll-triggered moments), or verifying a UI slice. Skip it for pure backend/schema slices. |
| `references/system-design.md` (Book 5, extends Phases 2–4, 8, 9) | Designing how any feature stores, queries, or caches data; writing the Architecture Document's Data & Access Patterns section; choosing an index/cache/queue/search/rate-limit/feed strategy; reviewing scalability in Phase 4; a query, count, or page is slow; or a new infrastructure component (cache, queue, search engine) is being proposed. |
| `references/templates.md` (Book 3) | Creating one of the standard project documents for the first time (`NEXT_SESSION.md`, per-slice note, module Feature Summary, ADR, Decision Log, Risk Register, Release Checklist, Repository Standards, Design System doc, Definition of Ready, Project Bootstrap Checklist). |

## The full lifecycle

```
Bootstrap → Classify → Discovery → Requirements → Planning Docs → Architecture Review → Modules → Slices
  (once)     (Ph.0)     (Ph.1)       (Ph.2)          (Ph.3)             (Ph.4)          (Ph.5)   (Ph.6)
                                                                                                    │
                                     ┌──── one loop, per slice, forever ────────────────────────────┐
                                     │ Read docs → Ready check → Plan → Approve →                    │
                                     │ Implement → Verify → Document → Commit → Update "what's next"  │
                                     └──────────────────────────────────────────────────────────────┘
                                                                                                    │
                                                                                                    ▼
                                                                                      Project Completion (Ph.16)
```

## Where to start

- **Brand-new project, nothing built yet**: run the Project Bootstrap Checklist (`coding-standards.md`), then `methodology.md` Phase 0 (classify: Prototype / Internal Tool / Production SaaS / Regulated-Enterprise — this calibrates everything downstream), then Phase 1 discovery.
- **Picking up an existing project this session for the first time**: read `NEXT_SESSION.md` in the target repo (if it exists) — it names the exact narrow reading order for the next slice. If no such file exists yet, read `methodology.md` Phase 12's cold-start reading order.
- **About to write code for a specific slice**: go straight to `coding-standards.md` Phase 8 (Definition of Ready, then the implementation sequence). If the slice has a UI surface, also load `frontend-design.md`.
- **About to design any screen, layout, container, small component, or animation — or the user mentioned a style (bento grid, a reference site) or the UI feels generic/janky/stucky**: `frontend-design.md`, starting from its Design Discovery section. The visual direction, and the design concept it's derived from, are asked for and recorded in `docs/DESIGN.md`, never defaulted.
- **Considering dispatching parallel subagents for a task**: `coding-standards.md` Phase 7 — fan out only when sub-tasks are genuinely independent and can be isolated (separate worktrees for anything that writes code); the core implement-verify-document-commit loop for one slice stays single-threaded.
- **Deciding how a feature stores/queries data, planning the architecture's data layer, or something is slow (a query, a count, a page)**: `references/system-design.md` — run its five-question access-pattern interview, pick from the problem-shape tables, and respect its grade dial (an index solves most of it; exotic structures need a measurement first).
- **A slice just got implemented**: `coding-standards.md` Phases 9–11 (verify, document, commit). If it touched storage, also run `references/system-design.md`'s verification step (seeded volume + query plan against the performance budget).
- **Something in the plan doesn't match reality, or implementation is stuck**: `methodology.md` Phase 14 (plan-vs-reality conflicts) and `coding-standards.md` Phase 15 (recovery procedure).
- **Preparing to ship**: `methodology.md` Phase 16 plus `templates.md`'s Final Quality Audit (PRD traceability, whole-codebase bug/security/naming/dead-code audit) and Release Readiness Checklist, in that order.

## Non-negotiable habits (apply regardless of which phase is active)

- **One slice at a time, one responsibility per slice.** Never implement multiple slices together, even related ones — "I'm already in this file" is exactly how scope drift starts.
- **Documentation before implementation, and one source of truth.** Once PRD/Architecture/Modules/Slices are approved, nothing — no later document, no chat conversation — silently contradicts them. Conflicts get flagged and resolved explicitly (`methodology.md` Phase 14), never quietly patched away.
- **Disclose necessary-but-unlisted work.** If a slice needs infrastructure or a fix outside its stated scope, say so before doing it — don't silently add it and don't silently skip it.
- **Verify empirically, not by assertion.** Passing lint/types/unit tests is necessary, not sufficient — actually run the app and drive the feature for anything with a runtime surface. This catches bugs the other checks structurally cannot.
- **Treat every library/framework behavior claim as unverified until checked**, especially anything version-specific — training data skews toward what was historically common, not what's current in this project's actual dependency versions.
- **A slice must be Ready before it starts, and Done only once it's actually verified** — both gates matter; skipping the entry gate is how implementation starts on missing information.
- **When verification keeps failing, stop and re-read the architecture/module docs before trying another fix** — repeated failure is information about the plan, not just an obstacle to brute-force past.
- **Classify the project's grade (Phase 0) before discovery starts** — a prototype, an internal tool, and a system holding real payment/health data need genuinely different rigor, even with similar feature lists.
- **Match the structure to the question — never brute-force what an index can answer.** The cost of answering a question should depend on the size of the answer, not the size of the data (Instagram doesn't scan its user table to say "wrong password" — one indexed lookup, one hash compare). Every hot query names its backing structure in the Architecture Document's Data & Access Patterns section (`references/system-design.md`); and the inverse holds too: no cache, queue, or search engine joins the stack without a measurement proving the plain database failed.
- **Never default the visual design — down to the smallest control.** The design direction is the user's decision, made in design discovery (`frontend-design.md`) from real options — bento grid, editorial, dense console, or a reference site inspected and measured in a real browser — and recorded in `docs/DESIGN.md`. The direction itself is derived from the project's own content and domain, not a borrowed template; every recurring container runs through the Container Calculation Procedure — content extremes, spacing-scale padding/gaps, optical alignment, viewport fit — before it's Ready, never sized or nudged by feel; color and type follow a stated distribution/harmony rule and scale ratio, not habit; a nav bar hugs the true viewport edge instead of being stranded in the same centered gutter as body copy; a long dropdown gets search, a short one doesn't, decided per list; motion is choreographed (parent/child timelines, an intentional first-visit intro, at most one project-earned signature interaction) rather than bolted on piece by piece. Screens are designed around the user's task (quick-add, bulk-add, one task per screen), animations follow the transform/opacity rule, and UI slices are verified at multiple viewports on a throttled network.
- **Design uniqueness is specificity, not randomness.** Every major design decision cites the project-specific fact that produced it (the uniqueness guarantee, `frontend-design.md`); a signature geometry motif is derived from the project's own artifacts; and the sameness audit gates DESIGN.md sign-off — if the design would read at home in any other AI-built site, or a different team would plausibly land on it, it isn't done. When browser tooling is available, run live inspiration research on showcases (Awwwards and peers) by *concept*, extracting named techniques from multiple live sites — never cloning one.
- **Fan out to parallel agents only when the work is genuinely independent.** Isolate any agent that writes code in its own worktree, brief it with full context since it shares none of yours, and re-verify its output yourself rather than taking its word for it (`coding-standards.md` Phase 7). A single slice's implement-verify-document-commit loop is never split across agents.

The full Golden Rules list (shared by Books 1 and 2) is in `references/methodology.md`, bottom section.
