# Foundational Axioms

TrueAlphaSpiral begins from a separation that ordinary generative systems often blur:

\[
\text{Capability} \not\Rightarrow \text{Authority}
\]

A system may be able to propose, synthesize, predict, optimize, or construct a candidate transition without thereby possessing authority to make that transition consequential.

## Axiom 1 — Capability does not imply authority

Let \(C(x)\) denote computational capability to propose or construct action \(x\), and let \(A(x)\) denote externally grounded authority for \(x\).

TAS rejects the implication:

\[
C(x) \Rightarrow A(x)
\]

Authority must be established independently of generative capability.

## Axiom 2 — Proof precedes consequence

Consequential mutation is admissible only after the required predicates have been recomputed and verified.

For an effectful transition \(x\):

\[
\operatorname{Commit}(x)
\Rightarrow
\operatorname{Proof}(x)
\]

The verifier is not permitted to infer proof from confidence, fluency, model identity, process identity, or successful proposal generation.

## Axiom 3 — Fail closed

When admissibility cannot be established, execution does not guess.

\[
\neg\operatorname{Prove}(\operatorname{Adm}(S,x))
\Rightarrow
\neg\operatorname{Commit}(x)
\]

Depending on the failure class, the result is either an authenticated refusal or a fail-stop state \(\bot\).

## Axiom 4 — Trajectory integrity is constitutive

The current system state is not identified solely by its visible operational configuration.

\[
S=(O,\Gamma)
\]

Authenticated lineage \(\Gamma\) is part of state identity. Therefore two operationally equal configurations may still be distinct system states:

\[
O_a=O_b
\quad\land\quad
\Gamma_a\neq\Gamma_b
\quad\Rightarrow\quad
S_a\neq S_b
\]

## Axiom 5 — Complete mediation

Every consequential effect must cross the admissibility boundary.

Let \(E\) be the set of possible effect paths. TAS requires:

\[
\forall e\in E,
\neg \mathbb{A}(e)
\Rightarrow
\neg \mathbb{C}(e)
\]

where \(\mathbb{A}\) means admissible and \(\mathbb{C}\) means consequentially committed.

The architecture fails if an alternate execution path can bypass the gate.

## Consequence

The foundational design objective is therefore not to make a generator intrinsically authoritative. It is to make unauthorized consequence unreachable through deterministic mediation.
