# Book 5 — System Design & Data Structures: Access-Pattern-First Engineering

The failure mode this book prevents: **brute-force defaults** — code that answers a question by
scanning everything, because a scan was the first thing that compiled. Full-table walks to check
if a username exists. `OFFSET 50000` pagination. Recomputing a count on every page load. Loading
every row into application memory and filtering there. Each one works perfectly in the demo with
40 rows and falls over — or quietly burns money — at 400,000.

The counter-example to internalize: when Instagram tells you "wrong password," it has not
searched its user table. It did **one indexed lookup** on the identifier (a B-tree or hash
index — the answer arrives in a handful of page reads regardless of whether there are ten
users or two billion), pulled a single row, and ran **one hash comparison** against the stored
password hash. The error message is deliberately uniform ("incorrect username or password") so
attackers can't tell which half failed, and a **rate-limit counter** (a small fixed-size
structure keyed by account and IP, not a log query) decides whether to even bother checking.
Nothing in that flow costs more when the platform grows a thousandfold. That is the discipline:
**the cost of answering a question should depend on the size of the answer, not the size of
the data.**

This book is technology-agnostic in the same way the rest of the workflow is: the *shapes*
below (membership, exact lookup, range, prefix, top-K, traversal, proximity) exist in every
stack; only the concrete tool names differ. Where concrete names appear, treat them as
examples to verify against your project's actual stack and versions — never as facts to
assume (Book 2's anti-hallucination rules apply to performance claims doubly).

---

## The access-pattern interview — five questions per feature

Before choosing storage, index, cache, or algorithm for any slice, answer these five questions
in writing (they go in the slice plan; the aggregate answers live in the Architecture Document's
Data & Access Patterns section, below):

1. **What question does the code ask of the data?** Not "what data do we have" — what
   *question*. "Is this username taken?" is a membership test. "Show this user's last 20
   orders" is a range scan on an index. "Top 10 sellers this week" is a top-K. The question's
   shape, not the entity's shape, determines the structure.
2. **How often is it asked, and what's the read/write ratio?** A 1000:1 read-heavy question
   justifies precomputation and caching; a write-heavy one justifies queues and batching.
3. **How big does the data get at 10x the expected load?** (Same 10x as Phase 4.) 10k rows
   forgive everything; 10M rows forgive nothing. Write the number down — "a lot" is not a number.
4. **How fresh and how exact must the answer be?** Exact-and-instant is the most expensive
   combination. Many product questions tolerate approximate ("~1.2M views") or stale-by-seconds
   (a cached count) — and saying so unlocks structures that are orders of magnitude cheaper.
5. **What happens when it's slow?** A slow admin report annoys one person; a slow login blocks
   everyone. Hot-path questions get the strongest structures; cold-path ones get the simplest.

A slice that touches storage is not **Ready** (Book 2, Phase 8) until these five answers exist
for its hottest query.

---

## Problem shape → data structure

The left column is the question the code asks. **Default** is what to reach for first in a
typical database-backed web app — note how often it is simply "an index." **At scale** is what
to research *when the measured numbers demand it* (see the grade dial below), not before.

| Question shape | Default (right ~95% of the time) | At scale / specialized |
|---|---|---|
| "Does X exist?" (username taken, email registered, seen-before) | **Unique index** on the column — one B-tree descent, O(log n) | **Bloom or cuckoo filter** in front of the store: tiny memory, O(1), "definitely not present / probably present" — perfect as a cheap pre-check that eliminates most lookups (how large systems answer membership without touching the database); breached-password checks use k-anonymity range queries, never full downloads |
| Exact lookup by key (session token → user, id → row) | **Primary key / hash index**; hash map in process memory for small fixed sets | Distributed KV / cache tier (see cache-aside pattern below) |
| Range or ordering (orders between dates, newest-first lists) | **B-tree index matching the sort**, composite `(user_id, created_at)` for per-user timelines | Time-series partitioning when a single table's index no longer fits hot memory |
| Prefix search / autocomplete | **B-tree prefix scan** (`LIKE 'abc%'` uses the index; `'%abc%'` does not — that's a scan) | **Trie / finite-state transducer**, or a search engine's edge-ngram analyzer; precomputed top suggestions per prefix |
| Full-text search ("find posts mentioning…") | The database's built-in **inverted index** (e.g., Postgres full-text search) | Dedicated search engine (inverted index + relevance scoring) as a *separate* indexed copy of the data — never `LIKE '%term%'` |
| Top-K / leaderboard | `ORDER BY score DESC LIMIT k` on an **indexed column** | **Sorted set / skip list** (the structure behind Redis leaderboards): O(log n) insert, instant rank queries; a **heap** for streaming top-K in one pass |
| Counting things shown on every page (badges, likes, followers) | A **counter column** maintained on write (transactionally or via the outbox pattern) — never `COUNT(*)` per request | **Sharded counters** for hot rows (split one contended counter into N, sum on read); **HyperLogLog** for distinct-counts (unique visitors) at ~kilobytes of memory, ±2% |
| "How many times has X occurred?" over huge streams | Aggregate table updated by a background job | **Count-min sketch** — fixed memory, approximate frequencies |
| Scheduling / "what's due next?" | A `due_at` **indexed column** polled by a worker | **Priority queue / heap**; delay queues in a message broker |
| FIFO work (emails to send, webhooks to deliver) | A jobs table with status + index, or the platform's job runner | A real **message queue** with acknowledgments, retries, and dead-letter handling |
| Rate limiting (login attempts, API quotas) | **Token bucket or sliding-window counter** in a fast KV store, keyed by account *and* IP — fixed memory per key | Same structure, replicated; never "count rows in the requests table" |
| Uniqueness across time ("did we already process this webhook/payment?") | **Idempotency key** with a unique index; insert-first, act-second | Same, plus TTL cleanup; a bloom filter pre-check when volume is extreme |
| Relationships / traversal (followers, org charts, dependency graphs) | **Adjacency table** (`follower_id, followee_id`, indexed both directions) — relational databases handle 2–3 hop queries fine | Graph database *only* when deep/variable-depth traversal is the core product, not a feature |
| Proximity ("stores near me") | The database's **geospatial index** (R-tree/GiST) | **Geohash / quadtree** bucketing for massive point sets; precomputed region tiles |
| Sortable unique IDs | **UUIDv7 / ULID** (time-ordered → index-friendly) or the database's sequence | Snowflake-style IDs when generating across many nodes |
| "Page 400 of results" | **Keyset (cursor) pagination** — `WHERE (created_at, id) < (…) LIMIT n` uses the index; `OFFSET n` reads and discards n rows and gets slower every page | Same. Keyset pagination is the answer at every scale; build it in from slice one |

Two structures deserve a special callout because they embody the mindset:

- **The bloom filter** answers "is X possibly in this set?" in constant time and a few
  megabytes for hundreds of millions of items, with zero false negatives. It's the canonical
  "don't touch the database to say no" tool: caches use it to skip lookups for keys that were
  never stored, crawlers use it for "have I seen this URL," and registration flows at extreme
  scale use it to reject taken usernames instantly (a "probably taken" still verifies against
  the unique index — the filter's job is making the *common* case free).
- **The index** is the humblest and most-skipped answer. Most "we need Redis/Elastic/a new
  service" conversations in small projects are actually "we forgot an index" conversations.
  Prove the index insufficient with numbers before adding a component.

---

## Workload shape → system design pattern

| Workload | Pattern | The mechanism |
|---|---|---|
| Read-heavy, tolerable staleness (product pages, profiles) | **Cache-aside + CDN** | Check cache → miss → read DB → populate with TTL. Invalidate on write or accept TTL staleness — decide which, per entity, in writing. Static assets go to a CDN, always. |
| Read-heavy, must reflect writes (dashboards over big data) | **Precompute**: materialized views / summary tables | A background job maintains the expensive aggregate; requests read a plain indexed table. Freshness is a stated number ("updated every 5 min"), not an accident. |
| Write-heavy bursts (checkout, ingestion, spikes) | **Queue + async workers** | Accept fast (validate, enqueue, 202), process at a controlled rate. Every consumer is idempotent (the idempotency-key row above) because every queue delivers at-least-once. |
| Fan-out (followers see new posts) | **Fan-out on write vs. read — hybrid** | Push new items to each follower's precomputed feed list (write fan-out) for normal users; for accounts with millions of followers, *don't* — merge their posts in at read time. The hybrid exists because neither extreme survives the celebrity case. |
| Hot counter / hot row (viral post's like count) | **Shard the counter, or buffer-and-flush** | N sub-counters summed on read, or increments batched in a fast KV store and flushed to the database periodically. |
| Cross-service consistency (order placed → email sent) | **Outbox pattern** | Write the event to an `outbox` table *in the same transaction* as the business change; a relay publishes it. Never "write DB, then hope the second call succeeds." |
| Search alongside a database | **Indexed copy, one-way sync** | The search engine holds a denormalized copy fed by the outbox/CDC stream; the database stays the source of truth. |
| Failure containment when a dependency is down | **Timeouts + retries with backoff + circuit breaker** | Every external call has an explicit timeout; retries are bounded and jittered; repeated failure opens the circuit and degrades gracefully instead of queueing doom. |
| Protecting the trust boundary cheaply | **Reject early, in order of cost** | Rate limit (KV counter) → validate shape → authenticate (indexed lookup + hash compare) → authorize → *then* touch business data. The login example again: the expensive step runs last and rarely. |

---

## Caching as a system, not a bolt-on — the discipline behind the big platforms

Cache-aside in the table above is the entry move. At Production-SaaS grade and up, caching becomes a *layered system with correctness rules*, and this is exactly where large platforms differ from tutorials — not in whether they cache, but in how deliberately they handle the failure modes caching creates. Every rule here exists because skipping it produces a named production incident:

**The ladder — cache at the cheapest layer that can answer.** In cost order: the user's own browser (`Cache-Control: immutable` + content-hashed asset filenames — a returning visitor re-downloads *zero* unchanged bytes), the CDN (every static asset, always; plus whole-page or fragment caching for anonymous read-heavy pages), the edge KV / application cache (Redis or equivalent — session lookups, rendered fragments, expensive query results), and only then the database (whose own buffer pool is a cache too — a hot working set that fits in DB memory is why "add Redis" is often premature). Each request should be answered at the highest rung that can answer it correctly; a request that reaches the database for something the CDN could have served is paying 100× the cost.

**Invalidation is decided per entity, in writing, from exactly three honest options**: (1) **TTL-only** — accept staleness up to N seconds; state N and who signed off; (2) **event-driven** — writes publish invalidations through the outbox/CDC stream (never fire-and-forget from application code, which loses invalidations exactly when the process crashes mid-write); (3) **versioned keys** — the key embeds a version/updated-at, so stale entries are never *read* and simply age out. "We invalidate when we remember to" is not one of the options.

**The failure modes that separate industrial caching from tutorial caching:**

- **Stampede (dogpile) protection.** When a hot key expires, a thousand concurrent requests all miss and all hit the database at once — the cache's expiry becomes a self-inflicted outage. Fixes, cheapest first: **request coalescing / single-flight** (one request recomputes, the rest wait on it), **probabilistic early refresh** (each request has a small chance of refreshing *before* expiry, so expiry never lands on a crowd), or a **soft-TTL** where one worker refreshes while everyone else keeps reading stale.
- **Stale-while-revalidate as the default read posture** for anything not correctness-critical: serve the cached value instantly, refresh in the background. The user gets sub-millisecond reads; freshness is eventual by a stated number of seconds. HTTP has this built in (`stale-while-revalidate`); application caches implement the same idea with soft/hard TTL pairs.
- **TTL jitter.** A batch job that warms 10,000 keys with identical TTLs schedules a synchronized mass-expiry stampede for exactly TTL seconds later. Add ±10–20% random jitter to every TTL, always — it costs nothing and deletes an entire incident class.
- **Negative caching.** "This does not exist" is a cacheable answer (with a *short* TTL — existence can change). Without it, requests for missing things (deleted profiles, mistyped slugs, enumeration probes) bypass every cache layer and hammer the database — an accidental denial-of-service vector the bloom filter row above solves at larger scale.
- **Hot-key handling.** One viral entity can exceed what a single cache node serves. First fix: a tiny in-process L1 (per-instance memory, milliseconds of TTL) in front of the shared L2 — absorbs the flood with bounded staleness. Bigger scale: replicate the hot key under N suffixed copies and read a random one.
- **Cache warming for predictable load** — deploys that clear caches, morning traffic ramps, a scheduled sale: pre-populate the known-hot keys *before* traffic arrives instead of letting the first thousand users pay the misses. A deploy that empties the cache into peak traffic is a self-inflicted stampede.

**Correctness under replication — the bug class caching and read-replicas share.** A user writes, then immediately reads from a cache/replica that hasn't caught up, and their own change is missing — "I saved it and it's gone." Decide the answer per flow, in writing: **read-your-own-writes** (after a write, that user's reads pin to the primary/fresh path for the replication-lag window — session pinning or a short "recently wrote" flag), or **explicit optimism** (the UI shows the write locally without re-fetching — the optimistic-UI pattern from Book 4, solving the same problem at the presentation layer). Fan-out of this decision is cheap; support tickets from "my edit disappeared" are not.

**Request-layer batching (the N+1 killer at the API tier).** The N+1 table row above covers SQL; the same disease reappears one layer up when each item of a list independently fetches its author/thumbnail/price from a service or loader. The fix is the **dataloader pattern**: collect all keys requested within one tick/request, issue one batched fetch, fan results back out — plus per-request memoization so the same key is never fetched twice in one render. GraphQL made the pattern famous; it applies to any composed read path.

**Connection pooling is not optional at scale.** Databases hold a small number of expensive connections; serverless and per-request architectures can exhaust them instantly. A pooler (built into the driver, or PgBouncer-style middleware) with stated pool sizes per environment is part of the Data & Access Patterns section, not an ops afterthought.

Every mechanism above obeys the escalation rule below: it enters the architecture with a measurement proving the previous, simpler rung failed — cached-vs-uncached latency, hit ratio, replication lag, connection counts — recorded in an ADR with the threshold that triggers re-evaluation.

---

## Staying up under load — resilience patterns for the trust boundary and beyond

Timeouts, bounded retries, and circuit breakers (table above) keep *one* failing dependency from spreading. At high traffic the harder problem is **the system protecting itself from its own demand**:

- **Backpressure over buffering.** An unbounded queue in front of a slow consumer doesn't absorb overload — it hides it, converting "some requests fail fast" into "every request times out three minutes from now, and the retry storm doubles the load." Queues get **bounded depths**; when full, the system says no *at the front door* (429 + `Retry-After`) instead of collapsing at the back.
- **Load shedding, cheapest-first.** When saturated, drop work in order of cost and value: anonymous/crawler traffic before logged-in users, prefetch and analytics before interactive requests, feed refreshes before checkouts. Deciding this priority order *in advance* (in the Architecture Document) is what makes shedding a design instead of an outage.
- **Graceful degradation tiers, stated per feature.** Each user-facing feature names its degraded mode: recommendations fall back to "most popular," search falls back to prefix match, the personalized homepage falls back to a cached generic one. A feature with no stated degraded mode degrades anyway — by erroring.
- **Bulkheads.** Isolate resource pools (connections, workers, threads) per dependency or per feature class, so one slow downstream can't drain the shared pool and take unrelated features with it — the same reasoning as the circuit breaker, applied to capacity instead of calls.
- **Idempotency everywhere retries exist** (which is everywhere): client retries, queue redeliveries, and load-balancer replays all mean every mutation path either has a natural idempotency key or gets an explicit one. This is the Book-2 mutation checklist's rule, promoted to a system-wide invariant at this grade.

Grade-dial note: bounded queues and stated degradation modes are cheap enough to justify at Production-SaaS grade; bulkheads and formal shedding tiers are High-scale-grade machinery that arrive with load-test numbers, not vibes.

---

## The grade dial — Phase 0 decides how far up this book you climb

This book is the easiest one in the workflow to over-apply. **Resume-driven architecture — adding
Kafka, microservices, or a graph database because big companies have them — is the same failure
as the brute-force scan, with better marketing.** The project grade (Phase 0) sets the ceiling:

- **Prototype** — one database, correct indexes, keyset pagination, password hashing done right.
  Nothing else from this book. The win at this grade is *not designing yourself into a corner*:
  time-ordered IDs, cursor pagination, and an outbox-shaped events table cost nothing now and
  make every later pattern a bolt-on instead of a rewrite.
- **Internal tool** — the above, plus cache-aside for anything measured slow, and a jobs table
  for anything that shouldn't block a request.
- **Production SaaS** — the above, plus: rate limiting on the trust boundary, a real queue with
  dead-lettering (bounded, with a stated full-queue behavior), counter columns / materialized views
  for every count or aggregate that renders on a hot page, a search index if search is a feature,
  read replicas when read load asks (with a written read-your-own-writes answer), and the caching
  ladder run deliberately — per-entity invalidation choices, TTL jitter, stale-while-revalidate
  as the default read posture, stampede protection on hot keys, and stated degradation modes for
  every user-facing feature.
- **High-scale / regulated** — only here do the probabilistic structures (bloom filters,
  HyperLogLog), sharded counters, feed fan-out hybrids, partitioning strategies, hot-key L1/L2
  replication, bulkheads, and formal load-shedding tiers earn their operational cost — and each
  one arrives with a measured number proving the previous tier failed.

**Escalation rule: a new component or exotic structure may enter the architecture only with a
measurement attached** — the query plan, the latency percentile, the row count — recorded in an
ADR. "We might need it later" is Phase 14 material (flag it as a future concern), not a license
to build it now.

---

## The research protocol — "best after research" means *this*

Claims about performance are the most hallucination-prone claims there are. For every structure
or pattern this book recommends, before it enters the architecture:

1. **Verify the mechanism in current documentation** for the project's actual database/runtime
   *versions* (Book 2's rule). Whether the query planner uses an index for a given operator, or
   what a managed queue guarantees, changes between versions.
2. **Ask the database, not intuition**: run the query plan (`EXPLAIN` or equivalent) against a
   dataset seeded to the 10x size from question 3 of the interview. A plan that says "sequential
   scan" on a hot path is a failed Ready check, not a note for later.
3. **Measure the hot path with realistic data**, and compare against `docs/PERFORMANCE_BASELINE.md`'s
   numeric budgets — the budgets are the pass/fail line, not vibes.
4. **Record the decision as an ADR** with the numbers: the question shape, the options
   considered, the measurement that picked the winner, and the threshold that would trigger
   re-evaluation ("revisit when the table passes 5M rows or p95 exceeds 200ms").

---

## Where this hooks into the phases

- **Phase 2 (Requirements)** — capture the raw material as testable NFRs: expected entity
  volumes at launch and at 10x, hottest pages/endpoints, freshness tolerances ("dashboard may
  lag 5 minutes"), and the numeric latency budgets that seed `PERFORMANCE_BASELINE.md`.
- **Phase 3 (Architecture Document)** — add a **Data & Access Patterns** section. Per core
  entity, one block:

  ```
  ### <Entity>
  Expected volume: <launch> → <10x>            Read:write ≈ <ratio>
  Hottest questions:
    1. "<question>" — shape: <membership|exact|range|prefix|top-K|count|traversal|proximity>
       → structure: <index/cache/queue/…>  (budget: <p95 target>)
  Growth plan: <what changes at 10x, and the measured threshold that triggers it>
  ```

- **Phase 4 (Architecture Review)** — the Scalability and Performance bullets get teeth: walk
  the Data & Access Patterns section and confirm every hot question names a structure whose
  cost doesn't grow with total data size, and that the trust boundary rejects abuse in
  cheap-first order.
- **Phase 8 (per-slice Ready check)** — the five-question interview, answered for this slice's
  hottest query, is part of Ready for any slice touching storage.
- **Phase 9 (Verify)** — for slices with a data path: seed realistic volume, run the query
  plan, and check the measured latency against the budget. "It returned quickly on my 12 test
  rows" verifies nothing.

---

## Red flags — grep-able smells that mean this book was skipped

Any of these on a hot path is a defect to fix (or an ADR to justify), not a style preference:

- A **membership or existence check without an index** backing it — or worse, `SELECT *` then
  checking in application code.
- **`LIKE '%term%'`**, or filtering/sorting in application memory over data the database could
  have filtered/sorted via an index.
- **`OFFSET`-based pagination** on anything that grows.
- **`COUNT(*)` (or any aggregate) executed per request** for a number rendered on every page.
- **N+1 query loops** — one query per item of a list the database could join or batch.
- **Unbounded in-memory collections** standing in for a queue, cache (no eviction policy = a
  memory leak with good intentions), or "we'll sort it in code."
- **Rate limiting or "recent activity" answered by scanning history tables** instead of a
  counter structure.
- **A second data system (cache, search engine, queue) added with no measurement** demonstrating
  the database alone failed — or a cache with no stated invalidation/TTL policy.
- **Authentication that leaks or scans**: distinguishable "no such user" vs "wrong password"
  errors, missing rate limiting, or any credential check that isn't a single indexed lookup
  plus a constant-cost hash comparison.
- **A whole batch of cache keys written with identical TTLs** — a synchronized mass-expiry
  stampede on a timer; TTLs get jitter, always.
- **A hot key whose expiry is a crowd event** — no single-flight/coalescing, no early refresh,
  no soft-TTL: the cache's own expiry schedule becomes the outage.
- **Cache invalidation fired from application code outside the transaction** ("write DB, then
  tell the cache") — lost exactly when the process dies between the two; invalidations ride the
  outbox/CDC stream or the key is versioned.
- **A user's own write missing from their next read** (cache or replica lag) with no written
  read-your-own-writes answer.
- **An unbounded queue in front of a slow consumer** — overload hidden as latency, plus a retry
  storm when it finally surfaces; queues are bounded and the front door says no.
- **Per-item fetches to a service/loader inside a list render** — the N+1 disease at the API
  tier; batch with a dataloader-style collector.
- **Serverless/per-request handlers opening raw database connections** with no pooler and no
  stated pool sizes.
