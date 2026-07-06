---
commission: dyad-system-engine
title: claim/invariant validated-factory engine
owner: dyad-bond                  # this file is bond's Truth layer
status: draft                     # draft -> ratified by the Operator
model: single-identity-claim      # a claim is ONE entity that persists through its lifecycle states
supersedes:                       # bond-internal; not required to read this document
  spec: dyad-bond commission spec
  pin: c736f4b
  scope: adopts the single-identity claim model; supersedes the earlier two-record framing and resolves the three points a cairn review falsified
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
first-instance targets, not hardcoded.

**Label convention — every label is defined in THIS document; none points at an external list.**
**R-**n is a requirement, §2 · **A-**n is a precondition the engine must enforce, §3 · **B-**n is a
declared trust-boundary assumption, R7 · **D-**n is a Gate-0 delivery precondition, §4 · **F-**n is an
acceptance falsifier, §4.

## 1. Intent — what and why

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

## 2. Semantic requirements — bond's Truth, the contract the engine must enforce

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

## 3. Constraints

**Preconditions the engine must enforce — any violation must HALT the operation with a named cause, and
each gets a seeded corpus case.** A-1 committed-state, running only against a clean tree at a real
commit · A-2 UTF-8 and LF · A-3 claim-core version match · A-4 source-integrity, including mid-run
mutation · A-5 cross-file atomicity — `graduate` relocates a claim across two files, so the removal from
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

**Solution-shape properties — the WHY only.** The HOW — dependency budget, runtime form, size, module
layout — is cairn's `SPECIFICATION.md`, not stated here. *Portable* — retarget to another dyad's
substrate by configuration, not code change, per F-5 · *auditable* — a non-builder dyad can read and
check the engine's logic · *no hidden persistent state* — the engine runs and exits, holding no state
between runs beyond the corpus files themselves · *self-contained* — no third-party runtime
dependencies, so the engine is durable against dependency rot; the concrete runtime is cairn's to
choose.

## 4. Acceptance criteria — the F-set

The `done_when`. NON-NEGOTIABLE. Each atom is binary: {MET | REFUTED | UNVERIFIED}.

| atom | breach-condition ⇒ REFUTED unless MET |
|---|---|
| F-1.1 in-memory determinism | two runs over identical in-memory input differ ≥1 byte |
| F-1.2 sha-determinism | two runs over identical source shas differ ≥1 byte |
| F-2.1 malformed-field halt | a schema-invalid field on `new`/`graduate` does not halt |
| F-2.2 dup-id halt | a new claim id colliding with an existing id in EITHER file does not halt |
| F-2.3 missing-source halt | a missing/unreadable corpus file does not halt |
| F-3 atomicity guard | an interrupted `graduate` leaves the claim in both files or in neither |
| F-4 no semantic drift | any engine-written field ≠ what was validated pre-write |
| F-5 portability | a second dyad's substrate needs code changes, not config, to retarget |
| F-6 trust-boundary declaration | a run's output omits its Class-B assumptions declaration |
| F-7.1 dirty-tree halt | a dirty tree does not halt |
| F-7.2 encoding/EOL halt | CRLF or encoding drift does not halt |
| F-7.3 schema-version halt | a claim-core schema-version mismatch does not halt |
| F-7.4 mid-run TOCTOU halt | a mid-run source mutation does not halt |
| F-8.1 orphan-grounding halt | a `grounded_in` edge pointing to a non-existent claim id does not halt |
| F-8.2 split-state halt | the same claim id present in BOTH corpus files does not halt |
| F-8.3 double-graduation halt | graduating a claim already in the invariant state does not halt |
| F-8.4 retire-orphan halt | retiring a claim that a live claim still grounds on does not halt |

**Gate-0, checked BEFORE any F-atom.** D-1 the four operations — validate, create-candidate, graduate,
and retire — are invokable and run to completion · D-2 seeded malformation corpus for every F-atom,
including the two-file and atomicity cases · D-3 per-atom OBSERVED run-record — `atom → command →
observed exit/output`, not a summary attestation · D-4 resolved pinned provenance of the deliverable.

## 5. Out of scope for the builder — stays bond's, permanently

Authoring claim content (e.g. statement, one_liner, prescription) · deciding when a claim is READY to
graduate, and when a claim is no longer RELEVANT to retire — both human dispositions by bond; the engine
executes a graduation or retirement bond has disposed and never judges readiness or relevance · the
claim-core field-list's semantic content, which bond ratifies and the builder consumes. Not a
precondition/viability-edge relation between claims — that is a distinct, not-yet-built edge, out of
scope here · not a proposal to any shared library.
