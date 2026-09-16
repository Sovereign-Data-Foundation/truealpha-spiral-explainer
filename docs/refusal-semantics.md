# Refusal Semantics

TAS distinguishes **non-mutation** from **no state transition**.

A refused proposal can leave protected operational state untouched while still changing the full constitutive state by appending authenticated evidence.

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

## Evidence timeline vs admitted state progression

The current reference implementation exposes two views of WakeChain history:

- `evidence_timeline()` contains Genesis, admissions, and refusals.
- `state_sequence()` contains Genesis and admitted links only.

This distinction is now covered directly by merged PR `TrueAlpha-spiral/TrueAlpha-spiral#364`.

For one admission followed by one refusal, the implementation requires:

```text
evidence timeline: Genesis -> Admission -> Refusal
state sequence:    Genesis -> Admission
```

The method name `state_sequence()` denotes authorized operational progression in the implementation. It is not the same object as the explainer's full irreducible state

\[
S=(O,\Gamma).
\]

Accordingly, a refusal can satisfy both:

\[
S_{n+1}\neq S_n
\]

and

\[
\Pi_O(S_{n+1})=\Pi_O(S_n),
\]

where \(\Pi_O\) is the admitted operational-state projection.

## Why refusal matters

A refusal receipt is not merely a log message. Once bound into \(\Gamma\), it becomes part of the authenticated trajectory against which future transitions may be evaluated.

This gives a stronger statement than "failure increases knowledge":

> **Refusal increases authenticated information about trajectory.**

That information may include the attempted operation, parent state, applicable policy, evidence set, failure predicate, verifier identity, evaluation time, and receipt identity.

## Deterministic refusal identity

The current `RefusalArtifact` computes `refusal_receipt_id` as a SHA-256 hash over a canonical payload containing:

- reason,
- stable failure code,
- candidate hash,
- rule version,
- parent context,
- `REFUSED` decision state,
- verifier identity,
- timestamp.

PR #364 makes the vertical slice pass its already-resolved evaluation timestamp into the refusal artifact instead of letting the refusal constructor generate a fresh wall-clock value.

For fixed canonical payload \(p^-\):

\[
R^- = H(p^-)
\]

and therefore

\[
p^-_a=p^-_b
\Rightarrow
R^-_a=R^-_b.
\]

The claim is deliberately bounded to fixed inputs. Distinct attempts evaluated at different timestamps remain distinct payloads and may have different receipt IDs.

## Runtime null collapse is a refusal

The vertical slice first verifies the proposal. If verification admits it and a sovereign runtime is present, the runtime is then consulted.

If the runtime produces no valid token indices, the implementation records:

```text
RUNTIME_NULL_COLLAPSE
```

and routes the event through the same refusal machinery rather than treating runtime failure as an untracked side path.

Thus:

\[
\operatorname{Verified}(x)
\land
\operatorname{RuntimeNullCollapse}(x)
\Rightarrow
\operatorname{REFUSED}(x).
\]

The evidence timeline advances; admitted operational progression does not.

## Recovery anchors to the prior checkpoint

A refused branch remains in evidence but is not promoted into the authorized recovery state.

If \(g_a\) is the last admitted gene and \(g_r\) is the following refused gene, then current tests require recovery to use:

\[
\operatorname{checkpoint}(g_r)=g_a.
\]

If refusal occurs before any admitted successor to Genesis, recovery anchors to `GENESIS`.

This preserves the distinction between **recording a failed attempt** and **adopting the failed attempt as state**.

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

With recovery added, the executable refusal invariant is:

\[
\boxed{
\operatorname{Refuse}(x)
\Rightarrow
\Delta O=0
\land
\Delta\Gamma\neq0
\land
\operatorname{RecoveryAnchor}=\operatorname{LastAdmittedCheckpoint}
}
\]

This is the core refusal invariant of the irreducible-state model as exercised by the current vertical-slice implementation.
