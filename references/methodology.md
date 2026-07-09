# Book 1 — AI Development Methodology

Part of a four-book set. Start at `SKILL.md` if you haven't already — it explains what's in each book and when to load which. This book covers the big-picture arc: understanding the business, planning the system, decomposing it, and closing the loop at release. For day-to-day implementation discipline (how a slice actually gets built, verified, committed), see `coding-standards.md`. For frontend experience and design (design discovery, layout systems, motion, UI verification), see `frontend-design.md`. For copy-pasteable document templates, see `templates.md`.

**Phase numbers are shared across all books, not restarted per book** — this book covers Phases 0–6, 12, 14, and 16. The gaps (7–11, 13, 15) live in `coding-standards.md`; `frontend-design.md` extends Phases 1, 3, and 9 for UI work rather than adding new phases.

---

## Purpose

This is a standard workflow for building any software project with an AI coding assistant — a web app, SaaS product, CRM, ERP, finance system, healthcare tool, or internal business tool. It is technology-agnostic: nothing here depends on a specific language, framework, or industry.

The workflow exists to solve five recurring failure modes of AI-assisted development:

- **Context loss** — a new session (or a long session that's compacted) forgets decisions already made and re-derives or contradicts them.
- **Hallucination** — the assistant assumes API behavior, library defaults, or business rules instead of checking them.
- **Scope drift** — "while I'm in there" edits spread a change across files that had nothing to do with the task.
- **Silent rework** — the same design question gets re-litigated in a later session because nothing recorded that it was already answered.
- **Unverified completion** — code that type-checks and passes unit tests but was never actually run, so a real bug ships anyway.

Everything in this three-book set is built around one idea: **documentation is not an artifact you produce at the end — it's the mechanism that makes long, multi-session AI-assisted work possible at all.** A project built this way should be resumable by a session with zero memory of any prior conversation, using only the files in the repo.

---

## Phase 0 — Project Classification

Before discovery starts, spend five minutes classifying the project into one working bucket. This single decision calibrates how deep everything downstream needs to go — how many rounds Phase 1's discovery loop needs, how strict Phase 9's verification gates are (`coding-standards.md`), how much Phase 10 documentation is warranted (`coding-standards.md`), and what Phase 16's release checklist actually requires. Skipping this doesn't avoid the decision — it just makes the decision by accident, usually by under-building for what the project turns out to need.

| Classification | Typical signal | Default posture |
|---|---|---|
| **Prototype** | Proving a concept, throwaway or heavily rewritten before real use, no real user data | Lightweight discovery, minimal verification gates, skip most of Phase 10/12's documentation overhead — optimize for speed of learning, not durability |
| **Internal Tool** | Used by employees only, not customer-facing, limited blast radius if wrong | Standard discovery and verification, but security/compliance questions can be lighter unless the tool touches sensitive internal data (payroll, HR, financials) |
| **Production SaaS / Consumer App** | Real external users, real data, reputational and revenue stakes | Full workflow as written: full discovery loop, full verification gates, full documentation discipline |
| **Regulated / Enterprise-Grade** | Payment data, health data, financial records, enterprise procurement/compliance requirements, contractual SLAs | Everything in Production, plus: mandatory security sign-off gate before release (Phase 16), compliance requirements captured as explicit testable NFRs (Phase 2), audit-trail coverage treated as launch-blocking, dependency vetting (Phase 8, `coding-standards.md`) enforced strictly, not loosely |

This is a **working hypothesis, not a final answer** — Phase 1's "Establishing the quality and security bar" section is where it gets properly interrogated and either confirmed or revised. If Phase 1 reveals the classification was wrong (a "simple internal tool" turns out to touch payroll data, say), update it there and let that revision flow through — don't silently keep building at the original, now-incorrect bar.

Record the classification explicitly, in one line, in the PRD's document control section — it should be the first thing a new session sees.

---

## Phase 1 — Project Discovery

Never begin coding immediately. Start by understanding the business, not the software.

### Discovery is a loop, not a form

Don't ask one round of questions and move on. Run discovery as an explicit loop:

1. **Ask broad, open questions first** — the problem, the users, the current workflow.
2. **Listen for what's vague, missing, or self-contradictory** in the answer.
3. **Drill into exactly that** with a specific, pointed follow-up — not a repeat of the same broad question.
4. **Repeat** until an answer round produces no new information and no new ambiguity.
5. **Play the whole picture back** — a short summary of the problem, goals, users, scope, and quality bar as you now understand it — and get explicit confirmation before moving to Phase 2. If anything in the playback gets corrected, that's a signal to loop again, not a one-line patch.

Discovery is done when you can restate the project back accurately without the other person correcting you — not when you've asked a fixed number of questions.

### What to identify

Through that loop, not assumption, identify:

- The business problem and why it's urgent now
- Business goals and how success will be measured
- Target users and every distinct user role
- The current (often manual/informal) workflow being replaced or improved
- Concrete pain points in that current workflow
- Explicit scope boundaries — what this project will **not** do
- The longer-term vision this fits into

If the person you're building for doesn't know an answer yet, record it as an open assumption rather than silently picking one — assumptions made here become load-bearing for every later document.

**Anti-pattern:** inferring a business rule from what "most apps like this do." If it wasn't stated, ask. **Anti-pattern:** treating a single pass through the question list as discovery. The loop only ends at the playback-and-confirm step above.

### Design discovery runs inside this loop

If the project has any user-facing surface, design discovery is part of this same Phase 1 loop, not a later styling pass: who operates the UI and how often, what visual direction they want (offered as real options — bento grid, editorial, dense console, minimal, or a reference site they admire — never defaulted), brand constraints, and the realistic device/network floor. If they name a reference site, inspect it in a real browser and extract concrete measurements rather than guessing at it. The full question list, the reference-extraction procedure, and the workflow-first screen principles live in `frontend-design.md`; the answers land in `docs/DESIGN.md` (Phase 3).

### Establishing the quality and security bar

This is the question people most often skip, and the one with the largest downstream cost when skipped: **what grade of software is this, actually?** A prototype, an internal MVP, and a system that will hold real customer payment data, health records, or financial transactions have almost nothing in common architecturally, even if their feature lists look similar. Ask this explicitly, early, and don't infer it from the domain — a "simple internal tool" can still carry serious compliance obligations, and a consumer app can be genuinely low-stakes.

This is where Phase 0's quick classification gets properly tested. Treat it as a hypothesis to confirm or overturn, not a settled fact — if these questions reveal it was wrong, correct it now, before an architecture gets built around the wrong assumption.

Ask, and keep drilling until each has a concrete answer, not a vibe:

- **What's actually at stake if this is breached, wrong, or down?** Money moving, health data, minors' data, legal liability, reputational damage, someone's physical safety, or genuinely none of the above?
- **What's the target grade**: throwaway prototype, internal tool, production consumer app, or regulated/industrial-grade system? Each implies a different default posture — don't let "production-grade" be assumed without saying what that means concretely for this project.
- **What data is sensitive**, and under which classification — PII, payment data (PCI DSS), health data (HIPAA), children's data (COPPA), financial records (SOX), EU personal data (GDPR)? Ask industry-by-industry, don't wait for the answer to be volunteered.
- **What's the realistic threat model?** Opportunistic bots and credential-stuffing, or a specifically motivated adversary (a competitor, an insider, a nation-state-adjacent actor)? This changes how much you invest in things like rate limiting, WAF rules, and secrets rotation.
- **What compliance regimes are already known to apply**, and which are aspirational for later (e.g., "we'll need SOC 2 before our first enterprise customer, but not at launch")?
- **What's the expected availability/reliability bar** — best-effort, or an SLA with real financial penalties?
- **Who is responsible for security sign-off**, and is there a required review/pen-test gate before this can go live?
- **What does "done" mean for auth specifically** — password-only is fine for many products; regulated data often mandates MFA, session-timeout policies, and audit trails as launch-blocking, not nice-to-have.

Record the answer as an explicit, named target (e.g., "V1 targets production-grade for a consumer app handling payment data via a PCI-compliant processor; not aiming for SOC 2 at launch, revisit before enterprise sales") in the PRD's non-functional requirements. Every later architecture decision — session strategy, logging, encryption at rest, rate limiting, audit trails — should trace back to this stated bar, not be invented ad hoc per module.

---

## Phase 2 — Requirement Discovery

Collect functional and non-functional requirements exhaustively before any planning document is written. Typical categories (adapt to the domain):

- Identity, authentication, authorization
- Core domain entities and their lifecycles (products, cases, patients, invoices, tickets — whatever the domain's nouns are)
- Payments/billing, if applicable
- Reporting and analytics
- Notifications and communication
- Security and compliance obligations (industry-specific: PCI, HIPAA, SOC2, GDPR, etc.) — expand the quality/security bar established in Phase 1 into concrete, testable controls here: password/session policy, rate limiting and lockout thresholds, encryption at rest and in transit, audit-log coverage (which actions must be logged, and for how long), secrets management, and who/what the authorization model must defend against
- Performance and scale expectations
- SEO/discoverability, if public-facing
- Integration points with external systems

For every category above, keep drilling the same way Phase 1 does: a vague answer ("it should be secure") is not a requirement — follow up until it's a testable statement. Write every business rule and every security control down as a testable statement before planning begins — "a customer may cancel an order only before it ships" is testable; "orders should be flexible" is not. "Passwords must be hashed with a current, salted algorithm and admin accounts must be rate-limited after 5 failed attempts per 15 minutes" is testable; "auth should be secure" is not.

---

## Phase 3 — Product Planning

Produce the following as durable, versioned documents (not chat history):

| Document | Answers |
|---|---|
| **Product Requirements Document (PRD)** | What are we building and why? What's explicitly out of scope? |
| **Architecture Document** | How is it built? What are the non-negotiable technical constraints? |
| **Module Breakdown** | What are the independent units of the system, and how do they depend on each other? |
| **Development Slices** | What is the smallest independently-shippable unit of work, module by module? |
| **Roadmap** | What order do slices get built in, and what's the critical path to a usable release? |
| **AI Development Rules** | What conventions and guardrails apply to every session, regardless of which slice? |
| **Implementation Workflow** | What exact stage sequence does every slice go through? |
| **Repository Standards** (`docs/REPOSITORY_STANDARDS.md`, see `coding-standards.md`) | What does correctly-styled code in this repo look like — naming, import ordering, folder/file conventions, error-message style, logging style, comments policy, commit message format? |
| **Design System** (`docs/DESIGN.md`, see `frontend-design.md`) — only if the project has a UI | What is the chosen visual direction and why? What are the layout system, tokens (spacing, type, color roles, radii, motion), per-breakpoint behavior, and workflow patterns (quick-add, bulk-add) every UI slice follows without re-deciding them? |
| **System Conventions** (`docs/CONVENTIONS.md`, template in `templates.md`) | How does the system *behave* consistently across sessions — API response/error envelope, pagination, dates/timezones/money representation, structured-logging shape and never-log list? Repository Standards covers how code *looks*; this covers how it *acts*. |
| **Security Baseline** (`docs/SECURITY_BASELINE.md`, template in `templates.md`) | The standing security checks every slice is reviewed against. Per-slice "Security Checklist deltas" (Phase 6) are deltas *against this document*. |
| **Performance Baseline & Budgets** (`docs/PERFORMANCE_BASELINE.md`, template in `templates.md`) | The standing performance checks and the project's *numeric* budgets (page load, bundle size, API latency, query time). Per-slice "Performance Checklist deltas" are deltas against this; Phase 9's performance pass compares against these numbers, not vibes. |

These become **the project's single source of truth**. Once approved, no other document — and no chat conversation — should silently contradict them. If a downstream document needs to diverge from the PRD, the PRD is updated first, with a note explaining why, not quietly worked around.

No implementation begins before these exist and are approved by whoever owns the product decision.

---

## Phase 4 — Architecture Review

Before writing the first line of implementation code, stress-test the architecture document itself against:

- **Scalability** — does this hold up at 10x the expected initial load?
- **Security** — where is the actual trust boundary, and does every mutation re-check it?
- **Performance** — what's the expected hot path, and is it designed for that?
- **Maintainability** — can a new session understand this subsystem from its own documentation alone?
- **Simplicity** — is there a simpler design that meets the same requirements? (Prefer it. Cleverness is a cost, not a feature.)
- **Future expansion** — does a known future requirement (e.g., multi-tenancy, a second region, a second currency) require a rewrite, or just an addition?

Revise the architecture document now. It is far cheaper than discovering the gap six modules in.

---

## Phase 5 — Module Decomposition

Split the system into modules that are each independently understandable without reading the others. A module is the right size when a new session can read its one section of the Module Breakdown, plus its own prior Feature Summary, and know everything needed to extend it.

Typical shape (adapt names to the domain):

- Core Platform / Data Infrastructure (always first — everything else depends on it)
- Identity & Access Management
- The domain's primary entities (Products, Cases, Patients, Listings, Policies…)
- Supporting domain concerns (Inventory, Scheduling, Documents…)
- Transaction/commitment flow (Cart & Checkout, Booking, Application Submission…)
- Fulfillment/lifecycle management (Orders, Case Management, Claims Processing…)
- Cross-cutting concerns (Notifications, Audit, Search, Analytics)
- Administration

For each module, document: purpose, in/out-of-scope boundary, dependencies on other modules, the tables/entities it owns, and which future module(s) depend on it. This dependency graph is what later lets you safely reorder work.

---

## Phase 6 — Development Slices

Split every module into slices small enough to implement, review, and document in one sitting. A good slice:

- Has one responsibility (one schema addition, one action, one page, one service function)
- Is independently testable — it has its own named test cases, not "tests will come later"
- Has a Security Checklist delta and a Performance Checklist delta — deltas *against the project's standing baselines* (`docs/SECURITY_BASELINE.md`, `docs/PERFORMANCE_BASELINE.md`, templates in `templates.md`), even if both say "N/A beyond baseline" — forcing the question is the point
- Has minimal dependencies, and those dependencies are stated explicitly
- Never spans two modules' worth of unrelated work

**Slice size rule:** if a slice cannot reasonably be implemented, reviewed, tested, documented, and committed within a single focused session (roughly 1–3 hours), split it before implementation begins. A slice that keeps growing mid-implementation should be paused and split, not pushed through as one oversized commit.

**Never implement multiple slices at once**, even when they feel related. "I'm already in this file" is not a reason to expand scope — it's exactly the moment scope drift happens.

### Right-sizing: slices are a risk instrument, not a ritual

Slicing is not free. Every slice carries a fixed overhead — the cold-start reading, the Ready check, verification, documentation, the commit — and if slices are cut too small, that overhead dominates and the process gets slower with no safety gained. The point of a slice is **the unit of verification and recovery**: small enough that when something breaks, the cause is localized to one change, and abandoning it costs one slice, not a week. Calibrate deliberately:

- **Size by risk, not uniformly.** Schema changes, auth/authorization, payments, and anything with concurrency implications get *small* slices — these are where a tangled failure is most expensive to unwind. Low-risk, repetitive work gets *bigger* slices: five similar CRUD endpoints, or one screen's set of similar states, can be **one slice with five named test cases** — splitting work that trivially rhymes into five ceremonies is over-slicing, not discipline.
- **One responsibility ≠ one tiny step.** The rule is one *responsibility* per slice. "The supplier CRUD endpoints" is one responsibility; "the CREATE endpoint" and "the DELETE endpoint" as separate slices is usually ceremony.
- **Signs of over-slicing** (merge slices): the overhead steps routinely take longer than the implementation itself, verification never catches anything because each change is trivial, and the per-slice notes read as near-duplicates.
- **Signs of under-slicing** (split slices): verification failures are hard to localize to a cause, diffs span many layers at once, the session exhausts its context mid-slice, or review becomes skimming because the diff is too big to actually read.
- **The Phase 0 grade sets the default**: a Prototype tolerates big slices and light gates — speed of learning is the goal; Production and Regulated projects bias small in the risky areas listed above and normal-sized elsewhere. Uniformly tiny slices everywhere is the accidental-decision version of calibration, same as skipping Phase 0 is.

---

## Phase 12 — Context & Memory Management

This is the phase that makes a project survivable across dozens of separate AI sessions. Chat history is not persistence — treat every session as if it starts with total amnesia, and make the repository itself carry the memory.

### The core artifacts

| File | Purpose | Update cadence |
|---|---|---|
| `PRD.md`, `ARCHITECTURE.md`, `MODULES.md`, `DEVELOPMENT_SLICES.md` | The stable source of truth | Rarely; only with explicit sign-off |
| `ROADMAP.md` (+ a Progress Log section) | Sequencing, critical path, and a glance at what's done | One line per completed slice |
| `CHANGELOG.md` | What changed, in human terms | One entry per completed slice |
| `docs/features/<slice-id>.md` | A lightweight note on exactly what one slice did | Once per slice |
| `docs/features/<module-name>.md` | The full, consolidated story of a module | Once per module, at its last slice |
| `NEXT_SESSION.md` | The single file a cold session reads to know what to do right now | Overwritten (not appended) every slice |
| `docs/adr/*.md` | Immutable records of real architectural decisions | Only when a real decision is made; never edited, only superseded |
| `docs/DECISIONS.md` | A running, lightweight log of small decisions too minor for an ADR | Appended whenever a small "we chose X because Y" decision is made |
| `docs/RISKS.md` | Open technical/business uncertainties not yet resolved | Only when a meaningful risk surfaces; entries closed out (not deleted) once resolved |
| `docs/RELEASE_CHECKLIST.md` | The final gate before a production deployment, separate from feature completeness | Checked immediately before each release (see Phase 16) |
| `docs/REPOSITORY_STANDARDS.md` | Naming, style, and convention rules every session follows without re-deciding them | Set early; revised rarely, only with explicit sign-off |
| `docs/DESIGN.md` (UI projects only, see `frontend-design.md`) | The approved design direction, tokens, layout system, and workflow patterns every UI slice follows | Set in Phase 3; revised rarely, only with explicit sign-off |
| `docs/CONVENTIONS.md` | System behavior conventions: API/error envelope, pagination, dates/money, logging shape | Set in Phase 3; revised rarely, only with explicit sign-off |
| `docs/SECURITY_BASELINE.md`, `docs/PERFORMANCE_BASELINE.md` | The standing checklists (and numeric budgets) that per-slice deltas and Phase 9 reviews run against | Set in Phase 3; budgets revisited only deliberately |
| `.env.example` + a config section in the docs | Every environment variable the system needs, documented in the same commit that introduces it | Updated per slice, whenever config is added (see Phase 8) |
| `PROJECT_BOOTSTRAP.md` | One-time repo setup checklist (CI, hooks, templates, security policy) | Once, at repo creation; not part of the ongoing per-slice cycle |

All templates for these files are in `templates.md`.

### The two-tier documentation pattern

Writing a full Feature Summary after *every single slice* creates too much documentation to read at the start of the next session, and most of it is redundant module-to-module. Writing one only at the *end* of a module means losing the specific story of each slice along the way. Use both, at different granularities:

- **Per-slice note** (`docs/features/<slice-id>.md`): three paragraphs, tops. What this slice did, any bug it caught, what the next slice needs to know. Written every time.
- **Per-module Feature Summary** (`docs/features/<module-name>.md`): the full picture — files touched by layer, database changes, APIs added, business rules implemented, testing completed, security/performance notes, known issues, what depends on this module now. Written once, when the module's last slice lands, consolidating its per-slice notes into one document a completely new session can read instead of replaying the whole module's history.

### The cold-start reading order

Define, and keep current, the exact order a new session should read documents in — and keep it *narrow*. A session picking up one slice of one module should not need to load every other module's Feature Summary. Typically: the PRD section relevant to this module → the architecture sections this slice touches → the module's entry in the Module Breakdown → the specific slice's entry in Development Slices → that module's prior Feature Summary (if any) → the specific per-slice notes for directly preceding slices → the specific role documents this slice needs (Phase 7, `coding-standards.md`).

### Standing decisions

When a real design tension gets resolved mid-project — a conflict between two earlier documents, or a genuine trade-off with no single obviously-correct answer — record the resolution as a **standing decision**, not just as an artifact of the commit that resolved it. `NEXT_SESSION.md` should carry a short "standing decisions this session should not re-litigate" list. This is what prevents the same question from being re-asked (and possibly re-answered differently) three modules later.

### What NOT to persist

- Chat transcripts or a narrative of the conversation — the code and the docs above are the record.
- Anything already derivable by reading the code (don't document "the button is blue," the component says so).
- Speculative future scope not yet approved — that belongs in the Roadmap as a future slice, not as a note.

---

## Phase 14 — Handling Conflicts Between Planning Docs and Reality

Plans made before implementation began will sometimes turn out to be wrong, incomplete, or internally inconsistent once real code exists. This is expected, not a failure of planning. Handle it explicitly:

- **Never silently patch a baseline document** (PRD, Architecture, Modules, Development Slices) to make a conflict disappear. Flag it, explain the conflict in concrete terms, and either get sign-off to change the baseline or resolve it at the implementation layer with the baseline left intact and the resolution recorded as a standing decision.
- **A slice's own named test case can itself be wrong** once more context exists (e.g., it references a table or route that a later, deliberately-resequenced plan hasn't built yet). When that happens: identify the gap, propose the smallest change that keeps the test's actual intent, and record why — don't quietly delete the test's requirement, and don't force a literal reading that reintroduces an already-rejected design.
- **Sequencing conflicts**: if a roadmap defines both a numeric ordering and a dependency-driven critical path, and they disagree about what's "next," the critical path wins — numeric ordering is a convenience label, not a dependency graph.
- **When a fix to one slice changes the meaning of an earlier slice's test** (e.g., loosening a strict check because a later, approved decision made the strict version impossible to satisfy), re-verify the earlier test's original intent still holds under the new logic before moving on — don't just make it pass.

This phase pairs closely with Phase 15 (Recovery Procedure, `coding-standards.md`) — that's the "what to do right now" operational counterpart to this phase's "how to think about it."

---

## Phase 16 — Project Completion

Roadmap completion and release readiness are two different gates — a project can have every planned slice done and still not be safe to deploy. Keep them separate.

**Roadmap completion:**

- Verify every roadmap item for the release tier is actually done, not just started.
- Run full regression testing, not just the newest slice's tests.
- Perform a dedicated security review across the whole surface, not just the last slice.
- Perform load/performance testing against realistic expected traffic.
- Finalize all documentation — confirm the module Feature Summaries are complete for every module, and the Changelog reads as a coherent history.
- Review `docs/RISKS.md`: every open risk is either closed, explicitly accepted by whoever owns that call, or is itself launch-blocking until resolved. Don't let it just go stale and unread.

**Release readiness (the final gate, checked separately, every release, not just the first one):**

Maintain a standing `docs/RELEASE_CHECKLIST.md` (template in `templates.md`), checked in full immediately before every production deployment — not folded into roadmap tracking, because it covers concerns the roadmap was never scoped to catch: environment variables and secrets present in the target environment, migrations tested against a production-like copy of the data, a concrete and tested rollback plan, monitoring/alerting actually live before traffic arrives, an on-call/support path defined, and (for regulated/enterprise-grade projects per Phase 0) explicit security sign-off recorded, not assumed.

Execute the production deployment only once both gates pass, then verify the specific things that only exist in production (real payment provider, real email delivery, real error monitoring) actually work, not just that the build succeeded.

---

## Golden Rules

- Documentation before implementation.
- One source of truth — and when something contradicts it, the source of truth wins until it's deliberately changed.
- One slice at a time.
- One responsibility per slice.
- Size slices by risk, not uniformly — small where failure is expensive (schema, auth, payments, concurrency), bigger for low-risk repetitive work; over-slicing is overhead, not discipline.
- Never rewrite working code unnecessarily.
- Never modify unrelated files.
- Disclose necessary but unlisted work — don't silently add it, and don't silently skip it because it wasn't named.
- Verify empirically before trusting an assumption, especially about framework/library behavior.
- Run the real thing before declaring a slice done — tests passing is necessary, not sufficient.
- Review before commit; document before the next slice.
- Keep AI context clean and intentional — read narrowly, load only what the current slice needs.
- When plans and reality conflict, flag it — don't silently patch the plan or silently ignore the conflict.
- Build software incrementally, not all at once — and make every increment resumable by someone (or some session) who wasn't there when it started.
- Classify the project's grade before discovery starts, and let that classification set the bar for everything after it.
- A new dependency is a liability until justified — check maintenance status and license before it's in the tree, not after.
- Roadmap completion and release readiness are different gates — passing one doesn't mean the other is satisfied.
- A slice must be Ready before it's implemented, not just Done when it's finished — the entry gate matters as much as the exit gate.
- When verification keeps failing, stop and re-read before trying another fix — repeated failure is information, not an obstacle to push through.
- Not every decision needs an ADR, but every decision worth re-litigating needs to be written down somewhere.
- Never default the visual design — the direction is asked for in design discovery, recorded in `docs/DESIGN.md`, and followed by every UI slice; screens are designed around the user's task, not the data model.

This list is shared by all four books — the other books link back here rather than repeating it.
