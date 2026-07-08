---
name: build-web-app
description: Technology-agnostic, documentation-driven workflow for building any web application (or other software project) with an AI coding assistant, from initial idea through production release. Covers project classification, discovery, requirements, architecture, module/slice decomposition, the per-slice implement-verify-document-commit loop, context/memory management across sessions, anti-hallucination discipline, recovery from stuck implementations, and release readiness. Use when starting a new app/project, planning or architecting a system, deciding what to build next, implementing or verifying a development slice, writing project documentation (PRD, architecture doc, roadmap, ADRs), or preparing for a production deployment.
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

This skill is a navigator over three detailed reference books. Read this file first, then load
only the reference file(s) the current moment actually needs — loading all three for a small
task defeats the point of the workflow's own context-management principle.

| Reference | Load it when… |
|---|---|
| `references/methodology.md` (Book 1, Phases 0–6, 12, 14, 16) | Starting a new project, running discovery, writing planning docs (PRD/Architecture/Modules/Slices/Roadmap), reviewing architecture, deciding what's next, resolving a plan-vs-reality conflict, or closing out the project. |
| `references/coding-standards.md` (Book 2, Phases 7–11, 13, 15) | Implementing, verifying, documenting, or committing one specific slice — the file loaded on essentially every working session once planning is done. |
| `references/templates.md` (Book 3) | Creating one of the standard project documents for the first time (`NEXT_SESSION.md`, per-slice note, module Feature Summary, ADR, Decision Log, Risk Register, Release Checklist, Repository Standards, Definition of Ready, Project Bootstrap Checklist). |

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
- **About to write code for a specific slice**: go straight to `coding-standards.md` Phase 8 (Definition of Ready, then the implementation sequence).
- **A slice just got implemented**: `coding-standards.md` Phases 9–11 (verify, document, commit).
- **Something in the plan doesn't match reality, or implementation is stuck**: `methodology.md` Phase 14 (plan-vs-reality conflicts) and `coding-standards.md` Phase 15 (recovery procedure).
- **Preparing to ship**: `methodology.md` Phase 16 plus `templates.md`'s Release Readiness Checklist.

## Non-negotiable habits (apply regardless of which phase is active)

- **One slice at a time, one responsibility per slice.** Never implement multiple slices together, even related ones — "I'm already in this file" is exactly how scope drift starts.
- **Documentation before implementation, and one source of truth.** Once PRD/Architecture/Modules/Slices are approved, nothing — no later document, no chat conversation — silently contradicts them. Conflicts get flagged and resolved explicitly (`methodology.md` Phase 14), never quietly patched away.
- **Disclose necessary-but-unlisted work.** If a slice needs infrastructure or a fix outside its stated scope, say so before doing it — don't silently add it and don't silently skip it.
- **Verify empirically, not by assertion.** Passing lint/types/unit tests is necessary, not sufficient — actually run the app and drive the feature for anything with a runtime surface. This catches bugs the other checks structurally cannot.
- **Treat every library/framework behavior claim as unverified until checked**, especially anything version-specific — training data skews toward what was historically common, not what's current in this project's actual dependency versions.
- **A slice must be Ready before it starts, and Done only once it's actually verified** — both gates matter; skipping the entry gate is how implementation starts on missing information.
- **When verification keeps failing, stop and re-read the architecture/module docs before trying another fix** — repeated failure is information about the plan, not just an obstacle to brute-force past.
- **Classify the project's grade (Phase 0) before discovery starts** — a prototype, an internal tool, and a system holding real payment/health data need genuinely different rigor, even with similar feature lists.

The full Golden Rules list (shared by Books 1 and 2) is in `references/methodology.md`, bottom section.
