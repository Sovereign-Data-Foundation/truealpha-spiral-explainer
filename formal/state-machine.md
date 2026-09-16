# Formal State Machine

This document collects the minimal transition semantics used throughout the explainer and maps them to the current refusal-path implementation.

## State space

Let

\[
S=(O,\Gamma)
\]

where \(O\in\mathcal O\) is operational state and \(\Gamma\in\mathsf{WF}\) is a well-formed authenticated lineage.

Let proposal space be \(\mathcal I\).

Define a transition function

\[
T:(S\times\mathcal I)\to S\cup\{\bot\}.
\]

## Admissibility

Let

\[
\operatorname{Adm}:S\times\mathcal I\to\{0,1\}
\]

be a deterministic predicate reconstructed from the applicable authenticated evidence.

A representative factorization is

\[
\operatorname{Adm}(S,x)
=
\operatorname{Auth}(x)
\wedge
\operatorname{Authority}(x)
\wedge
\operatorname{Scope}(x)
\wedge
\operatorname{Invariant}(S,x)
\wedge
\operatorname{Fresh}(S,x).
\]

Additional context-binding, runtime, and witness predicates may be required by a concrete implementation.

## Receipt extension

For receipt \(r\), define

\[
\operatorname{Extend}(\Gamma,r)
\]

only when

\[
\operatorname{parent}(r)=\operatorname{Tip}(\Gamma)
\]

and all receipt-validation rules hold.

## Transition relation

For proposal \(x\):

\[
T(S_n,x)=
\begin{cases}
(O_{n+1},\operatorname{Extend}(\Gamma_n,r^+_x))
& \operatorname{Adm}(S_n,x)=1\land\operatorname{RuntimeOK}(x),\\[4pt]
(O_n,\operatorname{Extend}(\Gamma_n,r^-_x))
& \operatorname{Adm}(S_n,x)=0,\\[4pt]
(O_n,\operatorname{Extend}(\Gamma_n,r^-_{\text{runtime}}))
& \operatorname{Adm}(S_n,x)=1\land\neg\operatorname{RuntimeOK}(x),\\[4pt]
\bot
& \text{evaluation cannot be safely established at all.}
\end{cases}
\]

The third branch reflects the current `CanonicalVerticalSlice`: a runtime null-collapse or runtime exception after verifier admission is converted into a refusal before admitted state progression is recorded.

## Admission invariant

For a genuinely effectful admitted proposal:

\[
\Delta O\neq0
\]

and its receipt advances lineage:

\[
\Delta\Gamma\neq0.
\]

Hence:

\[
\Delta S\neq0.
\]

## Refusal invariant

For a refused proposal:

\[
\Delta O=0
\]

while

\[
\Delta\Gamma\neq0.
\]

Hence:

\[
\Delta S\neq0.
\]

This proves:

\[
O_{n+1}=O_n
\centernot\implies
S_{n+1}=S_n.
\]

## Authorized-state projection

The current implementation exposes `WakeChain.state_sequence()` as the sequence of Genesis plus admitted links only. To avoid confusing that implementation term with the full irreducible state, define an admitted operational projection

\[
\Pi_O:S\to\mathcal A
\]

where \(\mathcal A\) denotes authorized operational progression.

For admission:

\[
\Pi_O(S_{n+1})\neq\Pi_O(S_n).
\]

For refusal:

\[
\Pi_O(S_{n+1})=\Pi_O(S_n)
\]

while the full state still changes because

\[
\Gamma_{n+1}\neq\Gamma_n.
\]

Thus the two implementation histories correspond to:

\[
\texttt{evidence\_timeline()}\sim\Gamma
\]

and

\[
\texttt{state\_sequence()}\sim\Pi_O(S).
\]

This mapping is architectural, not an assertion that the two concrete data structures are mathematically identical to the abstract objects in every implementation detail.

## Recovery function

Let

\[
\operatorname{Checkpoint}:S\to G
\]

return the latest admitted checkpoint gene, with Genesis as the initial checkpoint.

For a refusal event \(r^-\), current vertical-slice behavior requires

\[
\operatorname{RecoveryAnchor}(r^-)
=
\operatorname{Checkpoint}(S_n).
\]

The refusal itself is appended to evidence but does not become the recovery checkpoint.

Therefore:

\[
\operatorname{Refuse}(x)
\Rightarrow
\Delta O=0
\land
\Delta\Gamma\neq0
\land
\operatorname{RecoveryAnchor}=\operatorname{LastAdmittedCheckpoint}.
\]

## Deterministic refusal receipts

Let the canonical refusal payload be

\[
p^-=(q,c,h,v,p,d,g,t)
\]

where the components represent reason, failure code, candidate hash, rule version, parent context, decision state, verifier identity, and resolved evaluation timestamp.

Define the refusal receipt identifier as

\[
R^-=H(\operatorname{Encode}(p^-)).
\]

Then, assuming deterministic canonical encoding and hash function:

\[
p^-_a=p^-_b
\Rightarrow
R^-_a=R^-_b.
\]

The merged refusal-path milestone ensures the evaluated timestamp is supplied by the execution boundary, so repeated executions with the same fixed evaluation inputs do not acquire accidental receipt variation from an internal wall clock.

## Reconstruction condition

Where the implementation supports deterministic reconstruction, define

\[
L:\mathsf{WF}\to\mathcal O
\]

and require

\[
O=L(\Gamma).
\]

A candidate state that cannot be reconciled with its authenticated lineage is not well formed.

## Complete mediation

Let \(E\) be all paths capable of consequential mutation. The protected system requires

\[
\forall e\in E,
\neg\mathbb A(e)\Rightarrow\neg\mathbb C(e).
\]

This is not satisfied merely by implementing a verifier. It requires architectural exclusion of bypass paths.

## State identity

For states

\[
S_a=(O_a,\Gamma_a),\qquad S_b=(O_b,\Gamma_b),
\]

identity requires equality of the irreducible pair:

\[
S_a=S_b
\iff
O_a=O_b\land\Gamma_a=\Gamma_b.
\]

Operational equivalence alone does not establish state identity.

## Executable correspondence

Merged pull request `TrueAlpha-spiral/TrueAlpha-spiral#364` directly tests:

1. refusal appends to WakeChain evidence without advancing the admitted state sequence,
2. recovery uses the prior admitted checkpoint rather than the refused branch,
3. runtime null-collapse routes through refusal,
4. fixed refusal inputs produce stable `refusal_receipt_id` values.

These tests operationalize the refusal branch of the abstract transition relation above.
