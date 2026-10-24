# Lesson 016 — a full four-point NTT, its inverse, and the LDE

Introduced: 2026-10-23. Understanding has not been checked.

**One idea:** forward NTT evaluates a polynomial; inverse NTT recovers its coefficients. Binius64 composes these operations to extend a bit table to a larger domain.

Prerequisite: [Lesson 015](015-additive-ntt-subspace-polynomials.md). All additions below are field additions (XOR). The four-element example is explanatory; the actual BitAnd LDE uses AES-field elements.

## 1. Specify both bases before transforming

Work in $\mathbb{F}_{4}=\{0,1,\alpha ,\alpha +1\}$, with $\alpha ^{2}=\alpha +1$. The ordered domain basis is $(1,\alpha )$, so the evaluation order is

$$
d_0=0,\quad d_1=1,\quad d_2=\alpha,\quad d_3=\alpha+1.
$$

The index's low bit selects 1; its high bit selects $\alpha$. There is no bit-reversal step in this reference walkthrough.

The normalized subspace polynomials are $s_{0}(Z)=Z$ and $s_{1}(Z)=W(Z)=Z^{2}+Z$, because $W(\alpha )=1$. Thus the novel polynomial basis is

$$
N_0=1,\quad N_1=Z,\quad N_2=W,\quad N_3=ZW.
$$

Take the novel-basis coefficient vector $[1,\alpha ,\alpha +1,1]$. It represents

$$
\begin{aligned}f(Z)&=1+\alpha Z+(\alpha+1)W(Z)+ZW(Z)\\&=(1+\alpha Z)+W(Z)((\alpha+1)+Z).\end{aligned}
$$

It does **not** mean monomial coefficients $[1,\alpha ,\alpha +1,1]$. In monomials, this f is $1+Z+\alpha Z^{2}+Z^{3}$.

## 2. The butterfly

For input pair (a,b) and known twiddle t, the forward operation is

$$
\begin{aligned}u&=a+tb,\\v&=u+b=a+(t+1)b.\end{aligned}
$$

This evaluates the split $A+s_{j}B$ on sibling cosets where $s_{j}$ has constant values t and t+1. Its inverse is

$$
b=u+v,\qquad a=u+tb.
$$

The second inverse line uses the **recovered b**. This ordering is exactly what the Rust reference updates implement. The butterfly is invertible even at $t=0$; it needs no division by the domain size.

## 3. Forward transform, including exact indices

For four entries, `log_d = 2`. The reference computes `half = 2^(log_d-layer-1)`, then pairs `idx_0` with `idx_1 = idx_0` OR half. Each block's twiddle comes from GenericOnTheFly::twiddle(layer,block).

| Stage | Pairs and twiddles | Buffer after stage |
|---|---|---|
| Input | Novel coefficients | $[1, \alpha , \alpha +1, 1]$ |
| layer 0 | (0,2), (1,3), both $t=0$ | $[1, \alpha , \alpha , \alpha +1]$ |
| layer 1 | (0,1), $t=0$; (2,3), $t=\alpha$ | $[1, \alpha +1, \alpha +1, 0]$ |

For the last pair, $(a,b)=(\alpha ,\alpha +1)$. Since $\alpha (\alpha +1)=1$,

$$
u=\alpha+\alpha(\alpha+1)=\alpha+1,\qquad v=(\alpha+1)+(\alpha+1)=0.
$$

The final buffer is $[f(0),f(1),f(\alpha ),f(\alpha +1)]$. Direct substitution in $1+Z+\alpha Z^{2}+Z^{3}$ gives the same values.

Why those twiddles? In dimension two, the normalized evaluation rows are $[s_{0}(1),s_{0}(\alpha )]$=$[1,\alpha ]$ and $[s_{1}(\alpha )]=[1]$. Layer 0 selects the latter row, whose remaining tail is empty and gives $t=0$. Layer 1 selects the first row, whose tail $[\alpha ]$ gives subset sums 0 and $\alpha$ for blocks 0 and 1.

## 4. Inverse transform, by hand

Reverse the layer order, preserving each layer's twiddles:

| Inverse stage | Recovered pairs | Buffer after stage |
|---|---|---|
| Input | Evaluations | $[1, \alpha +1, \alpha +1, 0]$ |
| undo layer 1 | $t=0$ gives $(1,\alpha )$; $t=\alpha$ gives $(\alpha ,\alpha +1)$ | $[1, \alpha , \alpha , \alpha +1]$ |
| undo layer 0 | $t=0$ recovers $(1,\alpha +1)$ at (0,2), $(\alpha ,1)$ at (1,3) | $[1, \alpha , \alpha +1, 1]$ |

For the nonzero-twiddle inverse pair, $(u,v)=(\alpha +1,0)$. Then $b=\alpha +1$ and $a=(\alpha +1)+\alpha (\alpha +1)=\alpha$. We recover the original novel coefficients exactly.

## 5. The actual BitAnd low-degree extension

The inspected LowDegreeExtension::transform does this:

$$
\begin{gathered}\text{64 bit evaluations on }D\\\xrightarrow{\text{inverse NTT on }D}\text{64 novel coefficients}\\\xrightarrow{\text{append 64 zero coefficients}}\text{128 coefficients}\\\xrightarrow{\text{forward NTT on }D^{\prime}}\text{128 evaluations on }D^{\prime}.\end{gathered}
$$

Here D′ is the span of seven standard AES basis elements, and D is the prefix span of the first six. In raw AES polynomial coordinates, their points are respectively bytes 0…127 and 0…63. These are additive subspaces, not subfields of those cardinalities.

The first 64 outputs reproduce the input bits. The upper 64 are evaluations on the other coset, used to construct the LDE lookup. Because the contexts are built from compatible prefix bases, their first 64 novel polynomials agree. Padding with zeros therefore preserves the **same degree-below-64 polynomial**. Appending zero evaluations instead would specify a different interpolation problem.

Source precision: transform allocates an already-zero 128-entry buffer, fills/interpolates its first half, then transforms the whole buffer. It does not allocate a separate coefficient vector and explicitly append zeros. The arrows describe the mathematics of that implementation. The reference engine is used during lookup-table construction; this does not imply every production transform uses this slow backend.

## 6. Why the two contexts matter

Do not assume GenericOnTheFly::subspace(i) means the first i elements of its original basis. Its body returns normalized subspace-polynomial images from evals[ell−i]. In particular, reusing a larger context for a shorter buffer can change the domain.

Concrete example: $\mathbb{F}_{16}=\mathbb{F}_{2}[u]/(u^{4}+u+1)$, with raw $u=2$ and $u^{2}=4$. Build a dimension-three context on $(1,u,u^{2})$. Its dimension-two image basis is $(1,u^{2}+u)$, raw [1,6], because

$$
s_1(Z)=\frac{Z^2+Z}{u^2+u},\qquad s_1(u)=1,\quad s_1(u^2)=u^2+u.
$$

A four-entry forward transform of [0,1,0,0] using that larger context returns [0,1,6,7]. A freshly built dimension-two context on (1,u) returns [0,1,2,3]. Both represent the linear polynomial in their respective coordinate domains; those domains differ.

This example verifies the full-layer, zero-skip scalar reference behavior. Partial-layer skip_early/skip_late conventions, other context types and optimized engines still require separate study. The example does not establish that every API accepting a context behaves identically.

## 7. Evidence and limits

All code is pinned at revision 4428e759dd27956bd5b9ae5025d67eb8ccc1accf:

- [Reference forward/inverse loops](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/math/src/ntt/reference.rs): NeighborsLastReference, pair indexing, update ordering and reversed inverse layers.
- [Generic context](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/math/src/ntt/domain_context.rs): generate_evals_from_subspace, subspace and twiddle.
- [Actual BitAnd LDE](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/bitand/ntt_lookup.rs): LowDegreeExtension::new and transform.

Independent scalar checks matched all 256 $\mathbb{F}_{4}$ coefficient vectors against direct novel-polynomial evaluation and inverse recovery; verified the listed intermediate buffers; checked 256 $\mathbb{F}_{16}$ fixtures against the shortened image domain; and matched the actual-shaped AES inverse64/pad/forward128 construction against independent product-form Lagrange interpolation for all 64 unit bit words and four representative full words. These checks do not run Binius Rust, validate SIMD/memory behavior, or measure speed.

**Remember:** coefficients $\to$ forward $\to$ values; values $\to$ inverse $\to$ coefficients. LDE changes where a polynomial is evaluated, while preserving the polynomial.

Next: random batching and re-randomization, including claims initially evaluated at different row points.
