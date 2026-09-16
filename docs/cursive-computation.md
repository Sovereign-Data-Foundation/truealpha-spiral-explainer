# Cursive Computation

Cursive computation is the TAS model in which authenticated execution history is carried forward as part of the current state rather than being treated as detachable audit exhaust.

## State

\[
S_n=(O_n,\Gamma_n)
\]

The lineage coordinate \(\Gamma_n\) is an ordered authenticated trajectory. Each accepted evaluation extends that trajectory with a receipt bound to the prior tip.

## The cursive stroke

For an evaluated instruction \(x_n\), define a receipt \(r_n\). The lineage transition is

\[
\Gamma_{n+1}=\operatorname{Extend}(\Gamma_n,r_n)
\]

subject to

\[
\operatorname{parent}(r_n)=\operatorname{Tip}(\Gamma_n).
\]

This parent binding is what turns a collection of records into a trajectory.

## Admission stroke

When the proposal is admissible, both operational state and lineage advance:

\[
O_{n+1}\neq O_n,
\qquad
\Gamma_{n+1}\neq\Gamma_n.
\]

## Refusal stroke

When the proposal is not admissible, protected operational state is preserved while lineage advances:

\[
O_{n+1}=O_n,
\qquad
\Gamma_{n+1}\neq\Gamma_n.
\]

The refusal is therefore not absence of computation. It is an authenticated non-effect with evidentiary consequence.

## History-dependent identity

Two states can expose the same operational data yet remain computationally distinct:

\[
O_a=O_b,
\qquad
\Gamma_a\neq\Gamma_b
\Rightarrow
S_a\neq S_b.
\]

This prevents a system from collapsing distinct trajectories into a single identity merely because their visible outputs coincide.

## Rebase is not rewrite

A stale candidate derived from an earlier lineage head cannot silently replace its parent claim with the current head. It must be re-derived against the new authenticated parent while preserving evidence of the prior attempt.

If candidate \(e_B\) was derived from head \(H_0\), but the authenticated head has advanced to \(H_1\), then a valid successor is a new object:

\[
e'_B=\operatorname{Rebase}(e_B,H_1)
\]

with an explicit relation back to the stale candidate rather than historical erasure.

## Cursive identity

The essential rule is:

> The present is not merely a configuration. It is a configuration together with the authenticated path that made it present.
