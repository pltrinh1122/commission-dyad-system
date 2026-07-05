---
repo: commission-dyad-system
kind: commission-quarry
form: commissioning-protocol      # the Commons form for commission quarries
form_status: proposed             # cairn-authored; headed to the Commons via dyad-steward
form_ref: github.com/pltrinh1122/dyad-cairn/kb/HOW-commission.md
topology: neutral-quarry          # standalone-repo topology fixed by the protocol
status: draft                     # requirements being populated; not yet locked
parties:
  requirements: dyad-bond
  tests: dyad-bond
  specification: dyad-cairn
  build: dyad-swe
  acceptance: dyad-bond
layout:
  REQUIREMENTS.md: dyad-bond       # intent, semantic requirements, acceptance falsifiers
  tests/: dyad-bond                # seeded malformation corpus
  SPECIFICATION.md: dyad-cairn     # structural design
  src/: dyad-swe                   # the engine
---

# commission-dyad-system

The commission quarry for the **dyad-system claim/invariant validated-factory engine**. This is a
standalone repository — not a document passed between dyads, and not a subfolder inside any one dyad's
own repo.

## Form — the Commissioning Protocol

This repo's structure is not ad-hoc; it conforms to the **Commissioning Protocol**, the Commons form
for commission quarries, defined in `github.com/pltrinh1122/dyad-cairn/kb/HOW-commission.md`. The
protocol fixes the standalone-repo topology, the role-accountability split, and the Issue → Spec-Rub →
PR → Merge pipeline. It does *not* mandate a fixed file tree or front-matter — it names artifacts by
example — so the layout and metadata here are this commission's own instantiation. The protocol is
cairn-authored and headed to the Commons via dyad-steward, so the form tracks a draft.

Under its topology a commission is a dedicated, standalone Git repository, decoupling each dyad's
*agent state* — its own repo and reasoning — from the *project state* held here, so participating dyads
collaborate concurrently with no dyad routing pull requests for the others.

## Roles — per the Commissioning Protocol

Accountability is split by discipline. For this commission:

- **Commissioner — dyad-bond**, the Philosopher, accountable for **Truth**. Hands off the raw friction
  and semantic intent; owns `REQUIREMENTS.md` and the test corpus. The *Operator* is bond's human half,
  the ratifier of bond's dispositions.
- **Prime-Commissionee — dyad-cairn**, the Architect, accountable for the **Map**. Receives the
  commission directly; owns `SPECIFICATION.md`.
- **Sub-Commissionee — dyad-swe**, the Builder, accountable for the **Vehicle**. Receives a delegated
  commission from cairn; owns `src/`.

## Layout — this commission's instantiation

The protocol does not mandate a fixed tree; this commission instantiates the layout below, each owner
writing only its own paths.

| path | owner | holds |
|---|---|---|
| `REQUIREMENTS.md` | Commissioner / dyad-bond | intent, semantic requirements, acceptance falsifiers |
| `tests/` | Commissioner / dyad-bond | the seeded malformation corpus that exercises the falsifiers |
| `SPECIFICATION.md` | Prime-Commissionee / dyad-cairn | the structural design satisfying the requirements |
| `src/` | Sub-Commissionee / dyad-swe | the engine satisfying the specification |

## Workflow — Issue → Spec-Rub → PR → Merge

Project communication is confined to this repo's GitHub Issues; local DMs are deprecated for project
execution. Every mutation after the Genesis Commit follows one pipeline:

1. **Solicit** — an Issue defines the required state transition or artifact mutation.
2. **Falsification** — the assigned dyad posts a formal Falsification report in the Issue comments,
   exposing contradictions before any code.
3. **Execution** — the assigned dyad authors the change on an isolated branch and opens a PR stating
   `Closes #N`.
4. **Anchor** — ratifying the PR merges the artifact to `main` and closes the Issue.

The one exception is the **Genesis Commit** — the first addition to the empty repo — pushed directly to
`main` to establish baseline ground truth.

## Status

The Commissioning Protocol is a cairn-authored draft headed to the Commons via dyad-steward. Bond's
requirements here are being populated and are not yet ratified. Start at `REQUIREMENTS.md`.
