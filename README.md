# TrueAlphaSpiral Explainer

**TrueAlphaSpiral (TAS)** is a deterministic execution architecture for consequential computation. Its central rule is simple:

> **Capability does not imply authority. Proof must precede consequence.**

This repository is the public explainer and formal-specification companion for the TrueAlphaSpiral architecture. It is organized so a reader can move from the core state model to admissibility, refusal semantics, constitutional transitions, and implementation-oriented reference material without changing vocabularies between layers.

## Core state model

TAS treats state as an irreducible pair:

\[
S_n := (O_n, \Gamma_n)
\]

where:

- \(O_n\) is the protected **operational state**.
- \(\Gamma_n\) is the ordered, authenticated **lineage** of evaluated events and receipts.

Operational equality is therefore not sufficient for state identity:

\[
O_a = O_b \;\not\Rightarrow\; S_a = S_b
\]

If the lineages differ, the states differ:

\[
\Gamma_a \neq \Gamma_b \;\Rightarrow\; (O_a,\Gamma_a) \neq (O_b,\Gamma_b)
\]

This gives TAS its central history-sensitive property: **history is not external audit metadata; authenticated history is constitutive of state.**

## Refusal is a state transition

A rejected proposal does not mutate protected operational state, but it does append authenticated evidence to lineage:

\[
O_{n+1}=O_n
\]

\[
\Gamma_{n+1}=\operatorname{Extend}(\Gamma_n,r^-_n)
\]

Therefore:

\[
S_{n+1}\neq S_n
\]

The key distinction is:

> **Refusal preserves operational state while advancing constitutive state.**

This repository treats admission, refusal, and fail-stop as distinct transition outcomes rather than collapsing all non-mutation into a single notion of "nothing happened."

## Architecture at a glance

```text
Proposal / Instruction
        |
        v
+-----------------------+
|  Admissibility Gate   |
| auth / authority /    |
| scope / invariants /  |
| freshness / witness   |
+-----------+-----------+
            |
      +-----+-----+
      |           |
   ADMIT        REFUSE
      |           |
      v           v
  ΔO != 0      ΔO = 0
  ΔΓ != 0      ΔΓ != 0
      |           |
      +-----+-----+
            |
            v
      S' = (O', Γ')

Evaluation indeterminate or structurally unprovable:
            -> FAIL-STOP / ⊥
```

## Reading order

1. [Foundational Axioms](docs/foundational-axioms.md)
2. [The Irreducible State](docs/irreducible-state.md)
3. [Cursive Computation](docs/cursive-computation.md)
4. [Admissibility](docs/admissibility.md)
5. [Canonical Vertical Slice](docs/canonical-vertical-slice.md)
6. [Refusal Semantics](docs/refusal-semantics.md)
7. [Constitutional Meta-Layer](docs/constitutional-meta-layer.md)
8. [Formal State Machine](formal/state-machine.md)
9. [Terminology](spec/terminology.md)

## Core invariants

### 1. Capability is not authority

A generator may propose a transition. It does not thereby acquire authority to commit that transition.

### 2. Consequence is proof-gated

A consequential transition is admissible only when the required predicates can be recomputed and verified by the execution boundary.

### 3. Refusal is evidentiary

A refused transition preserves \(O\) but advances \(\Gamma\) with a refusal receipt.

### 4. Lineage is ordered

Lineage is not an unordered set of receipts. Each new receipt binds to the authenticated tip of the prior lineage.

### 5. Fail closed under ambiguity

If the system cannot prove that a candidate transition is admissible, it does not guess. It halts or refuses according to the applicable transition semantics.

### 6. Recovery anchors to admitted state

A refusal remains in the authenticated evidence timeline without becoming the checkpoint from which protected operational state is recovered. Recovery anchors to the last admitted checkpoint.

### 7. Fixed refusal inputs have stable receipt identity

When the canonical refusal payload is fixed—including the resolved evaluation timestamp—the refusal receipt hash is stable across repeated execution.

## Formal transition sketch

Let \(\operatorname{Adm}(S_n,x)\in\{0,1\}\) be the admissibility predicate for proposal \(x\).

\[
T(S_n,x)=
\begin{cases}
(O_{n+1},\operatorname{Extend}(\Gamma_n,r^+_n)) & \text{if } \operatorname{Adm}(S_n,x)=1 \\
(O_n,\operatorname{Extend}(\Gamma_n,r^-_n)) & \text{if } \operatorname{Adm}(S_n,x)=0 \\
\bot & \text{if evaluation itself cannot be established}
\end{cases}
\]

The operational mutation condition is deliberately narrower than state advancement:

\[
\Delta O \neq 0 \iff \text{admitted consequential mutation}
\]

while, for an evaluated event that produces a receipt:

\[
\Delta \Gamma \neq 0
\]

and therefore normally:

\[
\Delta S \neq 0
\]

whether the proposal is admitted or refused.

## Implementation correspondence

The current reference implementation makes an important projection explicit through `WakeChain`:

- `evidence_timeline()` contains Genesis, admissions, **and refusals**.
- `state_sequence()` contains Genesis and **admissions only**.

In this explainer, \(S=(O,\Gamma)\) denotes the irreducible full state. The implementation method `state_sequence()` should therefore be read as the admitted operational-state progression, not as the complete \((O,\Gamma)\) object. A refusal advances the evidence timeline and the full constitutive state while leaving the admitted operational projection unchanged.

Merged PR `TrueAlpha-spiral/TrueAlpha-spiral#364` adds executable coverage for that distinction, recovery anchoring to the prior admitted checkpoint, runtime null-collapse routing through refusal, and stable refusal receipt IDs for fixed evaluation inputs.

## Status

This repository is an explainer/specification surface. It is intended to make the architecture independently inspectable and to separate claims about the model from executable verification work.

The specification is under active refinement. Definitions should be tightened by preserving one vocabulary across prose, equations, diagrams, and reference implementations.
