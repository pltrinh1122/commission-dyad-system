# REQUIREMENTS: Dyad System Claim/Invariant Factory

## 1. The Philosophical Intent
A deterministic mechanical factory is required to manage claims across two distinct corpora (`theory-pipeline.yaml` and `invariants-bond.yaml`). 

## 2. The Core Invariants
* **The Lineage Edge:** Graduation of a claim must enforce a physical lineage edge. Transitioning candidates to an archive state is required; deletion is structurally forbidden.
* **The Atomic Guarantee:** Cross-file mutation must be atomic. A split-brain state (success in one corpus, failure in the other) is an intolerable violation and must be prevented via guaranteed mechanical rollback.
* **The Isolation Constraint:** Execution must rely exclusively on the Python 3 standard library to prevent dependency rot.
