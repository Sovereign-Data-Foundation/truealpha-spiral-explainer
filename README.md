# TrueAlphaSpiral Explainer

**TrueAlphaSpiral (TAS)** is a deterministic execution architecture for consequential computation.

> **Capability does not imply authority. Proof must precede consequence.**

This repository explains the architecture by staying anchored to the smallest canonical computational unit already present in the implementation: **TAS_DNA / `TASGene`**.

## One datum

The canonical transition datum is

\[
G_i=(\text{origin},\text{context},\text{authority},\text{operation},\text{parent},\text{invariants},\text{decision},\text{receipt}).
\]

The implementation represents that datum as `TASGene`.

A gene can be `ADMITTED`, `REFUSED`, or `PENDING`, but the grammar of the datum does not change across branches.

## Two projections

The same TAS_DNA chronology supports two different views:

1. **Evidence timeline** — every represented admission and refusal.
2. **Authorized state progression** — admitted transitions only.

Let \(\mathcal G_n\) be the ordered gene chronology.

Every represented decision appends one datum:

\[
\mathcal G_{n+1}=\mathcal G_n\Vert G_i.
\]

For admission:

\[
G_i.d=\mathrm{ADMITTED}
\Rightarrow
\mathcal E_{n+1}=\mathcal E_n\Vert G_i
\land
S_{k+1}=F(S_k,G_i).
\]

For refusal:

\[
G_i.d=\mathrm{REFUSED}
\Rightarrow
\mathcal E_{n+1}=\mathcal E_n\Vert G_i
\land
S_{k+1}=S_k.
\]

That is the central distinction:

> **A refusal is preserved as evidence without becoming the next authorized state.**

The current implementation exposes exactly these two projections through `WakeChain.evidence_timeline()` and `WakeChain.state_sequence()`.

## Why this matters

The explainer previously introduced a composite full-state model of the form

\[
S=(O,\Gamma).
\]

That abstraction created a conceptual mismatch with the upstream implementation, where the documented state lineage advances only through admitted transitions while refusals remain in the evidentiary timeline.

This repository now keeps those semantics separate rather than silently redefining either one.

The canonical object to formalize is **TAS_DNA**. State and evidence are projections over the same lineage-bearing datum.

## Architecture at a glance

```text
Proposal
   |
   v
Verification / Admissibility
   |
   +-------------------+
   |                   |
 ADMITTED             REFUSED
   |                   |
   v                   v
TASGene              TASGene
 decision=ADMITTED    decision=REFUSED
   |                   |
   +---------+---------+
             |
             v
       Evidence timeline
       (both retained)

Authorized state progression:
  advances on ADMITTED
  unchanged on REFUSED
```

## Reading order

1. [TAS_DNA — One Datum, Two Projections](docs/tas-dna.md)
2. [Foundational Axioms](docs/foundational-axioms.md)
3. [Cursive Computation](docs/cursive-computation.md)
4. [Admissibility](docs/admissibility.md)
5. [Canonical Vertical Slice](docs/canonical-vertical-slice.md)
6. [Refusal Semantics](docs/refusal-semantics.md)
7. [Constitutional Meta-Layer](docs/constitutional-meta-layer.md)
8. [Formal State Machine](formal/state-machine.md)
9. [Terminology](spec/terminology.md)

## Core invariants

### Capability is not authority

A generator may propose a transition. Proposal capability does not grant authority to commit it.

### Proof precedes consequence

A consequential transition is admitted only after the required predicates are verified.

### One constitutional grammar

Admission and refusal are encoded with the same minimal TAS_DNA structure.

### Refusal is evidence, not state progression

A refused gene is retained in the evidence chronology but does not advance authorized state.

### Recovery anchors to admitted state

A refusal does not become the recovery checkpoint. Recovery anchors to the last admitted checkpoint.

### Fixed canonical refusal inputs have stable receipt identity

When the canonical refusal payload is fixed, including its resolved evaluation timestamp, the refusal receipt hash is stable across repeated execution.

## Implementation correspondence

The current upstream implementation explicitly defines:

- `TASGene` as the **canonical minimal transition unit**,
- `WakeChain.evidence_timeline()` as admissions plus refusals,
- `WakeChain.state_sequence()` as Genesis plus admissions only.

Merged PR `TrueAlpha-spiral/TrueAlpha-spiral#364` adds direct tests for refusal preservation, admission-only state progression, recovery anchoring, runtime null-collapse refusal, and deterministic refusal receipt IDs for fixed inputs.

## Status

This is an explainer and formalization surface. Its job is to describe the architecture already expressed by the canonical datum and implementation without inventing a second ontology to reconcile it.
