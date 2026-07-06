---
title: REQUIREMENTS — dyad-system: claim/invariant validated-factory engine
commission: dyad-system-engine
owner: dyad-bond
---

# REQUIREMENTS — dyad-system: claim/invariant validated-factory engine

> **Bond's Truth layer for this commission.** Intent, semantic requirements, and the acceptance
> falsifiers that cairn's specification and dyad-swe's engine are ratified against. See `README.md` for
> the repo topology, ownership map, parties, and lifecycle.

## 0. Terms & label index

Parties, ownership, and lifecycle are in `README.md`. This section defines only the vocabulary the
requirements below are written in.

**Domain.** *claim* — a single entity with one stable identity that persists through its lifecycle
states; the concept both corpus files hold under divergent conventions today. *candidate* — a claim in
its live, mutable, under-investigation state. *invariant* — the same claim in its ratified, frozen
state. *claim-core* — the field contract every claim satisfies in every state, defined in R1–R2.
*graduate* — transition a claim from candidate to invariant state, in place, preserving its id, per R3
and R6. *retire* — remove a claim that is no longer relevant to the dyad; the only removal, per R5.
*corpus files* — two YAML files that partition claims by state: `theory-pipeline.yaml` holds
candidates, `invariants-bond.yaml` holds invariants. A claim moves between them, keeping its id, when
its state changes. The engine treats their paths as configuration, per F-5, so these names are the
first-instance targets, not hardcoded. *tag* — a durable inline anchor at a claim's single prose home,
holding its stored canonical one-liner; the extraction component (§4) binds tags to claim ids over the
**union** of both corpus files. *view* — the deterministic single-pane emission of tagged claims (§4).

**Component map.** The engine is ONE engine with **two components** over the one claim model, under a
shared preamble: **§1** engine intent + claim model · **§2** shared constraints & solution-shape ·
**§3 Component A** — the claim/invariant validated-factory · **§4 Component B** — the single-pane
extraction view. Each component carries the same five-part skeleton (intent · semantic reqs ·
constraints · acceptance · out-of-scope).

**Label convention — every label is defined in THIS document; none points at an external list.**
**R-**n is a requirement, §3.2 · **A-**n is a precondition the engine must enforce, §2 (A-5 §3.3) ·
**B-**n is a declared trust-boundary assumption, R7 · **D-**n is a Gate-0 delivery precondition, §3.4 ·
**F-**n is an acceptance falsifier, §3.4. **Component B (§4)** uses namespaced counterparts — **RX-**n
requirement · **BX-**n trust-boundary · **DX-**n Gate-0 · **F-X-**n falsifier (the §2 preconditions are
reused unchanged) — kept in a DISTINCT label-space from the factory's because the two share a template
skeleton but NOT a contract: a same-numbered atom means different things in each (e.g. §3.4 `F-8.1` is
orphan-*grounding*; §4.4 `F-X-8.1` is orphan-*tag*).

## 1. Intent — the engine

This engine has **two components over one claim model**: **Component A (§3)** — a validated factory that
creates, graduates, and retires claims under enforced preconditions; and **Component B (§4)** — a
single-pane extraction view that emits those claims deterministically. Both share this intent, the claim
model below, and the constraints of §2.

Two corpus files hold what is conceptually one thing — a **claim** — under independently-evolved,
unaligned conventions, with nothing checking they stay compatible:

- `theory-pipeline.yaml` — live, mutable candidates. **Zero mechanical validation today.**
- `invariants-bond.yaml` — ratified, frozen invariants. Gated only by an opt-in, human-triggered lint.

**Why:** offload structural and mechanical checking off attention and tokens onto **gated machinery** —
a hard-oracle paid once by the machine, forever, instead of a trust-boundary paid by whoever happens to
notice. The motivating incident: a schema-illegal edit to bond's invariant corpus sat undetected for a
session or more, because the one check that could have caught it was opt-in and human-triggered, and
the sibling candidate corpus had no check at all.

**The claim model.** A claim is a **single entity with one stable identity** that persists through its
lifecycle states — *candidate*, then *invariant*. Graduation is a **state transition on that same
entity**, in place, preserving its id — not a new record minted alongside the old one. The two corpus
files are **state-partitions** of the one claim population: a claim lives in the candidate corpus while
under investigation and **moves, keeping its id**, to the invariant corpus when it graduates. A claim
leaves the system only by explicit **retirement**, never by graduation.

## 2. Shared constraints & solution-shape

Constraints and properties that bind **both components**. Grounding: Component B reuses A-1..A-4 and the
solution-shape properties unchanged (§4.3); Component A adds the mutation-specific constraints of §3.3.

**Preconditions the engine must enforce — any violation must HALT the operation with a named cause, and
each gets a seeded corpus case.** A-1 committed-state, running only against a clean tree at a real
commit · A-2 UTF-8 and LF · A-3 claim-core version match · A-4 source-integrity, including mid-run
mutation. *(A-5 cross-file atomicity is factory-specific — it applies only to `graduate` — and lives in
§3.3.)*

**Solution-shape properties — the WHY only.** The HOW — dependency budget, runtime form, size, module
layout — is cairn's `SPECIFICATION.md`, not stated here. *Portable* — retarget to another dyad's
substrate by configuration, not code change, per F-5 · *auditable* — a non-builder dyad can read and
check the engine's logic · *no hidden persistent state* — the engine runs and exits, holding no state
between runs beyond the corpus files themselves · *self-contained* — no third-party runtime
dependencies, so the engine is durable against dependency rot; the concrete runtime is cairn's to
choose.

## 3. Component A — claim/invariant validated-factory

The mutation-and-validation core over the claim corpus: the four operations — validate, create-candidate,
graduate, retire — each halting fail-closed on a violated precondition. Its contract is R1–R7 (§3.2), its
mutation-specific constraints are §3.3, and its acceptance is the F-set (§3.4).

### 3.1 Intent

Realize the engine intent of §1 as an enforced mutation contract: every claim enters and moves through
its lifecycle **only** through this factory's gated operations, so the structural checks §1 wants
offloaded are paid once by the machine, fail-closed, on every create/graduate/retire.

### 3.2 Semantic requirements — bond's Truth, the contract the engine must enforce

- **R1 — claim-core contract.** One field contract every claim satisfies in every state; it is not
  subclassing and not a per-state record type. Shared and always-present: id and global-uniqueness
  scope, a provenance/adjudication shape, and a unified re-examination-trigger vocabulary.
- **R2 — field partition by lifecycle phase, FIXED.** The partition is by *phase of the one claim*, not
  by *separate record*:
  - *claim-core, always:* id, provenance/adjudication, unified re-rub-trigger vocab.
  - *candidate-phase:* grade, coverage, prediction_pair, disposition_history — live and mutable while a
    candidate; frozen as history after graduation.
  - *invariant-phase:* prescription, scope, observability, math, root, grounded_in — absent while a
    candidate; required from graduation onward.
- **R3 — single identity; graduation is a state transition, not a new record.** A claim has ONE stable
  id for its whole life. `graduate` transitions the claim from candidate to invariant state **in
  place**, preserving its id; it never mints a new id or a parallel record. The transition **relocates**
  the claim from the candidate corpus to the invariant corpus, carrying its id and its history.
- **R4 — id-uniqueness and state-partition integrity.** An id names exactly ONE claim for its whole
  life, unique across the entire corpus. A claim occupies exactly ONE state at a time, so its id appears
  in exactly ONE corpus file; the same id present in BOTH files is a corruption and must halt.
- **R5 — removal is retirement only, never graduation.** Graduation changes a claim's state; it never
  removes the claim. The only removal is **retire** — an explicit dyad disposition taken when a claim is
  no longer relevant to the dyad. Retirement is refused, or must cascade explicitly, when another live
  claim still grounds on the retiring claim: a live grounding edge must never be left pointing at a
  removed claim.
- **R6 — graduation mutation ruleset.** On `graduate`: carry claim-core fields through unchanged;
  **freeze the candidate-phase fields as retained history** — they record how the claim earned invariant
  status and are no longer mutated, they are not discarded; and **require every invariant-phase field**
  to be supplied at graduation, HALTING fail-closed if any required one is absent or unresolved. The
  engine freezes the historical part and demands the semantic invariant-phase content, which is
  bond-authored; it never invents invariant-phase content.
- **R7 — declared trust boundary.** Every run DECLARES its assumed semantic preconditions, B-1..B-4
  below; the *truth* of them is bond's discipline, never mechanically verified from inside the engine.
  - B-1 claim-core completeness — every claim is created through the engine, none hand-authored around
    it · B-2 statement/one-liner fidelity · B-3 graduation-readiness and retirement-relevance judgment —
    bond's human disposition, never the engine · B-4 no un-managed claim drifting outside the two corpus
    files.

### 3.3 Constraints — the mutation-specific set

Beyond the shared preconditions of §2, the factory's mutating operations add — each HALTing on a named
cause with a seeded corpus case, as §2:

**A-5 — cross-file atomicity.** `graduate` relocates a claim across two files, so the removal from
the candidate corpus and the insertion into the invariant corpus must be atomic, both or neither; a
graduating claim is never left in both files or in neither. *Atomicity is evaluated at the committed-state
(git) boundary, not the filesystem instant — the two writes land as one commit or not at all; a
hard-interrupt torn working tree is permitted **iff** it fail-closed-halts (A-1/F-7.1) and is
`git restore`-recoverable to the single consistent state (claim in candidate), the engine holding no
transaction journal. The doubled phrasing above names the two forbidden **resting** states — the claim
duplicated (both) or lost (neither); both are refutations, and `neither` is not an acceptable rollback.*

**Recovery — bond states the intent; the mechanism is cairn's.** After any mid-run failure the corpus
must be left at, or recoverable to, its last known-good committed state, with no partially-applied state
persisting.

**Concurrency — bond states the intent; the mechanism is cairn's.** Ids must remain collision-free under
concurrent claim-creation on isolated branches. Bond requires the property; the id-generation and
merge-resolution strategy is cairn's to architect.

**Enforceability — bond states the intent; the plumbing is cairn/swe's.** A `validate` failure must be
able to BLOCK a change from landing, not merely report it. The motivating drift survived precisely
because the only check was opt-in and human-triggered; a check that can only be run by hand does not
close the requirement.

### 3.4 Acceptance criteria — the F-set

The `done_when`. NON-NEGOTIABLE. Each atom resolves to exactly one of {MET | REFUTED | UNVERIFIED}
(UNVERIFIED until an observed run-record exists per D-3). Each atom below carries its breach-condition:
observing the breach ⇒ REFUTED unless MET.

#### F-1.1: In-Memory Determinism
Breach ⇒ REFUTED: two runs over identical in-memory input differ ≥1 byte.

#### F-1.2: SHA-Determinism
Breach ⇒ REFUTED: two runs over identical source shas differ ≥1 byte.

#### F-2.1: Malformed-Field Halt
Breach ⇒ REFUTED: a schema-invalid field on `new`/`graduate` does not halt.

#### F-2.2: Dup-ID Halt
Breach ⇒ REFUTED: a new claim id colliding with an existing id in EITHER file does not halt.

#### F-2.3: Missing-Source Halt
Breach ⇒ REFUTED: a missing/unreadable corpus file does not halt.

#### F-3: Atomicity Guard
Breach ⇒ REFUTED: an interrupted `graduate` leaves the claim in both files or in neither.

#### F-4: No Semantic Drift
Breach ⇒ REFUTED: any engine-written field ≠ what was validated pre-write.

#### F-5: Portability
Breach ⇒ REFUTED: a second dyad's substrate needs code changes, not config, to retarget.

#### F-6: Trust-Boundary Declaration
Breach ⇒ REFUTED: a run's output omits its Class-B assumptions declaration.

#### F-7.1: Dirty-Tree Halt
Breach ⇒ REFUTED: a dirty tree does not halt.

#### F-7.2: Encoding/EOL Halt
Breach ⇒ REFUTED: CRLF or encoding drift does not halt.

#### F-7.3: Schema-Version Halt
Breach ⇒ REFUTED: a claim-core schema-version mismatch does not halt.

#### F-7.4: Mid-Run TOCTOU Halt
Breach ⇒ REFUTED: a mid-run source mutation does not halt.

#### F-8.1: Orphan-Grounding Halt
Breach ⇒ REFUTED: a `grounded_in` edge pointing to a non-existent claim id does not halt.

#### F-8.2: Split-State Halt
Breach ⇒ REFUTED: the same claim id present in BOTH corpus files does not halt.

#### F-8.3: Double-Graduation Halt
Breach ⇒ REFUTED: graduating a claim already in the invariant state does not halt.

#### F-8.4: Retire-Orphan Halt
Breach ⇒ REFUTED: retiring a claim that a live claim still grounds on does not halt.

**Gate-0 — checked BEFORE any F-atom.**

#### D-1: Operations Invokable
The four operations — validate, create-candidate, graduate, and retire — are invokable and run to
completion.

#### D-2: Seeded Malformation Corpus
A seeded malformation corpus for every F-atom, including the two-file and atomicity cases.

#### D-3: Per-Atom Observed Run-Record
`atom → command → observed exit/output`, not a summary attestation.

#### D-4: Resolved Provenance
Resolved pinned provenance of the deliverable.

### 3.5 Out of scope for the builder — stays bond's, permanently

Authoring claim content (e.g. statement, one_liner, prescription) · deciding when a claim is READY to
graduate, and when a claim is no longer RELEVANT to retire — both human dispositions by bond; the engine
executes a graduation or retirement bond has disposed and never judges readiness or relevance · the
claim-core field-list's semantic content, which bond ratifies and the builder consumes. Not a
precondition/viability-edge relation between claims — that is a distinct, not-yet-built edge, out of
scope here · not a proposal to any shared library.

## 4. Component B — the single-pane extraction view

The engine also answers a second standing requirement over the same corpus: a **deterministic
single-pane view** of the dyad's claims across its prose artifacts, so the Operator can evaluate the
Agent's evaluations. Per the scope correction — **one** engine; `commission-invariant-engine`'s extractor
is a **component**, not a peer commission — the extractor's requirements fold in **here**, and that
repo's placeholder `REQUIREMENTS.md` is **subsumed and retired** by this component.

**Provenance — learning, not obligation.** A working extractor was built and delivered **F-green** under
the 2026-06-17 two-party commission (cairn, Builder), against a *single* structural sidecar
(`invariants-bond.structure.yaml`). That effort enters here as **proven learning that de-risks** — its
behavioral contract is known-satisfiable — **not** as an artifact this (not-yet-built) engine is obliged
to reuse as-is. This component states **target behavior**; whether the built code is reused, reworked, or
rewritten under it is the Architect/Builder's call (§4.5). The single-identity / two-corpus model (§1,
R3–R4) is a **material change** from the pilot's single-sidecar world, so its behavior is **new scope**,
and its acceptance atoms are marked below **new** vs **green** to keep the attestation honest.

### 4.1 Intent — placed-and-bounded non-determinism

Full mechanical semantic extraction is impossible — somewhere a judgment says "this sentence is a claim."
The component does not eliminate that non-determinism; it **places and bounds** it: the semantic act —
**tagging a claim at its single prose home** — happens ONCE, Operator-gated, at source; everything
downstream (scan → collect → validate → emit) is **deterministic**. Same input → **byte-identical** view.
The motivating requirement, falsified in the pilot: an agent-pass extraction makes non-deterministic
selection/compression/grouping calls per entry, so two runs differ — a measurement instrument that reads
differently on identical input is not an instrument, and it contaminates every downstream evaluation.

### 4.2 Semantic requirements — the extraction contract (RX-n)

- **RX1 — deterministic extraction.** Scan a configured source-list of prose artifacts; collect tagged
  claim-blocks; emit a view (header: source-sha pins + generation stamp; body: grouped claim one-liners +
  source refs). Same input → byte-identical output. Idempotent. No network, no LLM call at runtime.
- **RX2 — bind tags to the UNIFIED claim view (NEW — supersedes the pilot's single sidecar).** Under the
  single-identity model, a claim's id lives in exactly ONE of the two state-partition corpora at a time
  (`theory-pipeline.yaml` candidates ∪ `invariants-bond.yaml` invariants, per R3–R4). A prose tag binds
  to a claim id resolved against the **union of both corpora** — not against a separate structural
  sidecar. The pilot's merge (`view = extract(md) ⊕ sidecar`, keyed on one file) does **not** carry over;
  the union-read is the component's defining rework.
- **RX3 — graduation-stable linkage (NEW).** Because a claim **relocates** between corpora on graduation
  (R3), keeping its id, a tag's binding must **survive the state transition**: a claim tagged in prose
  must not break its linkage when it moves `theory-pipeline.yaml` → `invariants-bond.yaml`. The union-view
  is what makes this hold — the tag binds to the *id*, resolvable in whichever corpus currently holds it.
- **RX4 — one-liner fidelity / no drift.** The inline tag carries **only** the stored canonical one-liner
  (authored at ratification, never regenerated); the claim's content single-homes at its prose source. Any
  emitted one-liner ≠ its stored source one-liner is a breach.
- **RX5 — declared trust boundary (extraction B-set, BX-1..BX-4).** Every emitted view DECLARES its
  assumed semantic preconditions — **BX-1** tagging-completeness (tagged = the whole claim class),
  **BX-2** one-liner fidelity, **BX-3** single-home integrity (no untagged paraphrase drifting), **BX-4**
  status truthfulness — whose *truth* is bond's discipline, never mechanically verified from inside
  (mirrors R7). An **untagged-candidate** heuristic scan MAY emit a candidate queue as Generate's inbox;
  it never blocks emission.

### 4.3 Constraints

The §2 precondition set applies to extraction runs **unchanged** — A-1 committed-state, A-2 UTF-8/LF,
A-3 version-match, A-4 source-integrity/TOCTOU — with one extension: source-integrity now pins **both**
corpus files' shas (the view reads the union, not one sidecar), checked before AND after scan. The §2
solution-shape properties — portable-by-config (F-5), auditable, no hidden persistent state,
self-contained/stdlib — apply identically. *(A-5 atomicity does not apply — extraction is read-only.)*

### 4.4 Acceptance criteria — the extraction F-set (F-X-n)

**Namespaced `F-X-` on purpose** (see §0): the factory F-set (§3.4) and this set are **template-fill
twins** — same skeleton, *different contracts* — so they stay in distinct label-spaces. Each atom
resolves to exactly one of {MET | REFUTED | UNVERIFIED} (UNVERIFIED until an observed run-record exists
per DX-3). Each atom carries its breach-condition (observing the breach ⇒
REFUTED unless MET) and a **prior** marker: **`green`** marks an atom the 2026-06-17 delivery already
passed *for the single-sidecar architecture* — a **de-risking prior, NOT a pass for the reworked
union-view component** (re-run required); **`new`** marks union-view / single-identity scope with no
prior validation.

#### F-X-1.1: Fn-Determinism
Breach ⇒ REFUTED: two runs over identical in-memory source differ ≥1 byte. *(prior: green)*

#### F-X-1.2: SHA-Determinism
Breach ⇒ REFUTED: two runs over identical source shas differ ≥1 byte. *(prior: green)*

#### F-X-2.1: Unclosed-Tag Halt
Breach ⇒ REFUTED: an unclosed tag does not halt. *(prior: green)*

#### F-X-2.2: Dup-Tag-ID Halt
Breach ⇒ REFUTED: a duplicate prose tag id does not halt. *(prior: green)*

#### F-X-2.3: Missing-Source Halt
Breach ⇒ REFUTED: a missing/unreadable source does not halt. *(prior: green)*

#### F-X-3: Staleness Guard
Breach ⇒ REFUTED: source mutated post-emit; the guard fails to arm. *(prior: green — extend to two-corpus pins)*

#### F-X-4: No One-Liner Drift
Breach ⇒ REFUTED: any emitted one-liner ≠ its stored source one-liner. *(prior: green)*

#### F-X-5: Portability
Breach ⇒ REFUTED: a second dyad's substrate needs code, not config. *(prior: green)*

#### F-X-6: Trust-Boundary Header
Breach ⇒ REFUTED: an emitted view omits its BX-1..BX-4 declaration. *(prior: green)*

#### F-X-7.1: Dirty-Tree Halt
Breach ⇒ REFUTED: a dirty tree does not halt. *(prior: green)*

#### F-X-7.2: Encoding/EOL Halt
Breach ⇒ REFUTED: CRLF/encoding drift does not halt. *(prior: green)*

#### F-X-7.3: Grammar-Version Halt
Breach ⇒ REFUTED: a tag-grammar version mismatch does not halt. *(prior: green)*

#### F-X-7.4: Mid-Scan TOCTOU Halt
Breach ⇒ REFUTED: a mid-scan source mutation does not halt. *(prior: green)*

#### F-X-8.1: Orphan-Tag Halt
Breach ⇒ REFUTED: a tag id present in **neither** corpus does not halt. *(prior: **new** — was "no sidecar entry")*

#### F-X-8.2: Split-State Halt
Breach ⇒ REFUTED: a tag id present in **both** corpora (an R4 corruption) does not halt. *(prior: **new**)*

#### F-X-8.3: Graduation-Linkage
Breach ⇒ REFUTED: a claim's tag linkage breaks across its graduation relocation. *(prior: **new**)*

#### F-X-8.4: Cross-Home-Dup Halt
Breach ⇒ REFUTED: the same id in >1 prose tag does not halt. *(prior: green)*

**Gate-0 (component) — checked before any F-X atom.**

#### DX-1: Runnable Extract Entrypoint
A runnable `extract` entrypoint.

#### DX-2: Seeded Corpus
A seeded corpus for every F-X atom, **including** the union-view and graduation-relocation cases the
pilot's single-sidecar scope never needed.

#### DX-3: Per-Atom Observed Run-Record
`atom → command → observed exit/output`.

#### DX-4: Resolved Provenance
Resolved pinned provenance of the deliverable.

### 4.5 Out of scope for the builder — extraction

Authoring tags/one-liners · deciding what is a claim/invariant · the candidate-queue triage ·
conflict-detection over the extracted set — all bond's, permanently (mirrors §3.5). **And, explicitly: this
component does not mandate reuse of the built `invariant_extractor.py`.** It states target behavior; the
built artifact is **proven learning, not a required dependency** — reuse, rework (the union-view adapter),
or rewrite is the Architect/Builder's disposition under these requirements. `commission-invariant-engine`
retires as a separate repo once this lands; the migration/retirement *mechanics* are gate #11 / the
Architect's fold proposal, not this component's.
