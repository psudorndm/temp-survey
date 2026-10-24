# Lesson 026 — Send the weights to the table

Introduced: 2026-10-23. Understanding unchecked. Code revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** a lookup evaluation can be rewritten by collecting its interpolation weights into buckets indexed by table address.

Prerequisites: [MLE weights](006-equality-weights-and-folding.md), [fixed-base limbs](025-fixed-base-limbs-and-transparent-power-table.md). Next: [proving the pushforward](027-logup-rational-identity-and-fraction-gkr.md).

## 1. The looked-up vector

A table $T$ has $M=2^m$ entries. An integer index vector $I$ has $N=2^n$ entries with $0\leq I[x]<M$. It defines the looked-up vector

$$
P[x]=T[I[x]].
$$

The input claim is $e=\widetilde P(r)$, where $r\in K^n$. Define the interpolation weights $w_x=\operatorname{eq}_n(r,x)$. Then

$$
e=\sum_x w_x T[I[x]].
$$

Gather rows that use the same address:

$$
Y[v]=\sum_{\{x:I[x]=v\}}w_x,\qquad 0\leq v<M.
$$

This is the **weighted pushforward** of the equality table along $I$. Reordering the finite sum gives

$$
\widetilde P(r)
=\sum_{v=0}^{M-1}T[v]Y[v]
=\langle T,Y\rangle.
\tag{1}
$$

Nothing probabilistic has happened yet. This identity holds for every table, index map and evaluation point.

## 2. A four-row numerical example

Use $\mathbb F_4$ with $\alpha^2=\alpha+1$, and little-endian row coordinates. Let

$$
T=[1,\alpha,\alpha+1,0],\qquad
I=[0,1,0,3],\qquad r=(\alpha,\alpha).
$$

The row weights are

$$
w=[\alpha,1,1,\alpha+1].
$$

Rows zero and two both address table position zero. Its bucket receives $\alpha+1$. The full pushforward is

$$
Y=[\alpha+1,1,0,\alpha+1].
$$

The looked-up vector and its evaluation are

$$
P=[1,\alpha,1,0],\qquad
\langle w,P\rangle=\alpha+\alpha+1=1.
$$

On the table side,

$$
\langle T,Y\rangle=(\alpha+1)+\alpha=1.
$$

The buckets contain field-valued interpolation weights, not integer access counts. Cancellation in these buckets is part of the exact identity.

## 3. Why this is a transpose

Define an incidence matrix $A$ by $A_{x,v}=1$ if $I[x]=v$, and zero otherwise. Each row selects exactly one table entry. Then

$$
P=AT,\qquad Y=A^{\mathsf T}w,\qquad
w^{\mathsf T}AT=(A^{\mathsf T}w)^{\mathsf T}T.
$$

Pulling values from table addresses and pushing weights back to those addresses are transpose operations under the field inner product.

This interpretation explains the name; the proof must still establish that the prover used the intended incidence matrix.

## 4. Twelve lookers share one pushforward

The fixed-base IntMul path has 12 gathered columns, called lookers in the lookup API. Column $j$ has its own index map $I_j$, old point $r_j$ and claim $e_j$. These points include the inverse-Frobenius transforms from Lesson 025.

The wrapper samples one fresh scalar $\gamma$ and forms

$$
\begin{aligned}
w_{j,x}&=\gamma^j\operatorname{eq}_n(r_j,x),\\
Y[v]&=\sum_{j=0}^{11}\sum_{\{x:I_j[x]=v\}}w_{j,x},\\
E&=\sum_{j=0}^{11}\gamma^j e_j.
\end{aligned}
$$

Equation (1) becomes $\langle T,Y\rangle=E$.

These are powers of one scalar, not Lesson 024's six-coordinate tensor weights. A fixed nonzero 12-entry claim-error vector defines a nonzero polynomial in $\gamma$ of degree at most 11, giving the local bound $11/|K|$ before the later proof obligations.

## 5. What the implementation actually builds

The inspected combined_lookers body:

1. Materializes the needed powers of $\gamma$.
2. Allocates each looker's numerator buffer at its own row count.
3. Expands its scaled equality tensor, folding $\gamma^j$ into the expansion.
4. Groups the lookers by table.
5. Scatter-adds each numerator entry into its integer-indexed table bucket.
6. Merges worker-local bucket accumulators and repacks the result.

The inner operation is exactly $Y[I_j[x]]\mathrel{+}=w_{j,x}$. The bounds-checked scatter also enforces the honest prover's index-range precondition; a prover-side assertion alone is not a verifier proof against malicious indices.

The current scatter parallelizes over groups of lookers, not individual rows, and allocates at most one scalar bucket accumulator per busy worker. Source comments express the intended performance regime; no speedup or peak-memory figure was measured here.

For IntMul the pushforward has 65,536 entries, regardless of how many constraint rows read the table. The prover still processes the looker rows. The saving is avoiding an additional commitment to each large looked-up vector, not eliminating linear witness processing.

## 6. What must be proved next

A dishonest prover can send arbitrary buckets. The remaining obligations are:

$$
Y=\sum_j\gamma^j(I_j)_*\operatorname{eq}_{r_j},
\qquad
\langle T,Y\rangle=E.
$$

The first binds the buckets to the index maps; the second binds the claimed looked-up values to those buckets. Later index claims must also be tied to the original exponent bits.

The actual transparent-table wrapper samples $\gamma$, receives a commitment to $Y$, then runs the reduction using a fresh logarithmic-derivative challenge. It ultimately queues two linear relations against the same $Y$ commitment. Commitment verification may be deferred by the channel; queuing a relation is not the same as independently finishing its proof.

## Sources and checks

- [IP lookup specification](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip/src/logup_star/mod.rs): pullback/pushforward identity and indexed-lookup interface.
- [Witness construction](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip-prover/src/logup_star/witness.rs): combined_lookers, combined_pushforward and scatter_add bodies.
- [Committed transparent wrapper](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/logup_star.rs): verify_transparent.
- [Prover mirror](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop-prover/src/logup_star.rs): prove_transparent.
- [Soukhanov, Logup*, ePrint 2025/946](https://eprint.iacr.org/2025/946): primary paper metadata and abstract reviewed. Its PDF returned HTTP403 in this session; the derivation here is independently given and checked against the pinned source, not attributed to an unread theorem.

Checked all 256 four-row maps into a four-entry table at all 16 row points over $\mathbb F_4$: 4,096 pushforward identities. Additional two-looker fixtures over $\mathbb F_{16}$ check the shared power weighting and table inner product. These are independent scalar models, not Rust or channel execution.

**Remember:** collect evaluation weights by address; the lookup value becomes an inner product over the table.

**Recall:** what could a dishonest prover change if we checked the table inner product but never checked the pushforward?
