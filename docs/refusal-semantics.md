# Refusal Semantics

A refusal is a TAS_DNA gene whose decision is `REFUSED`.

The architecture does not need a second kind of object for failure. Admission and refusal use the same canonical transition grammar:

\[
G_i=(o_i,c_i,a_i,x_i,p_i,\Phi_i,d_i,r_i).
\]

The refusal branch is selected by

\[
d_i=\mathrm{REFUSED}.
\]

## Refusal preserves evidence and state semantics simultaneously

Let \(\mathcal E_n\) denote the evidence projection and \(S_k\) the authorized state sequence.

A refusal gives

\[
\mathcal E_{n+1}=\mathcal E_n\Vert G_i
\]

while

\[
S_{k+1}=S_k.
\]

So the event is preserved without being promoted into authorized state.

This is the precise meaning of:

> **No state change does not mean no event.**

## Evidence timeline vs state sequence

The current reference implementation exposes exactly these projections:

- `WakeChain.evidence_timeline()` contains Genesis, admissions, and refusals.
- `WakeChain.state_sequence()` contains Genesis and admissions only.

After one admission followed by one refusal:

```text
evidence timeline: Genesis -> Admission -> Refusal
state sequence:    Genesis -> Admission
```

There is no need to reinterpret the refusal as a change to a larger composite state. The refusal already has a canonical home: the evidence chronology carried by TAS_DNA.

## Refusal-path invariant

For refused gene \(G_i\):

\[
\boxed{
d_i=\mathrm{REFUSED}
\Rightarrow
\mathcal E_{n+1}=\mathcal E_n\Vert G_i
\land
S_{k+1}=S_k}
\]

This is the central refusal invariant.

## Recovery anchor

A refused gene remains in evidence but does not become an authorized recovery checkpoint.

If \(G_a\) is the last admitted gene and \(G_r\) is the following refused gene, then recovery remains anchored to \(G_a\):

\[
\operatorname{RecoveryAnchor}(G_r)=G_a.
\]

If refusal occurs before any admitted successor to Genesis, recovery anchors to `GENESIS`.

## Runtime null collapse

The current `CanonicalVerticalSlice` may first verify a proposal successfully and then encounter a runtime null-collapse.

The implementation records

```text
RUNTIME_NULL_COLLAPSE
```

and emits a refused TAS_DNA gene rather than advancing authorized state.

Thus:

\[
\operatorname{Verified}(x)
\land
\operatorname{RuntimeNullCollapse}(x)
\Rightarrow
G_i.d=\mathrm{REFUSED}.
\]

The same gene grammar survives the runtime failure.

## Deterministic refusal identity

A refusal receipt is content-addressed from its canonical payload.

Let \(p_i^-\) be the canonical refusal payload and

\[
R_i^-=H(\operatorname{Encode}(p_i^-)).
\]

Then

\[
p_a^-=p_b^-
\Rightarrow
R_a^-=R_b^-.
\]

Merged PR `TrueAlpha-spiral/TrueAlpha-spiral#364` makes the execution boundary supply the resolved evaluation timestamp to the refusal artifact, preventing an internal wall clock from changing receipt identity for otherwise fixed inputs.

## Refusal is non-compensatory

A refusal is not equivalent to committing a mutation and then undoing it.

TAS prefers

\[
\text{verify}\rightarrow\text{admit}\rightarrow\text{commit}
\]

over

\[
\text{commit}\rightarrow\text{detect}\rightarrow\text{compensate}.
\]

The refused gene proves that the candidate reached the boundary and did not become authorized state.
