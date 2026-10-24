# Lesson 036 — Transpose the bits, then batch the rows

Introduced: 2026-10-23. Understanding unchecked. Implementation: Binius64 at 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** the prover exposes a short vector of bit-polynomial evaluations. Transposing its binary coordinates turns the desired consistency check into a linear query on packed witness elements.

Prerequisite: [the packing obstacle and notation](035-packed-bits-and-the-linearity-obstacle.md). Continue with [the succinct indicator](037-tensor-algebra-and-the-ring-switch-indicator.md).

## 1. Send partial evaluations before the fresh challenge

Set $q_y=\operatorname{eq}(z,y)$. The honest prover sends $d$ field elements

$$
v_i=\sum_y w(i,y)q_y,\qquad 0\le i<d.
$$

The verifier first checks the old claim:

$$
t=\sum_i\operatorname{eq}(r_{\mathrm{lo}},i)v_i.
\tag{1}
$$

This check alone permits many false vectors. Its purpose is to tie the incoming claim to the vector that will be checked against the packed oracle.

## 2. Transpose a binary matrix, not a field matrix

Let $c_b(a)$ be binary coordinate $b$ of $a\in K$. Regard the vector $v$ as the binary matrix $V_{i,b}=c_b(v_i)$. Transpose it and repack its columns:

$$
u_b=\sum_i\beta_i c_b(v_i).
\tag{2}
$$

Because the witness entries are bits, coordinate extraction distributes through their weighted sum:

$$
c_b(v_i)=\sum_y w(i,y)c_b(q_y)
\quad\text{over }\mathbb F_2.
$$

Substitute into (2):

$$
u_b=\sum_y\pi[y]\,c_b(q_y).
\tag{3}
$$

This is the crucial rearrangement. The verifier computes $u$ from received $v$; the prover does not supply an independent, unauthenticated transpose.

## 3. Fresh row batching gives one packed linear relation

After receiving $v$, sample $\rho\in K^\kappa$ and expand $\gamma_b=\operatorname{eq}(\rho,b)$. Define the binary-linear map

$$
L_\gamma(a)=\sum_b\gamma_b c_b(a).
$$

The next claim and public coefficient table are

$$
s'=\sum_b\gamma_bu_b,\qquad
A[y]=L_\gamma(q_y).
$$

Equation (3) proves

$$
\boxed{s'=\sum_y\pi[y]A[y].}
\tag{4}
$$

This is a linear functional of the packed oracle. It is generally not a single ordinary evaluation $\widetilde\pi(p)$. The oracle protocol must authenticate (4), including the correct transparent coefficient polynomial $\widetilde A$.

## 4. A complete two-cell example

In $\mathbb F_4$ with $\alpha^2=\alpha+1$, take packed cells $\pi=[1,\alpha]$, so the flattened bits are $[1,0,0,1]$. Choose $r_{\mathrm{lo}}=z=\alpha$.

$$
q=[\alpha+1,\alpha],\quad
v=[\alpha+1,\alpha],\quad
t=1.
$$

The binary matrix and its repacked transpose are

$$
V=\begin{pmatrix}1&1\\0&1\end{pmatrix},
\qquad u=[1,\alpha+1].
$$

Choose fresh $\rho=\alpha$, hence $\gamma=[\alpha+1,\alpha]$. Then

$$
s'=(\alpha+1)+\alpha(\alpha+1)=\alpha,
$$

while $A=[1,\alpha]$ and

$$
\sum_y\pi[y]A[y]=1+\alpha^2=\alpha.
$$

Neither step requires bit extraction to be $K$-linear.

## 5. The local soundness argument

Fix a binding packed oracle and the old query. Let $v^\star$ be its true partial-evaluation vector. If a false incoming $t$ passes (1), then $v\ne v^\star$. Binary transposition is invertible, so

$$
\Delta u=T(v-v^\star)\ne0.
$$

The polynomial

$$
D(\rho)=\sum_b\Delta u_b\operatorname{eq}(\rho,b)
$$

is a nonzero multilinear polynomial in $\kappa$ variables, of total degree at most $\kappa$. Conditional on the already fixed transcript and ideal uniform fresh $\rho$,

$$
\Pr[D(\rho)=0]\le\frac{\kappa}{|K|}.
$$

If $D(\rho)\ne0$, equation (4) is false and must be rejected by the later oracle relation check. Add that check's error separately. This is not a complete Fiat–Shamir, commitment, or protocol security theorem.

Freshness is essential: an adversary choosing $v$ after seeing $\rho$ could target the kernel of both tests. No independence among earlier reused reduction points is assumed here.

## 6. Exact source behavior and a specification distinction

[Verifier verify](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/ring_switch.rs#L48) receives the vector, checks its low-point evaluation, transposes, then samples log_packing coordinates. [Prover prove](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/ring_switch.rs#L284) follows the same order.

For GHASH it sends $128$ field elements and samples **seven** coordinates, even when the packed oracle has only one element. The vector's field payload is $128\cdot16=2048$ bytes in the plain field representation; this is not the entire proof size.

Blueprint §4.8's summary describes the row-batching point using the packed-table dimension. The pinned implementation uses the logarithm of the extension degree instead. The bound derived above follows the actual vector length and sample_many(log_packing), not that abbreviated dimension statement.

[TensorAlgebra::transpose](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/math/src/tensor_algebra.rs#L92) delegates to square_transpose over the binary base field. Low-level architecture kernels are not validated by this algebraic derivation.

## Checks and recall

[Independent validation](../../work/lessons035-037-validation.json): 66,624 honest $\mathbb F_4$ row-batch cases, all 15 nonzero two-entry error vectors and four GHASH fixtures. Every nonzero toy error has at most one root among four row challenges. No Rust execution or full adversarial transcript search.

**Takeaway:** transpose supplies the algebraic identity; the fresh challenge makes a false auxiliary vector hard to hide.

**Recall:** which message must be fixed before $\rho$ is sampled, and which later check authenticates $s'$?
