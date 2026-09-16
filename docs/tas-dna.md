# TAS_DNA — One Datum, Two Projections

The explainer uses one canonical transition datum: the TAS_DNA gene.

The upstream specification and implementation define the minimal transition unit as

\[
G_i =
(
\text{origin},
\text{context},
\text{authority},
\text{operation},
\text{parent},
\text{invariants},
\text{decision},
\text{receipt}
).
\]

The current implementation names this object `TASGene`.

The point of the formalization is not to invent a second state ontology around the gene. It is to make explicit that **one canonical datum can be viewed through two different projections**.

## 1. The datum

Let

\[
G_i=(o_i,c_i,a_i,x_i,p_i,\Phi_i,d_i,r_i)
\]

where:

- \(o_i\) — origin,
- \(c_i\) — context,
- \(a_i\) — authority,
- \(x_i\) — proposed operation,
- \(p_i\) — parent gene identity,
- \(\Phi_i\) — invariants evaluated,
- \(d_i\in\{\mathrm{ADMITTED},\mathrm{REFUSED},\mathrm{PENDING}\}\) — decision,
- \(r_i\) — receipt.

This is the smallest object that carries the constitutional relationship through the system.

## 2. One chronology

Let \(\mathcal G_n\) be the ordered chronology of processed TAS_DNA genes.

For every successfully represented decision event,

\[
\mathcal G_{n+1}=\mathcal G_n\Vert G_i.
\]

Admission and refusal are therefore not different species of data. They are different decision values inside the same canonical datum.

## 3. Evidence projection

Define an evidence projection

\[
\mathcal E(\mathcal G_n)
\]

that retains the full ordered chronology of represented genes.

Operationally, this corresponds to the WakeChain evidence timeline: admitted and refused genes are both retained.

For either decision,

\[
\mathcal E_{n+1}=\mathcal E_n\Vert G_i.
\]

## 4. Authorized-state projection

Define an authorized-state projection

\[
\mathcal S(\mathcal G_n)
\]

that advances only through admitted genes.

For an admitted gene,

\[
d_i=\mathrm{ADMITTED}
\Rightarrow
S_{k+1}=F(S_k,G_i).
\]

For a refused gene,

\[
d_i=\mathrm{REFUSED}
\Rightarrow
S_{k+1}=S_k.
\]

The refusal is still present in \(\mathcal G\) and therefore in the evidence projection. It simply does not become the next authorized state.

## 5. The key distinction

The architecture therefore does **not** require redefining state as a composite object such as

\[
S=(O,\Gamma).
\]

That construction may be useful in another model, but it is not needed to explain the current TAS_DNA implementation and it conflicts with the upstream specification's explicit admission-only state lineage.

The cleaner statement is:

\[
\boxed{
\text{one TAS\_DNA datum}
\longrightarrow
\begin{cases}
\text{evidence projection: all represented decisions}\\
\text{state projection: admitted decisions only}
\end{cases}
}
\]

There is no contradiction once the two projections are not collapsed into one variable.

## 6. Refusal

A refusal is therefore formalized as:

\[
\boxed{
G_i.d=\mathrm{REFUSED}
\Rightarrow
\mathcal E_{n+1}=\mathcal E_n\Vert G_i
\land
S_{k+1}=S_k
}
\]

This matches the current implementation:

- the refused `TASGene` is appended to WakeChain evidence,
- `state_sequence()` does not advance,
- recovery remains anchored to the last admitted checkpoint.

The negative event is preserved without being promoted into authorized state.

## 7. Admission

Admission uses the same datum:

\[
\boxed{
G_i.d=\mathrm{ADMITTED}
\Rightarrow
\mathcal E_{n+1}=\mathcal E_n\Vert G_i
\land
S_{k+1}=F(S_k,G_i)
}
\]

The difference between success and refusal is therefore carried by `decision`, not by changing the underlying grammar of the computational unit.

## 8. Self-similarity

This is the TAS_DNA claim in its narrowest form:

> **The same constitutional data grammar survives both admission and refusal.**

The gene carries origin, context, authority, operation, parentage, invariants, decision, and receipt in either branch. The local datum therefore preserves the same relationship that the larger execution architecture enforces.

## Source correspondence

This formalization follows the upstream definitions in:

- `core/gene.py` — `TASGene: the canonical minimal transition unit`,
- `docs/specs/intelligent_self_similar_design.md` — §3 two simultaneous histories and §4 TAS_DNA gene,
- `core/wakechain.py` — evidence timeline versus admission-only state sequence,
- merged PR `TrueAlpha-spiral/TrueAlpha-spiral#364` — executable refusal-path coverage.
