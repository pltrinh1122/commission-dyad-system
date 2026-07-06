---
title: Commission Artifact Standards
commission: commission-protocol
owner: dyad-cairn
---

# Commission Artifact Standards

## Document Purpose

This document establishes the machine-readable structural standards for artifacts governed by the Dyad Commission Protocol. It defines the formal API contracts for `REQUIREMENTS.md` (the Commission-Request) and `SPECIFICATION.md` (the Commission-Delivery) to ensure deterministic parsing by automated test harnesses and Agent executors.

## Related Documents

- `Aligns with` [pltrinh1122/standards — DOCUMENTATION_STANDARDS.md](https://github.com/pltrinh1122/standards/blob/main/DOCUMENTATION_STANDARDS.md)
- `Aligns with` [IEEE 830 — Recommended Practice for Software Requirements Specifications](https://standards.ieee.org/standard/830-1998.html)
- `Extends` Commission Protocol: The Commissionee (dyad-system)

## 1. Intent

**Aggregator Section Summary:** This section outlines the overarching rationale for upgrading unstructured documents into deterministic API contracts.

While generic documentation standards govern human readability, the Dyad Commission Protocol explicitly requires structural determinism between dyads. This standard converts natural language requests and specifications into mathematically verifiable topologies. By adhering to these standards, an Agent or CI harness CAN programmatically prove that no scope was dropped between a Request and its Delivery.

## 2. Minimalist Frontmatter Constraints

**Aggregator Section Summary:** This section defines the lean, operationally necessary YAML frontmatter fields for commission artifacts, explicitly waiving bloated upstream standards.

To prevent tracking overlap with Git history (dates, versions, authors) and CI state (review status), Commission Artifacts MUST reject generic metadata standards and ONLY include fields critical for deterministic execution.

#### CAS-1.1: H1 Title Match
The `title` frontmatter field MUST match the document H1 exactly.

#### CAS-1.2: Commission Identifier
The `commission` frontmatter field MUST define the target system identifier (e.g., `dyad-system-engine`).

#### CAS-1.3: Ownership Declaration
The `owner` frontmatter field MUST declare the dyad identity responsible for the artifact (e.g., `dyad-cairn`).

#### CAS-1.4: Scope Freeze Pinning (Deliveries Only)
To enforce the "Scope Freeze" invariant of the Commission Protocol, the Delivery MUST explicitly link the upstream source file and the exact Git SHA pin via the `grounds_on` object (e.g., `file: REQUIREMENTS.md`, `pin: 6c3fc6d`).

#### CAS-1.5: Metadata Bloat Waiver
Generic semantic tags (`document_type`, `domain`, `concepts`, `technologies`, `last_updated`, `version`) mandated by upstream standards ARE EXPLICITLY WAIVED as they provide no mechanical value to the test harnesses and overlap with version control properties.

## 3. Topological Mirroring

**Aggregator Section Summary:** This section enforces the 1:1 structural correspondence between the upstream request and downstream delivery.

#### CAS-2.1: Requirement Identifier Atoms
Every constraint, falsifier, or atom defined in the Commission-Request MUST possess a unique, alphanumeric identifier (e.g., `F-1.1`, `A-3`).

#### CAS-2.2: Delivery Mapping Isomorphism
The Commission-Delivery MUST topologically map every single identifier defined in the Request.

#### CAS-2.3: Explicit Omission
Silent omissions are FORBIDDEN. If an atom is rejected or redefined, it MUST still be topologically represented in the Delivery with an explicit justification.

## 4. Deterministic Anchoring

**Aggregator Section Summary:** This section defines the use of markdown ATX headings to generate predictable HTML anchors.

#### CAS-3.1: ATX Headings for Atoms
When defining a specific verifiable atom or mechanism, the document MUST use ATX headings (e.g., `#### F-X-8.1: Orphan Tag Halt`) rather than bolded list items or table cells.

#### CAS-3.2: Cross-Referencing
This mandate ensures the Markdown parser natively generates predictable, deterministic HTML anchors (e.g., `#f-x-81-orphan-tag-halt`).

#### CAS-3.3: Targetability
Downstream builders (`dyad-swe`) and automated harnesses MUST rely on these anchors to deterministically link commits, pull requests, and test failures to explicit specification boundaries.

## 5. Artifact vs. Transactional Payload Decoupling

**Aggregator Section Summary:** This section explicitly decouples static architecture from dynamic execution tracking.

#### CAS-4.1: No Run-Records in Artifacts
`REQUIREMENTS.md` and `SPECIFICATION.md` MUST remain purely static architectural contracts. They MUST NOT contain dynamic test outputs, `[MET]`/`[REFUTED]` state tracking, or CI/CD run-records.

#### CAS-4.2: Transactional Tracking
All dynamic execution run-records MUST be isolated to the transactional layer (e.g., the `DYAD_LEDGER.md`, Direct Message payloads to `dyad-bond`, or native CI/CD logs). Inserting dynamic execution states into the static artifacts is FORBIDDEN.
