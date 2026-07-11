# Book 2 — AI Coding Standards

Part of a four-book set. Start at `SKILL.md` if you haven't already. This book covers day-to-day operational discipline: how a slice actually gets implemented, verified, documented, committed, and recovered when something goes wrong. For the big-picture arc (discovery, planning, architecture, modules, context management), see `methodology.md`. For frontend experience and design (design discovery, layout systems, motion, UI verification), see `frontend-design.md`. For copy-pasteable document templates, see `templates.md`.

**Phase numbers are shared across all books, not restarted per book** — this book covers Phases 7–11, 13, and 15, plus the (unnumbered) Project Bootstrap Checklist. The gaps (0–6, 12, 14, 16) live in `methodology.md`; `frontend-design.md` extends Phases 1, 3, and 9 for UI work.

This book is the one loaded most often, once per development slice, for the entire life of the project.

---

## Project Bootstrap Checklist

This happens once, before Phase 0 (`methodology.md`) — when the repository itself is created, not once the product thinking starts. It's deliberately unnumbered rather than "Phase -1": it's a one-time setup checklist, not a step in the repeating methodology, and giving it a phase number would suggest it recurs the way the numbered phases do.

Maintain it as `PROJECT_BOOTSTRAP.md` (or fold it into a README section) and work through it before Phase 0 begins:

```
□ Initialize Git
□ Configure .gitignore
□ Create LICENSE
□ Create README
□ Configure formatter
□ Configure linter
□ Configure editor settings (e.g. .editorconfig)
□ Configure commit hooks (e.g. pre-commit lint/format/test gates)
□ Configure CI (build, lint, type-check, test on every push/PR)
□ Configure dead-code & unused-dependency detection (e.g. knip/ts-prune/depcheck for JS/TS, vulture for Python — whatever fits the stack), run in the module-close hygiene pass (Phase 10)
□ Configure issue templates
□ Configure pull request template
□ Configure automated dependency updates (Dependabot/Renovate or equivalent)
□ Configure CODEOWNERS
□ Configure a Security Policy (how to report a vulnerability)
□ Configure CONTRIBUTING guidance
□ Create the docs/ folder structure this workflow expects
```

None of this is optional busywork — it's exactly the kind of thing that's expensive to retrofit later (a repo with six months of history and no commit hooks doesn't get them for free) and cheap to do on day one. For a Prototype (Phase 0 classification, `methodology.md`), a reduced version of this is fine — skip CODEOWNERS/Security Policy/CONTRIBUTING if there's no team yet, but still initialize Git properly and configure a linter/formatter from the start. For Regulated/Enterprise-Grade projects, treat the whole list as mandatory, not optional, especially the Security Policy and CODEOWNERS.

---

## Phase 7 — Multi-Agent Orchestration

A session with access to genuine parallel subagents (Claude Code's Agent tool, or an equivalent orchestration layer) can move faster than one thread working serially — but only if the fan-out is deliberate. Dispatching agents by reflex ("split this into five agents") produces conflicting edits, duplicated work, and integration debt that costs more than the parallelism saved. This phase is concrete rules for when and how to actually use parallel agents, plus the role-scoping discipline that makes each one effective whether or not it runs in parallel.

### Role scoping, whether solo or parallel

Whether one thread is doing role-specialized *thinking* or several agents are doing it in parallel, define per-role boundaries — Frontend, Backend, Database/DAL, API, Testing, Security, Performance, Documentation, Review — for each: what files it may touch, what files it must never touch, what it reads before starting, what it hands off when done. This turns "review every mutation for security" from a scavenger hunt into "read one folder." The Frontend role's reading list always includes `docs/DESIGN.md` and `frontend-design.md` (Book 4).

**Load only the roles the current slice actually needs.** A schema-only slice doesn't need the frontend role's context; a pure-UI slice doesn't need the database role's — every role's context loaded is tokens spent before real work starts, for a solo thread and doubly so for an agent you're dispatching. Some slices genuinely span roles (a slice that adds a table *and* the action that writes to it *and* the page that calls it) — say so explicitly rather than pretending it's single-role, and load all the roles it actually touches.

### When fan-out is worth it

Parallelize when the sub-tasks are **genuinely independent** — no sub-task's output changes what another needs to do, and no two sub-tasks write to the same files:

- **Independent research/investigation** — "how does the current codebase handle X," "what does library Y's current API look like," "what's the existing test coverage for Z" have no dependency on each other and can run as parallel read-only agents whose findings you synthesize yourself before deciding anything.
- **Independent slices in disjoint file sets** — two roadmap slices that touch no shared file (a backend endpoint slice and an unrelated frontend copy fix, say) can implement in parallel, each in its own agent, **only if given filesystem isolation** (a worktree per agent) so neither's uncommitted changes collide with the other's or with the main session's working tree.
- **Independent review passes** — a correctness review and a security review of the same diff are legitimately parallel, because review doesn't mutate anything; each agent reads the diff and reports findings without touching files.
- **Parallel design exploration** — when a genuinely open visual/UX question has more than one reasonable answer (rare, since Design Discovery should already have resolved direction — `frontend-design.md`), spinning up parallel agents to prototype two directions for comparison is legitimate, distinct from having them build the same feature twice by accident.

**Do not parallelize the core implement → verify → document → commit loop for a single slice.** That loop is inherently sequential (code can't be verified before it's written), and running "one agent to implement, another to write tests, another to write docs" for the same slice at the same time is coordination overhead dressed up as speed. One slice, one thread, still.

### Scoping each agent

Every dispatched agent starts with zero memory of this conversation — it does not share your context, your prior decisions, or your understanding of *why* the task matters. A prompt that assumes shared context produces an agent that guesses, and guessing is exactly what this whole workflow exists to prevent. Every agent's brief states, explicitly:

- **The role and its boundary** — which single role (above) this agent needs, what files it may touch, what it must never touch.
- **What it needs to read first** — the specific docs (architecture, the relevant module's Feature Summary, `docs/DESIGN.md` for anything touching UI), not "read the codebase," which either wastes its context budget or gets skipped.
- **The actual task and its acceptance criteria** — what "done" means for this agent, concretely enough that its output can be checked without redoing the work.
- **What to hand back** — a diff, a report, a set of findings — and in what shape, so the orchestrating session can act on it without another round-trip to ask.

A one-line prompt to a subagent is a false economy: the agent fills the gaps with assumptions, and cleaning up a wrong assumption after the fact costs more than writing the brief would have.

### Isolation and reconciliation

Parallel agents that write code need filesystem isolation — concurrent, uncoordinated edits to the same working tree are a race condition applied to source control. Use a worktree (or equivalent) per implementing agent, and treat the merge back to the main branch as its own explicit step, never an assumption:

1. Each implementing agent finishes on its own branch/worktree, self-reviewed against its own role's checklist.
2. The orchestrating session reviews each agent's diff **as if it hadn't requested it** — the "review the diff as if you didn't write it" discipline from Phase 9, applied doubly here, because a subagent's account of its own work is a claim, not a verification.
3. Conflicts between two agents' changes — same file touched, or worse, the same *concept* implemented two different ways — are resolved explicitly before either merges, never by silently taking one and discarding the other's reasoning without checking why they diverged.
4. The one-slice-one-commit rule (Phase 11) still holds after reconciliation. Multiple agents' work merging into the main branch does not mean multiple uncoordinated commits; it means the orchestrating session commits deliberately, per slice, same as solo work.

### What doesn't change

Everything else in this workflow applies to agent-produced work exactly as it applies to your own: a subagent's slice still needs to be Ready before it starts (Phase 8), still goes through full Verification (Phase 9) — re-run by the orchestrating session, not taken on the subagent's word — still gets documented (Phase 10), and still follows the disclosure rule (necessary-but-unlisted work gets surfaced, whether the session doing the surfacing is you or an agent reporting back to you). Parallelism changes how the work gets produced; it changes nothing about how the work gets trusted.

---

## Phase 8 — Implementation

### Definition of Ready

Phase 9 (below) defines when a slice is *done*. A slice shouldn't start until it's *ready* — the two gates are symmetric, and skipping the entry gate is exactly how implementation starts on missing information and backtracks halfway through. A slice is **Ready** only when all of the following are true:

- Its stated dependencies are actually complete (not "should be fine")
- Its acceptance criteria are written down, not implied
- Its named test cases are defined, not deferred to "we'll figure out tests later"
- Its Security Checklist delta (against `docs/SECURITY_BASELINE.md`) is written, even if "N/A beyond baseline"
- Its Performance Checklist delta (against `docs/PERFORMANCE_BASELINE.md`) is written, even if "N/A beyond baseline"
- The files it's expected to touch are identified
- The agent role(s) or context (Phase 7) it needs are identified
- If the slice has a UI surface: `docs/DESIGN.md` exists and covers this screen's layout system and tokens, and the slice's spec names its empty/loading/error/populated states (see `frontend-design.md`) — a UI slice designed only in its populated state is not Ready. Any recurring container it introduces (a card, a button, a chip, a panel) has been run through the Container Calculation Procedure (`frontend-design.md`'s Geometry & Spatial Composition) — padding, gaps, and icon/text relationships computed against real content extremes, not sized to whatever's in the mock.

If any of these is missing, resolve it before writing code — treat "let's just start and figure it out" as a warning sign, not a time-saver.

### The implementation sequence

Every implementation session, for every slice, follows this sequence:

1. **Read the project documentation** — start from the cold-start reading order (see Phase 12, `methodology.md`), not from memory of a previous conversation.
2. **Identify the current slice** — confirm it against the Roadmap's actual sequencing (see the conflict-handling note in Phase 14, `methodology.md`), not just the next number in the list.
3. **Cross-check dependencies** — confirm the slice's stated dependencies are already built, or explicitly stub them.
4. **Explain the implementation plan** — including any necessary-but-unlisted infrastructure the slice will actually need (a test runner that doesn't exist yet, a route group that doesn't exist yet, a DAL file the architecture implies but the slice's own file list omits). Disclose these, don't silently add them and don't silently skip them.
5. **Wait for approval** before writing code, unless the person you're working with has explicitly authorized moving faster for this session.
6. **Search before you create.** Before writing any new helper, component, hook, query, or utility, search the codebase for an existing one that does the job (or nearly does — extending beats duplicating). Amnesiac sessions are the single biggest source of redundant code in AI-assisted projects: each session happily re-creates a `formatDate`, a validation helper, or a button variant that already exists three folders away. Reuse it, or extend it; only create new when the search genuinely comes up empty — and place the new one where the Repository Standards say shared code lives, so the *next* session's search finds it.
7. **Implement only that slice.** Touch only the files its scope actually requires, plus whatever was disclosed in step 4.
8. **Follow the architecture strictly.** If implementation reveals the architecture doc is wrong or silent on something, stop and flag it — don't quietly patch around it and don't quietly patch the architecture doc either, unless you have standing authorization to do so.
9. **Avoid unrelated changes.** A formatting pass, a rename, a "quick cleanup" of adjacent code — none of these belong in a slice's diff unless the slice is about that. (Cleanup has its own scheduled home: the module-close hygiene pass in Phase 10.)

### Introducing a new dependency

A third-party package is a standing liability, not a free convenience — it's code you didn't write, don't fully control the update cadence of, and are trusting with whatever access it needs. Before adding one, answer all four, briefly, and disclose the answer in the same breath as proposing the dependency (same spirit as step 4 above's "disclose necessary but unlisted work"):

1. **Justification** — what specific problem does this solve that isn't reasonably solved by the standard library or something already in the dependency tree? "It's popular" isn't a justification; "the standard library's X has no support for Y, which this slice requires" is.
2. **Maintenance status** — when was it last published, how does it handle security patches, is it effectively single-maintainer with a bus-factor of one? A stale or abandoned package is a liability regardless of how good its README looks.
3. **License check** — is its license compatible with this project's license and any constraints from clients/enterprise customers (some organizations flatly reject copyleft licenses in commercial products, for instance)? Don't assume MIT/permissive without actually checking.
4. **Removal strategy** — if this needs to be replaced later, does it leak into every call site, or is it isolated behind an abstraction (the same seam-based thinking as the architecture's own abstraction boundaries)? Prefer designs where swapping the dependency later is a contained change.

This applies to genuinely new packages, not to using more of a dependency that's already justified and in place. For regulated/enterprise-grade projects (Phase 0, `methodology.md`), treat all four as launch-blocking, not advisory — a security or license review that finds an unvetted dependency late is far more expensive than five minutes of justification up front.

### Schema & migration slices

Schema changes are the highest-blast-radius thing a session does — code can be reverted; a botched migration against real data often can't. Standing rules, regardless of stack:

- **Migrations are forward-only and immutable once applied anywhere beyond the local machine.** A wrong migration gets a new corrective migration, never an edit — editing an applied migration forks reality between environments.
- **Test the migration against a production-like copy of the data when it's written**, in its own slice's verification — not for the first time at release. Empty-database success proves almost nothing; real data has the nulls, duplicates, and orphans that break migrations.
- **Destructive changes use the expand–contract pattern, as separate slices**: (1) add the new column/table and write to both, (2) backfill and switch readers, (3) only then drop the old one — each step deployed and verified independently. Never rename or drop in the same slice that adds the replacement; that's the zero-downtime rule and the rollback rule in one.
- **Seed and reference data are versioned migrations too**, not manual inserts someone has to remember per environment.
- **Every index has a reason recorded** (the query it serves), and every new query pattern asks whether it needs one — this pairs with the Performance Baseline's no-unindexed-hot-query rule.

### Configuration & environment changes

The slice that introduces an environment variable updates `.env.example` **and** the config documentation *in the same commit* — not later, not at release. "Works locally, staging is missing a variable nobody wrote down" is otherwise guaranteed, and it always surfaces at the worst moment. Differences between dev/staging/prod behavior (a stubbed provider, a disabled job) are recorded when created. Secrets never enter source control or client bundles — that's a Security Baseline check, verified per slice, not a launch-day discovery.

### Third-party integration slices

Payments, email, webhooks, external APIs — where "worked in the demo" and "works in production" diverge most:

- **Sandbox/test-mode first, always**; switching to live credentials is its own explicitly-verified step (Phase 16 confirms it, but the switch itself is never silent).
- **Every incoming webhook verifies the provider's signature and is idempotent** — providers redeliver, sometimes many times; an unverified or non-idempotent handler is both a security hole and a double-charge/double-email bug waiting.
- **Every outgoing call has a timeout and a decided failure behavior** (retry with backoff, queue, or fail loudly — chosen, not defaulted), recorded in the Decision Log. What the user sees when the provider is down is part of the slice's spec, not an exercise for production.

### Mutation slices — the concurrency checklist

AI-generated code is systematically single-user-minded; these questions are asked (and answered in the slice's notes) for every slice that writes data:

- **What happens when two users do this at the same instant?** Uniqueness and referential rules are enforced by *database constraints*, not application-level "check then insert" — the check-then-act gap is a race by construction.
- **Is every multi-step write wrapped in a transaction**, so a failure halfway can't leave half-written state?
- **Is the operation safe against double-submit and network retries** — idempotent by design, or protected by an idempotency key for anything money-adjacent?
- **Are numeric updates atomic** (`SET stock = stock - 1 WHERE stock > 0`-style), never read-modify-write from application memory? Stock counts, balances, and quotas are exactly where this bug costs real money and only appears under production load.

### Risky slices ship behind a flag

For a system with real users, a slice that changes existing behavior riskily (a payment flow change, a data-model switch, a rewrite of a hot path) ships behind a feature flag with a defined kill switch — deploying and releasing become separate decisions, and a bad release becomes a toggle, not a rollback. Two rules keep this from rotting: the flag's *removal* is scheduled as its own future slice when the flag is created (permanent flags are dead code with extra branches), and the off-state is verified too — a kill switch nobody tested is a hope, not a control.

---

## Phase 9 — Verification

A slice is not complete until all of the following pass — and passing means actually running them, not asserting that they would pass:

- Lint
- Type checking
- Unit tests (including the slice's own *named* test cases specifically, not just "tests exist")
- Integration tests, where the slice's behavior spans a real boundary (a real database, a real HTTP call)
- End-to-end verification for anything with a runtime surface — actually start the app and drive the feature, in a real browser or real CLI invocation, not just through mocks. **This step catches classes of bugs the other steps structurally cannot**: a database that was never migrated, a test isolation leak that pollutes real dev state, an assumption about framework behavior that was subtly wrong, a form validation bug that only manifests with real input shapes (an unchecked checkbox, a null vs. undefined value). Treat "I ran it and watched it work" as a distinct, non-optional verification step, not a nice-to-have.
- For slices with a UI surface, the end-to-end step expands to the **UI verification checklist in `frontend-design.md`**: multiple viewports, all four content states (empty/loading/error/populated), a throttled-network and throttled-CPU pass, zero layout shift, the transform/opacity animation rule, `prefers-reduced-motion`, keyboard-only operation, contrast/touch targets, draft protection on long forms, and conformance to `docs/DESIGN.md` tokens.
- Security review against the project's **Security Baseline** (`docs/SECURITY_BASELINE.md`) plus the slice's stated delta — the baseline is the checklist; the delta is what this slice adds to it
- Performance review against the project's **Performance Baseline & Budgets** (`docs/PERFORMANCE_BASELINE.md`) plus the slice's delta — a numeric comparison where budgets exist (load time, bundle size, latency), not an impression
- Code review for correctness, reuse, and scope discipline — a pass distinct from the security/performance passes, ideally done by inspecting the diff as if you didn't write it

When a test fails, the default assumption is that the implementation is wrong, not the test. Never weaken an assertion to make a broken behavior pass.

### Which layer a test belongs to

"Tests exist" is not a strategy. The default distribution, adapted per project in the planning docs:

- **Unit tests** for pure logic: business rules, calculations, validation, state transitions. Fast, many, no I/O.
- **Integration tests** for anything that crosses a real boundary — a real (local/test) database, a real HTTP handler. This is where constraint enforcement, transactions, and the concurrency checklist above get proven, because mocks structurally cannot prove them.
- **A few real end-to-end tests** for the money paths — the flows the business actually depends on — kept few because they're slow and brittle in bulk.

Three rules that matter *especially* for AI-written tests:

- **Test behavior, not implementation.** A test that mocks every collaborator and asserts internal call order will happily stay green while the real app is broken — over-mocked tests are how "tests pass but nobody ran it" happens *with* tests. If a test would survive a correct refactor and fail on a real behavior change, it's testing the right thing.
- **Test data comes from factories/builders per test, never shared mutable seed data** — shared fixtures are how test-isolation leaks and order-dependent flakiness get built.
- **A flaky test is a bug, fixed the day it's noticed** — never rerun-until-green, never quarantined indefinitely. A suite that's allowed to be flaky stops being evidence, and Phase 9 runs on evidence.

---

## Phase 10 — Documentation Update

Immediately after a slice passes verification — not batched, not postponed:

- **A per-slice note** — even a short one — capturing what was built, what was disclosed as a necessary addition, what bugs were caught during self-review, and what the next slice needs to know.
- **The Roadmap's progress log** — mark the slice done, in one line, with a pointer to its note.
- **The Changelog** — one entry, written for a human skimming history, not for the AI itself.
- **The "what's next" file** — overwritten (not appended) to point at the next slice, including any gaps or conflicts already discovered in that slice's own spec (see Phase 12, `methodology.md`).
- **The module's full Feature Summary** — only when this slice is the *last* slice in its module (see Phase 12's two-tier pattern, `methodology.md`). Not every slice needs one; every module does, exactly once.
- **The module-close hygiene pass** — also when this slice closes a module, and *as its own separate commit* so it never contaminates a feature diff. This is the scheduled home for the cleanup that Phase 8 rightly bans from feature slices, run while the module is still fresh and cheap to fix rather than saved for a giant end-of-project scrub:
  - **Redundancy sweep**: duplicated logic within the module merged into one implementation; near-duplicate helpers/components consolidated; copy-paste variants unified behind parameters.
  - **Dead-code sweep**: unused exports, unreachable branches, orphaned files, commented-out code, and leftover experiments deleted — run the project's dead-code/unused-dependency tooling (configured at bootstrap), don't hunt by eye alone.
  - **Naming conformance**: every name this module introduced checked against `docs/REPOSITORY_STANDARDS.md` and the domain's real vocabulary — renames are cheap now, expensive after three more modules build on top.
  - **TODO reconciliation**: every TODO/FIXME the module left behind either becomes a slice, a Risk Register entry, or gets deleted — none survive as vague debt.
  - Verification (Phase 9's lint/type/test gates) runs again after the pass — hygiene that breaks the build isn't hygiene.
- **An ADR** — only if the slice made a real architectural decision (a new library, a new external dependency, a caching strategy, a schema-shape trade-off). Most slices won't need one.
- **A Decision Log entry** (`docs/DECISIONS.md`) — for the smaller decisions that don't rise to ADR weight but would still get silently re-litigated later if unrecorded: a naming choice with a real reason behind it, picking one library API over another already-approved one, a small trade-off made for now. One or two lines, dated, with the reason. This is what stops the same small argument from happening again in a later session — cheaper than an ADR, more durable than a comment.
- **A Risk Register entry** (`docs/RISKS.md`) — only if the slice surfaced a meaningful technical or business uncertainty that isn't resolved yet: a dependency on an external provider whose reliability is unknown, a scaling assumption that hasn't been load-tested, a compliance question without a confirmed answer, a security trade-off accepted for now but not permanently. This is deliberately not the same document as an ADR — an ADR records a decision that was made; a risk records an exposure that hasn't been closed out. Most slices won't need this either, but when one does, it's important enough to not just leave in a comment.

See `templates.md` for concrete templates of every document above.

---

## Phase 11 — Git Workflow

One slice, one commit. Recommended flow:

```
Implement → Self-review against checklist → Verify (Phase 9) → Document (Phase 10) → Commit → Stop
```

- Never bundle unrelated slices into one commit, even if they're small.
- Write the commit message to explain *why*, including any bug fixed or gap resolved during the slice — a future session (or a future you) reading `git log` should understand the slice's story without opening the diff.
- Don't amend published commits; a correction is a new commit.
- Confirm the commit's actual diff before committing — `git status`/`git diff` review, not just trust that you know what you changed. This catches accidental inclusion of generated artifacts, local secrets, or scope creep that snuck in.
- Get explicit confirmation before committing unless you have standing authorization for this session to commit autonomously.

---

## Phase 13 — Verification & Anti-Hallucination Discipline

Treat every claim about how a library, framework, or platform behaves as unverified until checked — especially version-specific behavior, since training data skews toward whatever was common historically, not necessarily what's current in this exact project's dependency versions.

Concretely:

- **Check the installed version's own documentation** (vendored docs, changelogs, or by reading the library's source/types) before relying on an API you're not fully certain about — particularly after a major version bump.
- **Verify empirically, not just by reading docs.** If a doc says a behavior should hold, write a small throwaway check that proves it in this actual codebase, then delete the throwaway and rely on the real test suite. "I read that this works" and "I watched it work" are different confidence levels — earn the second one for anything load-bearing.
- **Distrust your own first implementation of anything security- or correctness-sensitive** enough to write a test that would catch you being wrong — a deliberately-introduced violation that should fail, then confirm it does, then remove it. This applies to authorization boundaries, uniqueness constraints, drift-detection tests, and anything else whose entire value is "this fails loudly when someone breaks it."
- **When a test fails, suspect the implementation first, the test's assumption second, and "the framework must be broken" last, in that order.**
- **Treat real end-to-end runs as an oracle above unit tests for framework-integration questions** — a mocked test can be wrong about how the real framework behaves; running the real thing cannot.

---

## Phase 15 — Recovery Procedure

Sometimes implementation goes sideways — verification keeps failing, an assumption turns out wrong three files into a change, or the deeper you get, the less the current approach seems to fit. The instinct to push through and "just make it pass" is exactly how a small problem becomes a large, tangled one. When that happens:

1. **Stop implementing.** Don't attempt another quick fix on top of the last one — that's usually how a one-line problem becomes a ten-file problem.
2. **Re-read the Architecture document** for the area in question — confirm the current approach actually matches what was designed, rather than something that drifted during the struggle.
3. **Re-read the current module's documentation** (its Module Breakdown entry, its prior Feature Summary, its per-slice notes) — the answer is often already recorded, just not what was being followed.
4. **Determine what kind of problem this actually is**: a genuine gap or conflict in the plan (see Phase 14, `methodology.md`), a real architectural decision that needs making (write an ADR), or an unresolved uncertainty that shouldn't block indefinitely but needs to be tracked (write a Risk Register entry).
5. **Do not continue implementation until the conflict is actually resolved** — not papered over. If a fix restores green tests but you can't articulate *why* it works, that's a signal to keep investigating, not to move on.
6. **Once resolved, record it** — as a standing decision, an ADR, or a Risk Register entry, whichever fits — so the next session (or the next slice) doesn't rediscover the same dead end.

This is the deliberate opposite of "keep trying things until something works." Repeated verification failures are information about the plan or the approach, not just an obstacle to brute-force past.

---

## Golden Rules

The full Golden Rules list lives in `methodology.md` (Book 1) — it's shared across the whole three-book set and isn't repeated here to avoid the two copies drifting apart. The ones most relevant to this book's day-to-day work:

- Disclose necessary but unlisted work — don't silently add it, and don't silently skip it because it wasn't named.
- Verify empirically before trusting an assumption, especially about framework/library behavior.
- Run the real thing before declaring a slice done — tests passing is necessary, not sufficient.
- Review before commit; document before the next slice.
- A slice must be Ready before it's implemented, not just Done when it's finished.
- When verification keeps failing, stop and re-read before trying another fix.
