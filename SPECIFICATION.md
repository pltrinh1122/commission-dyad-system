---
commission: dyad-system-engine
title: Architectural Specification — claim/invariant validated-factory engine
owner: dyad-cairn
status: proposed
model: single-identity-claim
grounds_on:
  file: REQUIREMENTS.md
  pin: d5a1727
---

# SPECIFICATION — dyad-system: claim/invariant validated-factory engine

> **Cairn's Architectural Map.** This document mechanically implements the single-identity Model B contract.
> 
> *Note: All alphanumerical constraints and falsifiers (e.g., A-1, F-3, F-7.1) map directly to their definitions in [REQUIREMENTS.md](./REQUIREMENTS.md).*

## 0. Terms & label index

**Domain mapped to filesystem objects.** *candidate corpus* maps physically to `theory-pipeline.yaml` (a flat YAML array). *invariant corpus* maps physically to `invariants-bond.yaml` (a flat YAML array). A flat YAML array natively enforces layout consistency and surfaces concurrency edits as explicit Git conflicts.

## 1. Intent — what and why

**The engine execution boundary.** The machinery operates as a stateless CLI tool executing locally on a Git working tree. The state transitions are executed by the machinery, but the transaction boundary and audit trail rely entirely on the Git substrate.

## 2. Semantic requirements — bond's Truth, the contract the engine must enforce

- **R1 / R2 — claim-core & field partition.** The schemas strictly map to the phase definition. The engine evaluates schema conformance per state (A-3).
  - *claim-core*: `id` (UUIDv4), `schema_version` (1.0), `provenance` (string), `re_rub_triggers` (list of strings).
  - *candidate-phase*: `grade`, `coverage`, `prediction_pair`, `disposition_history`.
  - *invariant-phase*: `prescription`, `scope`, `observability`, `math`, `root`, `grounded_in`.
- **R3 — single identity.** The `graduate` operation reads the target UUID from `theory-pipeline.yaml`, combines the history fields with the CLI-supplied invariant fields, and appends the object to `invariants-bond.yaml`. The identity remains identical.
- **R4 — id-uniqueness.** `UUIDv4` generation inherently guarantees mathematical collision-resistance across isolated branches, satisfying the invariant without a centralized counter.
- **R5 — removal is retirement only.** The `graduate` operation structurally lacks removal capabilities. `retire` acts as the exclusive deletion path.
- **R6 — graduation mutation ruleset.** The `graduate` command takes `--invariant-data <file.yml>` as input. It copies candidate fields directly to the target schema (freezing them) and populates the invariant schema from the explicitly supplied data file. The engine generates no semantic strings.
- **R7 — declared trust boundary.** The engine CLI outputs the B-1..B-4 assumptions to stdout upon completion (F-6).

## 3. Constraints

**Preconditions the engine must enforce.**
- **A-1 committed-state / F-7.1 dirty-tree halt.** The engine checks `git status --porcelain`. A non-empty return triggers an immediate `HALT`.
- **A-2 UTF-8 and LF / F-7.2.** The file stream parser validates encoding and line endings prior to mutation.
- **A-3 / F-7.3 schema version.** Hardcoded `schema_version` check on ingest.
- **A-4 / F-7.4 mid-run TOCTOU.** The engine caches file modification timestamps on read, verifying them prior to the write flush.
- **A-5 cross-file atomicity (The F-3 / §3 resolution).** The engine executes physical writes to both files sequentially. If interrupted mid-write (e.g., SIGKILL), the filesystem is torn. This structurally dirties the Git tree. On any subsequent consumption attempt, the A-1/F-7.1 `HALT` explicitly blocks parsing of the torn state. The Operator executes `git restore .` to revert to the last known-good committed state. This explicitly guarantees the claim is NEVER left in "neither" or "both" consumed states.

**Recovery / Concurrency / Enforceability / Solution-shape.** 
- **Concurrency:** `UUIDv4` ensures IDs do not collide on independent branches.
- **Dependency budget:** Written in zero-dependency standard libraries (e.g., Python `yaml` / `json` built-ins). No external databases or hidden persistent state (`.sqlite` / `.log` files).

## 4. Acceptance criteria — the F-set

**The execution primitive mappings (D-1).**
| operation | execution mapping |
|---|---|
| `validate` | Reads both corpus files. Scans for duplicate UUIDs across the entire corpus (F-2.2, F-8.2). Scans for `grounded_in` edges pointing to nonexistent UUIDs (F-8.1). Verifies schema compliance (F-2.1). |
| `create-candidate` | Mints UUIDv4, appends input fields to `theory-pipeline.yaml`. |
| `graduate` | Relocates candidate to `invariants-bond.yaml`, applies R6 field freeze + invariant demand. F-8.3 checked prior to write. |
| `retire` | Scans corpus for inbound `grounded_in` edges (F-8.4). Unlinks and deletes the target UUID. |

**Testing (D-2, D-3, D-4):** Automated bash harnesses paired with malformed corpus fixtures simulate each F-set breach and capture the exact `HALT` exit codes.

## 5. Out of scope for the builder — stays bond's, permanently

The engine evaluates structural boundaries only. It strictly lacks capabilities to judge claim readiness, author human descriptions, or determine obsolescence.
