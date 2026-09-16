# Refusal Semantics

TAS distinguishes **non-mutation** from **no state transition**.

A refused proposal can leave protected operational state untouched while still changing the full system state by appending authenticated evidence.

## Refusal transition

For refused proposal \(x\):

\[
O_{n+1}=O_n
\]

and

\[
\Gamma_{n+1}=\operatorname{Extend}(\Gamma_n,r^-_x).
\]

Therefore:

\[
S_{n+1}\neq S_n.
\]

The refusal receipt should bind at minimum to the state and evidence necessary to establish what was evaluated and why no operational mutation occurred.

## Three outcomes

TAS should keep these outcomes distinct:

| Outcome | Operational state | Lineage | Meaning |
|---|---:|---:|---|
| Admission | changes as authorized | advances | Consequence committed |
| Refusal | unchanged | advances | Candidate evaluated and rejected |
| Fail-stop | no admissible successor | implementation-defined emergency record | Evaluation itself could not be safely established |

The exact fail-stop recording mechanism may vary by implementation. What must not happen is an unproven operational mutation.

## Why refusal matters

A refusal receipt is not merely a log message. Once bound into \(\Gamma\), it becomes part of the authenticated trajectory against which future transitions may be evaluated.

This gives a stronger statement than "failure increases knowledge":

> **Refusal increases authenticated information about trajectory.**

That information may include the attempted operation, parent state, applicable policy, evidence set, failure predicate, and receipt identity.

## Refusal is non-compensatory

A refusal is not equivalent to committing a mutation and then attempting to undo it.

Rollback after unauthorized consequence is categorically weaker than refusal-before-execution because the forbidden effect may already have escaped the protected boundary.

TAS therefore prefers:

\[
\text{verify} \rightarrow \text{admit} \rightarrow \text{commit}
\]

over

\[
\text{commit} \rightarrow \text{detect} \rightarrow \text{compensate}.
\]

## Preservation law

For a refused event:

\[
\Delta O=0
\quad\land\quad
\Delta\Gamma\neq0.
\]

Hence:

\[
\Delta S\neq0.
\]

This is the core refusal invariant of the irreducible-state model.
