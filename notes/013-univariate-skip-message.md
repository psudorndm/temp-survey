# Lesson 013 — Send a polynomial with prescribed zeros

2026-10-23 · BitAnd univariate skip · Introduced; understanding unchecked.

## The polynomial the message should describe

Keep the oblong operand polynomials â(Z,X), b̂(Z,X), ĉ(Z,X). Let τ be the row-weighting point from Lesson 012: up to three fixed small-field coordinates, followed by sampled large-field coordinates. Define the true weighted residual polynomial

\[
H(Z)=\sum_{x\in\{0,1\}^{\ell}}
\operatorname{eq}_{\ell}(\tau,x)
\bigl(\widehat a(Z,x)\widehat b(Z,x)+\widehat c(Z,x)\bigr).
\]

H has degree at most 126 in Z. When the AND constraints hold, every residual is zero at every bit-domain point d in D, so H(d)=0 there. H can nevertheless be nonzero outside D.

## What the prover sends

Choose a 128-point additive subspace D' inside the AES field containing the 64-point bit domain D. The remaining 64 points form the other half of D'. The prover sends 64 challenge-field values, intended as H evaluated on D' minus D, in the agreed domain order.

The verifier supplies the other 64 values itself: zeros on D. Interpolating all 128 values defines a polynomial g of degree below 128. Thus vanishing on D is enforced by the message format, not checked by independently reading the witness.

The distinction between H and g matters: H is defined by actual operand data; g is the polynomial encoded by the received message. An honest prover makes them equal. A dishonest prover may send arbitrary values, so their equality still has to be proved.

## Sample after the message, then use row sumcheck

After receiving the 64 values, the verifier samples z uniformly from K and computes g(z) by interpolation. The next reduction establishes

\[
g(z)\stackrel?=\sum_x\operatorname{eq}_{\ell}(\tau,x)
\bigl(\widehat a(z,x)\widehat b(z,x)+\widehat c(z,x)\bigr)=H(z).
\]

The bit axis is now fixed, leaving only row variables. Actual Binius64 batches this row check with operand re-randomization claims before obtaining the evaluation claims consumed by the shift reduction. Those batching details are a later lesson.

The operand fold uses the **64-point domain D**. Computing g(z) from the first message uses the **128-point domain D'**. These are different interpolation problems and use different Lagrange weights.

## A two-point toy example

Take D={0,1} and the valid one-row relation a(Z)=b(Z)=c(Z)=Z. At both domain points, the bitwise multiplication relation holds. Its residual polynomial is

\[
H(Z)=Z^2+Z,
\]

which vanishes on D but is nonzero elsewhere. On D'={0,1,α,α+1} in F₄, its evaluation list is [0,0,1,1]. The prover sends [1,1]; the verifier prepends [0,0]. That interpolates exactly H.

An all-zero message also defines a polynomial vanishing on D, but it does not equal H. In an F₁₆ challenge field containing this D', the two agree only at 0 and 1. The later identity check is what rejects the false message at other challenges; prescribed zeros alone establish no witness relationship.

## Why the random identity check is useful

If a constraint fails, the deterministic-zerocheck reasoning implies H is nonzero at some designated bit-domain point except with the remaining initial random-test error. Since g is forced to be zero at every such point, g and H must differ as formal polynomials. Once g is fixed before z, their discrepancy has few roots. A false weighted row claim is then caught by the row sumcheck, provided its terminal operands are authenticated.

This separates three obligations: initial row-error aggregation, random comparison of the two bit-axis polynomials, and subsequent proof/authentication of the row evaluation. No single stage replaces the others.

## Precise degree reconciliation: honest 126 versus encoded 127

The honest H has degree at most 126 because operand bit-axis degrees are at most 63. The inspected verifier, however, accepts an arbitrary 64-value message and interpolates it with 64 zeros on a 128-point domain. That determines a polynomial of degree at most 127. There is no explicit degree-126/parity check in the inspected skip function.

One extra linear relation would force the sharper degree bound. On an additive domain D', all Lagrange denominators are equal. The coefficient of Z^127 in the interpolant is a nonzero shared weight times the sum of all 128 evaluations. With the lower half zero, degree at most 126 is therefore equivalent to the transmitted 64 values summing to zero.

An honest message satisfies this relation. An arbitrary one need not. In the toy example, [0,0,1,0] interpolates to a cubic, whereas the honest product residual has degree at most two. This is a direct example of the one-degree distinction at smaller size.

For the inspected encoding, a conservative discrepancy-root bound is **127/|K|**, rather than reusing 126/|K| for arbitrary received messages. The paper's 126 bound describes the true product polynomial; its precise concrete soundness discussion should be reconciled with the encoded-message space. This local distinction does not by itself invalidate the protocol: the same low-degree identity argument applies with 127. It is not a complete security audit or proof of the full batched construction.

Another view: any polynomial vanishing on D is divisible by W_D(Z)=∏_{d∈D}(Z−d). Honest H has quotient degree at most 62; the received-message space permits quotient degree at most 63. The extra message value relative to the honest quotient dimension accounts for that one linear relation.

## Actual source sequence

At pinned revision `4428e759dd27956bd5b9ae5025d67eb8ccc1accf`:

- `verifier/src/protocols/bitand.rs::verify_univariate_skip` receives exactly 64 field values, prepends 64 zeros, samples z, then calls `domain.extrapolate` and returns z and the resulting claim.
- `UnivariateRoundProver::univariate_claim` builds the same zero-prefixed evaluation vector and extrapolates over its 128-point domain. The local variable is called coeffs, but these are evaluations, not monomial coefficients (Lesson 005).
- The BitAnd entry point creates the seven-dimensional AES subspace for the message. After sampling z, it reduces that domain to dimension six to compute operand Lagrange weights and invokes the row/re-randomization prover.

The verifier's interpolation function body and these callers were inspected; no extra degree-126 condition is visible on this skip path. Detailed univariate-message construction and the entire re-randomization verifier are not yet traced. No Rust execution.

Recall: why doesn't supplying 64 zeros let the prover fake everything? It only constrains g; the verifier subsequently requires g(z) to match the witness-defined H(z), through reductions ending in authenticated operand values.

Next: how the honest prover computes those 64 off-domain values efficiently, using small-field transforms, lookup tables, and the fixed coordinates from Lesson 012.

Sources: [Blueprint §4.3.2](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex); [actual skip verifier](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/bitand.rs); [prover interpolation](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/bitand/prover.rs#L125); [domain extrapolation](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/math/src/univariate.rs#L127).

Validation: independently interpolated honest and arbitrary toy messages, checked their degrees and exact identity-test coincidences over F₁₆, and built a degree-126 polynomial vanishing on an explicit 64-point AES subspace. Its 64 off-domain values sum to zero; mutating one value breaks that condition. No Rust runtime, full proof test, or complete soundness accounting.
