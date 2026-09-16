# Formal State Machine

This document collects the minimal transition semantics used throughout the explainer.

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

Additional witness predicates may be required by a concrete implementation.

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
& \operatorname{Adm}(S_n,x)=1,\\[4pt]
(O_n,\operatorname{Extend}(\Gamma_n,r^-_x))
& \operatorname{Adm}(S_n,x)=0,\\[4pt]
\bot
& \text{evaluation cannot be safely established.}
\end{cases}
\]

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
