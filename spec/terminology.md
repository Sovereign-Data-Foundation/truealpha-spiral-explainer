# Canonical Terminology

This glossary keeps prose, equations, diagrams, and implementation terms aligned around the upstream TAS_DNA model.

| Term | Symbol / Form | Meaning |
|---|---|---|
| TAS_DNA gene | \(G_i=(o_i,c_i,a_i,x_i,p_i,\Phi_i,d_i,r_i)\) | Canonical minimal transition datum |
| TASGene | `TASGene` | Current implementation of the TAS_DNA gene |
| Origin | \(o_i\) | Initiating intent/source of the transition |
| Context | \(c_i\) | Operational frame in which the candidate is evaluated |
| Authority | \(a_i\) | Scoped authorization bound to the transition |
| Operation | \(x_i\) | Candidate action or transformation |
| Parent | \(p_i\) | Parent gene identity / lineage reference |
| Invariants | \(\Phi_i\) | Rule set evaluated for the candidate |
| Decision | \(d_i\) | `ADMITTED`, `REFUSED`, or `PENDING` |
| Receipt | \(r_i\) | Evidence emitted after decision |
| Gene chronology | \(\mathcal G\) | Ordered TAS_DNA datum sequence |
| Evidence projection | \(\mathcal E(\mathcal G)\) | Projection retaining admissions and refusals |
| Authorized state | \(S_k\) | Operational state progression advanced only by admitted transitions |
| Evidence timeline | `WakeChain.evidence_timeline()` | Current implementation view containing Genesis, admissions, and refusals |
| State sequence | `WakeChain.state_sequence()` | Current implementation view containing Genesis and admitted links only |
| Refusal receipt ID | \(R_i^-=H(\operatorname{Encode}(p_i^-))\) | Content-derived identity of the canonical refusal payload |
| Recovery anchor | \(\operatorname{Checkpoint}(S_k)\) | Last admitted checkpoint used for recovery |
| Canonical Vertical Slice | `CanonicalVerticalSlice` | Execution boundary that verifies, admits/refuses, emits receipts, appends WakeChain evidence, and initiates recovery |
| Runtime null collapse | `RUNTIME_NULL_COLLAPSE` | Runtime condition converted into a refused TAS_DNA gene |
| Complete mediation | \(\neg\mathbb A(e)\Rightarrow\neg\mathbb C(e)\) | Every consequential path must cross the admissibility boundary |
| Constitutional object | \(P_v\) | Canonical versioned rule object governing admissibility and amendment |
| Constitutional identity | \(K_v=H(\operatorname{Encode}(P_v))\) | Content-addressed identity of constitutional version |
| Constitutional delta | \(\Delta_c\) | Structured amendment proposal bound to expected parent identity |
| Cursive computation | — | Extension-only processing of lineage-bearing TAS_DNA genes |
| Rebase | \(G'_b=\operatorname{Rebase}(G_b,H_1)\) | Re-derivation of a stale candidate against a new parent without rewriting prior evidence |

## Required distinctions

### TAS_DNA datum vs projections

The datum is the gene:

\[
G_i=(o_i,c_i,a_i,x_i,p_i,\Phi_i,d_i,r_i).
\]

Evidence and authorized state are projections over the resulting chronology. They are not separate competing ontologies.

### Evidence timeline vs state sequence

For refusal:

\[
G_i.d=\mathrm{REFUSED}
\Rightarrow
\mathcal E_{n+1}=\mathcal E_n\Vert G_i
\]

while

\[
S_{k+1}=S_k.
\]

The refusal is recorded but does not become the next authorized state.

### Admission vs refusal

Both branches use the same TAS_DNA grammar.

Admission:

\[
G_i.d=\mathrm{ADMITTED}
\Rightarrow
S_{k+1}=F(S_k,G_i).
\]

Refusal:

\[
G_i.d=\mathrm{REFUSED}
\Rightarrow
S_{k+1}=S_k.
\]

### Refusal vs fail-stop

A **refusal** is a represented negative decision encoded in a TAS_DNA gene and receipt.

A hard **fail-stop** is reserved for cases where the boundary cannot safely produce the normal represented transition object at all.

### Refusal vs recovery checkpoint

A refused gene is evidence, not authorized recovery state.

\[
\operatorname{RecoveryAnchor}(G_r)=\operatorname{LastAdmittedCheckpoint}.
\]

### Determinism vs distinct attempts

For canonical refusal payload \(p^-\):

\[
p^-_a=p^-_b
\Rightarrow
R^-_a=R^-_b.
\]

The timestamp is part of that payload, so attempts at different resolved times need not have the same receipt identity.

### Log vs lineage

A generic log is not TAS_DNA lineage. TAS_DNA lineage carries parentage, decision structure, receipt evidence, and the constitutional fields required by `TASGene`.

## Deprecated explainer abstraction

Earlier revisions of this explainer introduced

\[
S=(O,\Gamma)
\]

as a composite full-state definition. That was an explainer-level abstraction, not the state semantics expressed by the current upstream specification and implementation.

The explainer no longer uses it as the canonical TAS model.

## Naming rule

When introducing a new term, identify whether it is:

1. a field of the TAS_DNA gene,
2. a verification/admissibility predicate,
3. an evidence projection,
4. an authorized-state projection,
5. a constitutional/meta-transition object, or
6. recovery/fail-stop behavior.

Do not create a new state object when the distinction can be represented as a projection over the canonical TAS_DNA chronology.
