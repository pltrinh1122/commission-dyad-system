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

> **Cairn's Architectural Map.** This document defines the structural schemas, layout, id-generation, and mechanical execution logic required to implement the single-identity Model B contract defined in `REQUIREMENTS.md`.
> 
> *Note: All alphanumerical constraints and falsifiers (e.g., A-1, F-3, F-7.1) referenced in this document map directly to their definitions in [REQUIREMENTS.md](./REQUIREMENTS.md).*

## 1. Corpus Structure & Layout

The corpus consists of two flat YAML files stored in the repository root (or configurable directory path).
To satisfy portability (F-5) and auditable layout, the files are standard YAML lists of claim objects.

- **`theory-pipeline.yaml`**: A YAML array `[]` of candidate-phase claims.
- **`invariants-bond.yaml`**: A YAML array `[]` of invariant-phase claims.

A YAML array layout inherently enforces order and prevents silent overwriting during merge conflicts, natively elevating concurrent modifications to explicit Git conflicts when necessary.

## 2. ID-Generation Strategy (Concurrency & Collisions)

**Mechanism:** `UUIDv4` (Universally Unique Identifier).
**Rationale:** To satisfy Section 3's concurrency constraint ("collision-free under concurrent claim-creation on isolated branches") without maintaining hidden persistent state or centralized counters, all claims will be minted with a randomly generated 128-bit UUIDv4 (e.g., `id: 550e8400-e29b-41d4-a716-446655440000`). This guarantees mathematical collision resistance across isolated branches.

## 3. Claim Schema

Every claim in the corpus follows a strict schema partition.

**Claim-Core Fields (Always Present)**
* `id` (string): The UUIDv4.
* `schema_version` (string): e.g., "1.0", to satisfy A-3 / F-7.3.
* `provenance` (string): Origin of the friction.
* `re_rub_triggers` (list of strings): The adjudication vocabulary.

**Candidate-Phase Fields (Live while candidate; Frozen history upon graduation)**
* `grade` (string)
* `coverage` (string)
* `prediction_pair` (string)
* `disposition_history` (list of strings)

**Invariant-Phase Fields (Absent in candidate; Required upon graduation)**
* `prescription` (string)
* `scope` (string)
* `observability` (string)
* `math` (string)
* `root` (boolean/string)
* `grounded_in` (list of strings): Must reference valid `id`s within the corpus (F-8.1).

## 4. Operational Mechanics

The engine exposes four mechanical CLI operations (D-1):

### A. `validate`
**Function:** A stateless validator ensuring semantic and structural integrity across the corpus.
**Enforcements:**
1. A-1 / F-7.1: Halts if `git status --porcelain` is not empty (dirty tree halt).
2. A-2 / F-7.2: Halts on non-UTF-8 or CRLF line endings.
3. F-2.2 / F-8.2: Scans both files. Halts if ANY duplicate `id` is found (either within one file or across both).
4. F-8.1: Validates all `grounded_in` edges point to an `id` that exists in the corpus.
5. F-2.1: Validates field schemas strictly matching Section 3.

### B. `create-candidate`
**Function:** Mints a new candidate.
**Execution:**
- Takes candidate-phase fields as input (e.g., via a `.yml` template file or flags).
- Generates a new `UUIDv4`.
- Appends the candidate object to `theory-pipeline.yaml`.
- Engine validates the new corpus state in memory before writing to disk.

### C. `graduate`
**Function:** Relocates a candidate to an invariant, freezing history.
**Execution:**
- Takes a target `id` and an `--invariant-data <file.yml>` containing the bond-authored invariant fields.
- Checks if the claim is already in `invariants-bond.yaml` (F-8.3 double-graduation halt).
- Removes the claim from `theory-pipeline.yaml`.
- Appends the candidate fields (now frozen history) and the new invariant fields to the claim.
- Appends the updated claim to `invariants-bond.yaml`.
- **Atomicity Guard (F-3 vs §3):** The engine writes to disk. If interrupted mid-write via SIGKILL, the files are physically torn on disk. However, the engine inherently enforces a clean tree precondition (A-1). The torn tree is dirty. On the next run (or any consumption attempt), `validate` immediately halts (F-7.1), explicitly trapping the torn state. The Operator executes `git restore .` to revert to the last known-good committed state, satisfying Section 3's "recoverable to" mandate and mathematically guaranteeing the claim is never "lost" (in neither file).

### D. `retire`
**Function:** Permanently deletes a claim.
**Execution:**
- Takes a target `id`.
- Scans the entire corpus to ensure no live claim's `grounded_in` field points to this `id` (F-8.4 retire-orphan halt).
- Removes the claim from whichever file it resides in.
