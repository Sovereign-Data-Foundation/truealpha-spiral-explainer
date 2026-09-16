# TAS_DNA Transition Diagram

```mermaid
flowchart TD
    P[Proposal / Instruction] --> V[Verification / Admissibility]
    V -->|Admitted| GA[TASGene<br/>decision = ADMITTED]
    V -->|Refused| GR[TASGene<br/>decision = REFUSED]
    V -->|Cannot safely represent normal decision| F[Fail-stop]

    GA --> E[Evidence timeline]
    GR --> E

    GA --> S1[Authorized state advances]
    GR --> S0[Authorized state unchanged]
```

## One datum, two projections

```mermaid
flowchart LR
    G[TAS_DNA gene G_i] --> E[Evidence projection]
    G --> S[Authorized-state projection]

    E --> EA[ADMITTED retained]
    E --> ER[REFUSED retained]

    S --> SA[ADMITTED advances state]
    S --> SR[REFUSED leaves state unchanged]
```

The same TAS_DNA grammar is used in both branches. The decision field determines whether the datum contributes only to evidence or also advances authorized state.

Formally:

\[
G_i.d=\mathrm{ADMITTED}
\Rightarrow
\mathcal E_{n+1}=\mathcal E_n\Vert G_i
\land
S_{k+1}=F(S_k,G_i),
\]

while

\[
G_i.d=\mathrm{REFUSED}
\Rightarrow
\mathcal E_{n+1}=\mathcal E_n\Vert G_i
\land
S_{k+1}=S_k.
\]
