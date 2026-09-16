# Canonical Vertical Slice

`CanonicalVerticalSlice` is the concrete execution boundary where the current TrueAlphaSpiral reference implementation turns admissibility into either an admitted transition or a first-class refusal event.

This document tracks the behavior established by the implementation and refusal-path tests merged in `TrueAlpha-spiral/TrueAlpha-spiral#364`.

## Boundary role

The vertical slice receives an operation together with an authority snapshot, a context snapshot, a WakeChain, and optionally a sovereign runtime. It verifies the candidate before consequential execution.

The core ordering is:

\[
\text{proposal}
\rightarrow
\text{verification}
\rightarrow
\text{admissibility}
\rightarrow
\begin{cases}
\text{admission} \\
\text{refusal}
\end{cases}
\rightarrow
\text{receipt}
\rightarrow
\text{WakeChain}
\]

Runtime authorization is attempted only after the verifier has admitted the candidate. A runtime anomaly can still collapse the candidate back onto the refusal path before an admitted state transition is recorded.

## Admission path

When verification succeeds and the runtime does not fail closed, the slice emits an admission receipt, creates an admitted `TASGene`, and appends that gene to the WakeChain.

At the abstract level:

\[
\operatorname{Adm}(S_n,x)=1
\Rightarrow
(O_{n+1},\Gamma_{n+1})
\]

with an admission event appended to authenticated lineage.

## Refusal path

When verification fails, or when the runtime produces a null-collapse/failure condition, the slice creates a `RefusalArtifact`, creates a refused `TASGene`, appends it to the WakeChain, and initiates recovery from the prior admitted checkpoint.

The refusal branch therefore satisfies two different invariants at once:

\[
\Delta O = 0
\]

while

\[
\Delta \Gamma \neq 0.
\]

A refusal is consequently an evidentiary event without an admitted operational transition.

## Two projections of history

The current `WakeChain` implementation deliberately exposes two views:

1. `evidence_timeline()` — Genesis plus **admissions and refusals**.
2. `state_sequence()` — Genesis plus **admitted links only**.

That distinction is directly exercised by the refusal-path tests. After one admitted operation followed by one refused operation, the evidence timeline contains three links while the admitted state sequence contains two.

This is compatible with the explainer's irreducible-state model once the names are kept distinct:

\[
S=(O,\Gamma)
\]

is the **full constitutive state**, so a refusal changes \(S\) by extending \(\Gamma\). By contrast, `WakeChain.state_sequence()` is an implementation-level projection of **authorized operational progression** and therefore does not advance on refusal.

Equivalently, if \(\Pi_O\) projects the full state onto admitted operational progression, then refusal gives

\[
S_{n+1}\neq S_n
\]

but

\[
\Pi_O(S_{n+1})=\Pi_O(S_n).
\]

This prevents the method name `state_sequence()` from being mistaken for the entire irreducible state defined elsewhere in this repository.

## Recovery anchoring

A refused branch does not become the recovery checkpoint.

If an admitted gene \(g_a\) is followed by refused gene \(g_r\), then the refused gene may retain lineage parentage to the admitted gene,

\[
\operatorname{parent}(g_r)=g_a,
\]

while recovery is anchored to the prior admitted checkpoint:

\[
\operatorname{checkpoint}(g_r)=g_a.
\]

At Genesis, a refusal before any admitted successor anchors recovery to `GENESIS`.

This keeps the refusal in the evidence history without promoting the refused branch into authorized state progression.

## Runtime null collapse

PR #364 adds direct coverage for runtime null-collapse.

If verification admits the operation but the sovereign runtime returns no valid token indices, the slice assigns the failure code

```text
RUNTIME_NULL_COLLAPSE
```

and routes the result through the normal refusal machinery.

The consequence is:

- no admitted state advancement,
- a refusal link appended to the evidence timeline,
- a refusal receipt,
- recovery anchored to the last admitted checkpoint.

Thus runtime ambiguity does not create a third path that bypasses refusal semantics.

## Deterministic refusal provenance

A refusal receipt is content-addressed from a canonical payload containing:

- human-readable reason,
- stable failure code,
- candidate hash when available,
- governing rule version,
- parent/evaluation context,
- explicit `REFUSED` decision state,
- verifier identity,
- evaluation timestamp.

The refusal receipt identifier is

\[
R^- = \operatorname{SHA256}(\operatorname{CanonicalEncode}(p^-)).
\]

PR #364 threads the vertical slice's already-resolved evaluation timestamp into `RefusalArtifact`. For fixed refusal inputs, including the same timestamp, the resulting `refusal_receipt_id` is therefore stable across repeated executions.

This matters because a wall-clock timestamp generated inside the refusal constructor would make two otherwise identical evaluations produce different receipt identities.

The determinism claim is intentionally bounded:

> **Identical canonical refusal payloads produce identical refusal receipt IDs.**

It does not mean separate real-world attempts at different evaluation times must share an identity.

## Refusal-path theorem

The implementation now directly tests the following structure:

\[
\boxed{
\text{Refuse}(x)
\Rightarrow
\Delta O=0
\land
\Delta\Gamma\neq0
\land
\operatorname{RecoveryAnchor}=\operatorname{LastAdmittedCheckpoint}
}
\]

and for fixed canonical refusal inputs:

\[
\boxed{
 p^-_a=p^-_b
\Rightarrow
R^-_a=R^-_b
}
\]

These are executable invariants rather than prose-only requirements.

## Source milestone

This explainer section corresponds to merged pull request `TrueAlpha-spiral/TrueAlpha-spiral#364`, **Add refusal-path coverage for CanonicalVerticalSlice and stabilize refusal receipt hashes**.
