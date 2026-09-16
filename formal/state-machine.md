# Formal State Machine

This document formalizes the current TAS implementation around one canonical datum: the TAS_DNA gene.

## 1. Canonical transition datum

Let

\[
G_i=(o_i,c_i,a_i,x_i,p_i,\Phi_i,d_i,r_i)
\]

with decision

\[
d_i\in\{\mathrm{ADMITTED},\mathrm{REFUSED},\mathrm{PENDING}\}.
\]

`TASGene` is the current implementation of this minimal transition unit.

No additional composite state definition is required to represent refusal lineage.

## 2. Ordered gene chronology

Let \(\mathcal G_n\) be the ordered chronology of represented TAS_DNA genes.

For each represented decision event,

\[
\mathcal G_{n+1}=\mathcal G_n\Vert G_i.
\]

Parentage requires

\[
p_i=\operatorname{Tip}(\mathcal G_n)
\]

under the applicable lineage semantics.

## 3. Evidence projection

Define

\[
\mathcal E:\mathcal G\to\text{EvidenceTimeline}
\]

as the projection retaining all represented admission and refusal events.

For admission or refusal:

\[
\mathcal E_{n+1}=\mathcal E_n\Vert G_i.
\]

The current `WakeChain.evidence_timeline()` is the implementation-level view corresponding to this projection.

## 4. Authorized-state projection

Let \(S_k\) denote the authorized operational state progression.

Define the state transition operator

\[
T_S(S_k,G_i)=
\begin{cases}
F(S_k,G_i), & d_i=\mathrm{ADMITTED},\\[4pt]
S_k, & d_i=\mathrm{REFUSED},\\[4pt]
S_k, & d_i=\mathrm{PENDING}.
\end{cases}
\]

Thus:

### Admission

\[
d_i=\mathrm{ADMITTED}
\Rightarrow
S_{k+1}=F(S_k,G_i).
\]

### Refusal

\[
d_i=\mathrm{REFUSED}
\Rightarrow
S_{k+1}=S_k.
\]

The current `WakeChain.state_sequence()` is the implementation-level admitted projection: Genesis plus admitted links only.

## 5. One datum, two projections

The complete relationship is therefore

\[
\boxed{
G_i
\longrightarrow
\begin{cases}
\mathcal E_{n+1}=\mathcal E_n\Vert G_i,\\
S_{k+1}=T_S(S_k,G_i).
\end{cases}}
\]

The same datum feeds both projections.

There is no contradiction between a refusal being present in evidence and absent from the next authorized state transition.

## 6. Admissibility

Let

\[
\operatorname{Adm}(S_k,x_i)\in\{0,1\}
\]

be the deterministic admission predicate.

A representative factorization is

\[
\operatorname{Adm}(S_k,x_i)
=
\operatorname{Auth}(x_i)
\wedge
\operatorname{Authority}(x_i)
\wedge
\operatorname{Scope}(x_i)
\wedge
\operatorname{Invariant}(S_k,x_i)
\wedge
\operatorname{Fresh}(S_k,x_i).
\]

If the predicate fails, the resulting gene records `REFUSED` rather than advancing authorized state.

## 7. Runtime fail-closed conversion

The current `CanonicalVerticalSlice` can admit at the verifier layer and still refuse at runtime.

Abstractly:

\[
\operatorname{Adm}(S_k,x_i)=1
\land
\neg\operatorname{RuntimeOK}(x_i)
\Rightarrow
d_i=\mathrm{REFUSED}.
\]

For runtime null-collapse:

\[
\operatorname{RuntimeNullCollapse}(x_i)
\Rightarrow
G_i.d=\mathrm{REFUSED}.
\]

The evidence projection advances; authorized state does not.

## 8. Recovery

Let

\[
\operatorname{Checkpoint}(S_k)
\]

return the most recent admitted checkpoint, with Genesis as the initial checkpoint.

For refusal:

\[
\operatorname{RecoveryAnchor}(G_i)
=
\operatorname{Checkpoint}(S_k).
\]

The refused gene is preserved in evidence but is not promoted into the recovery state.

## 9. Deterministic refusal receipt

Let \(p_i^-\) be the canonical refusal payload and define

\[
R_i^-=H(\operatorname{Encode}(p_i^-)).
\]

Then

\[
p_a^-=p_b^-
\Rightarrow
R_a^-=R_b^-.
\]

The evaluation timestamp is part of the canonical payload. Fixed inputs include a fixed resolved timestamp.

## 10. Complete mediation

Let \(P\) be the set of all consequential execution paths. TAS requires

\[
\forall e\in P,
\neg\mathbb A(e)
\Rightarrow
\neg\mathbb C(e),
\]

where \(\mathbb A\) denotes admissibility and \(\mathbb C\) denotes consequential commit.

A verifier alone is insufficient if another path can bypass it.

## 11. Upstream semantic correspondence

The upstream specification explicitly states two simultaneous histories:

- an evidentiary timeline that records admissions and refusals,
- a state lineage that advances only through valid committed transitions.

It then defines TAS_DNA as the minimal computational gene carrying origin, context, authority, operation, parent, invariants, decision, and receipt.

The formalization here preserves those semantics directly instead of redefining `state` to include the evidence chronology.

## 12. Executable correspondence

Merged PR `TrueAlpha-spiral/TrueAlpha-spiral#364` directly tests that:

1. a refused gene extends WakeChain evidence,
2. a refusal does not advance `state_sequence()`,
3. recovery anchors to the prior admitted checkpoint,
4. runtime null-collapse becomes a refusal,
5. fixed refusal inputs produce a stable `refusal_receipt_id`.

Those tests are the executable realization of the one-datum/two-projection model above.
