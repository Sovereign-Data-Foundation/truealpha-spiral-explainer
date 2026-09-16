# Canonical Terminology

This glossary is intended to keep prose, equations, diagrams, and reference implementations synchronized.

| Term | Symbol / Form | Meaning |
|---|---|---|
| State | \(S=(O,\Gamma)\) | Irreducible pair of operational state and authenticated lineage |
| Operational state | \(O\) | Protected consequential configuration |
| Lineage | \(\Gamma\) | Ordered authenticated evidentiary trajectory |
| Proposal | \(x\in\mathcal I\) | Candidate instruction or state mutation with no intrinsic authority |
| Admissibility | \(\operatorname{Adm}(S,x)\) | Deterministic predicate deciding whether consequence may occur |
| Admission receipt | \(r_x^+\) | Authenticated evidence that an admissible proposal committed |
| Refusal receipt | \(r_x^-\) | Authenticated evidence that a proposal was evaluated and rejected |
| Extend | \(\operatorname{Extend}(\Gamma,r)\) | Ordered append operation requiring valid parent binding |
| Tip | \(\operatorname{Tip}(\Gamma)\) | Current authenticated lineage head |
| Fail-stop | \(\bot\) | No admissible successor because safe evaluation cannot be established |
| Reconstruction map | \(L:\mathsf{WF}\to\mathcal O\) | Deterministic projection from well-formed lineage to operational state |
| Complete mediation | \(\neg\mathbb A(e)\Rightarrow\neg\mathbb C(e)\) | Every consequential path must cross the admissibility boundary |
| Constitutional object | \(P_v\) | Canonical versioned rule object governing admissibility and amendment |
| Constitutional identity | \(K_v=H(\operatorname{Encode}(P_v))\) | Content-addressed identity of constitutional version |
| Constitutional delta | \(\Delta_c\) | Structured amendment proposal bound to expected parent identity |
| Cursive computation | — | Computation in which authenticated trajectory is constitutive of present state |
| Rebase | \(e'_B=\operatorname{Rebase}(e_B,H_1)\) | Re-derivation of a stale candidate against a new parent without rewriting history |
| Null Collapse | — | Fail-closed non-admission behavior; use carefully and distinguish refusal from hard fail-stop |

## Required distinctions

### State advancement vs operational mutation

Do not use \(\Delta S\neq0\) as a synonym for operational consequence.

A refusal has

\[
\Delta O=0,
\qquad
\Delta\Gamma\neq0,
\qquad
\Delta S\neq0.
\]

Use \(\Delta O\neq0\) when specifically referring to consequential operational mutation.

### Refusal vs fail-stop

A **refusal** is a successfully evaluated negative decision and normally produces an authenticated refusal receipt.

A **fail-stop** means the evaluation itself cannot be safely established. It is represented by \(\bot\) at the abstract transition level.

### Equality vs identity

Two systems may expose equal operational state while remaining different full states:

\[
O_a=O_b
\quad\land\quad
\Gamma_a\neq\Gamma_b
\quad\Rightarrow\quad
S_a\neq S_b.
\]

### Log vs lineage

A log is not automatically lineage. TAS lineage requires order, parent binding, authenticated integrity, and participation in future state identity/admissibility.

## Naming rule

When introducing a new term, define whether it belongs to:

1. proposal space,
2. verification/admissibility,
3. operational state,
4. evidentiary lineage,
5. constitutional/meta-transition, or
6. recovery/fail-stop semantics.

Terms that cross layers should explicitly state how they map between them.
