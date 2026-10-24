# Lesson 037 — Why the verifier uses a tensor algebra

Introduced: 2026-10-23. Understanding unchecked. Implementation: Binius64 at 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** two separate copies of the challenge field let the verifier interpolate the ring-switch coefficient table without incorrectly treating bit projection as field-linear.

Prerequisite: [row batching and $A[y]$](036-ring-switch-transpose-and-fresh-row-batching.md). This lesson derives the current code formula independently; it does not claim to reproduce an unread paper proof.

## 1. The tempting formula is wrong off the Boolean cube

The previous lesson defined

$$
A[y]=L_\gamma(\operatorname{eq}(z,y)).
$$

At an arbitrary query $p\in K^m$, its MLE is

$$
\widetilde A(p)=
\sum_{y\in\{0,1\}^m}
\operatorname{eq}(p,y)L_\gamma(\operatorname{eq}(z,y)).
\tag{1}
$$

It is generally **not** $L_\gamma(\operatorname{eq}_K(z,p))$. Moving the outer equality weights inside $L_\gamma$ assumes the $K$-linearity that Lesson035 disproved.

For a concrete counterexample, use $\mathbb F_4$, $z=\alpha$ and $\gamma=(1,0)$, which is a valid row query at $\rho=0$. Then $L_\gamma=c_0$ and $A=[1,0]$. At $p=\alpha$,

$$
\widetilde A(\alpha)=1+\alpha,
\qquad
L_\gamma(\operatorname{eq}(\alpha,\alpha))
=L_\gamma(1)=1.
$$

These are different field elements.

## 2. Keep the two scalar actions separate

Use the commutative algebra

$$
R=K\otimes_{\mathbb F_2}K.
$$

An element has representation

$$
C=\sum_{i=0}^{d-1}a_i\otimes\beta_i,\qquad a_i\in K.
$$

It is stored as $d$ vertical coefficients. The vertical copy contains $a\otimes1$; the horizontal copy contains $1\otimes b$. Both copies agree on the base field $\mathbb F_2$, but their general elements are distinct.

The transpose exchanges factors:

$$
T(a\otimes b)=b\otimes a.
$$

Multiplication by $a\otimes1$ scales every stored coefficient by $a$. Multiplication by $1\otimes b$ can be implemented by transpose, vertical scaling by $b$, then transpose back. This is exactly the [scale_horizontal body](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/math/src/tensor_algebra.rs#L85).

Define a linear functional over the base field by

$$
\Phi_\gamma(a\otimes b)=L_\gamma(a)b.
$$

On the stored representation, it is computed by transposing the binary coefficient matrix and taking its $K$-inner product with $\gamma$. That is [fold_vertical](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/math/src/tensor_algebra.rs#L102).

$\Phi_\gamma$ respects multiplication by horizontal field scalars. It does not generally respect vertical field scalars.

## 3. Derive the compact equality expression

Embed $z_j$ vertically and $p_j$ horizontally:

$$
E(z,p)=\prod_{j=0}^{m-1}
\left(1+z_j\otimes1+1\otimes p_j\right).
\tag{2}
$$

In characteristic two the usual equality factor simplifies inside this ring:

$$
(1-v)(1-h)+vh=1+v+h.
$$

Expanding the Boolean interpolation identity in the two copies gives

$$
E(z,p)=\sum_y
\operatorname{eq}(z,y)\otimes\operatorname{eq}(p,y).
$$

Apply $\Phi_\gamma$ and use its horizontal linearity:

$$
\boxed{\widetilde A(p)=\Phi_\gamma(E(z,p)).}
\tag{3}
$$

This proves (1) without materializing its exponentially many table entries. At Boolean $p=y$, the horizontal bits also belong to the shared base field, so (2) reduces to $\operatorname{eq}(z,y)\otimes1$, as required.

The earlier auxiliary vector also has a tensor interpretation:

$$
C=\sum_i v_i\otimes\beta_i
=\sum_y q_y\otimes\pi[y],
\qquad
\Phi_\gamma(C)=s'.
$$

Thus the transpose used for the prover message and the algebra used for the transparent coefficient evaluator implement the same factor exchange.

## 4. Trace eval_rs_eq

[Verifier eval_rs_eq](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/ring_switch.rs#L104) starts from vertical one. At each pair of coordinates it forms

$$
E_{\mathrm{next}}=E+z_jE+E p_j,
$$

where the second product is vertical and the third horizontal. Both scaled values use clones of the same old $E$. Finally fold_vertical applies $\Phi_\gamma$.

The code checks that query dimensions match and that the expanded row query has $d$ entries. TensorAlgebra::new itself pads or truncates its input; it is not a rejection-based length validator. The ring verifier separately receives exactly $d$ values.

The evaluator uses $O(md)$ field operations plus binary transposes, rather than enumerating $2^m$ coefficients. With $d=128$ fixed this is linear in the number of packed-index coordinates. It is not merely $m$ ordinary scalar multiplications, and no runtime measurement is claimed.

The doc comment repeats “vertical” for both coordinate groups. The function body and scale_horizontal establish the two distinct actions.

## 5. This algebra is not a larger field

Even though both copies are fields, $R$ can have zero divisors. In the $\mathbb F_4$ example let

$$
e=\alpha\otimes1+1\otimes\alpha.
$$

Using $\alpha^2=\alpha+1$ in both factors gives $e^2=e$. Both $e$ and $e+1$ are nonzero, but

$$
e(e+1)=0.
$$

Therefore arbitrary field inverses are not available here. The equality evaluator needs only addition, multiplication by elements of the two copies, and the final linear functional.

## Evidence, checks and recall

The source cites Diamond–Posen, [Polylogarithmic Proofs for Multilinears over Binary Towers](https://eprint.iacr.org/2024/504), §2 and Construction3.1. The abstract/revision metadata were accessible; the PDF request returned403. The paper's full construction/proof has not been read in this project. The derivation above is grounded in the inspected pinned code bodies.

[Independent validation](../../work/lessons035-037-validation.json) checks (3) against dense interpolation in 1,092 $\mathbb F_4$ cases and four GHASH fixtures, including an empty packed-index point. It also checks the off-cube counterexample and zero-divisor example. No Rust execution.

**Takeaway:** separate the scalar that builds the bit query from the scalar that interpolates the packed query.

**Recall:** why may the horizontal interpolation coefficient pass through $\Phi_\gamma$, while the vertical coefficient generally may not?
