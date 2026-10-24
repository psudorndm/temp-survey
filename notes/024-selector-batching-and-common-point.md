# Lesson 024 — Six challenges combine 64 selector claims

Introduced: 2026-10-23. Understanding unchecked. Code revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf. Blueprint revision: 05061c6f9eca316b6844396883afb9ae37ff2fe9.

**One idea:** index 64 claims by six bits, then evaluate their claim table at a fresh six-coordinate point. This gives one weighted claim for a shared sumcheck.

Prerequisites: [equality weights](006-equality-weights-and-folding.md), [different-point batching](017-batching-and-rerandomization.md), [GKR](021-one-gkr-layer-and-line-reduction.md), and [twisted selectors](023-frobenius-twists-of-mle-claims.md).

## 1. What comes into phase 3

Let $K=\mathbb F_{2^m}$, $q=|K|$, and let $\ell$ be the row-variable count. The previous reduction supplies 64 claimed values

$$
u_i=\widetilde T_i(r_i),\qquad
T_i[x]=1+b_i[x](A[x]-1),\qquad 0\leq i<64.
$$

Here $A[x]$ is the variable base and $b_i[x]$ is exponent bit $i$. The tables share $A$, but have distinct bit selectors. The points $r_i$ are inverse-Frobenius transforms of one old point; they are correlated.

There is also a claim about the root of the output-side multiplication tree:

$$
c=\sum_{x\in\{0,1\}^{\ell}}
\operatorname{eq}_{\ell}(r_C,x)
\widetilde C_{\mathrm{lo}}(x)\widetilde C_{\mathrm{hi}}(x).
$$

The $C$ columns here are exponentiated output-word tables used by IntMul, not the raw integer words. This identity describes the MLE of their pointwise product, which generally differs from the product of their MLEs at $r_C$.

## 2. Why six field elements give 64 weights

Write the binary index as $i=\sum_{j=0}^5 i_j2^j$. Sample

$$
\gamma=(\gamma_0,\ldots,\gamma_5)\in K^6,\qquad
w_i=\operatorname{eq}_6(\gamma,i)
=\prod_{j=0}^5\bigl((1-\gamma_j)(1-i_j)+\gamma_ji_j\bigr).
$$

The aggregate claim is

$$
S=\sum_{i=0}^{63}w_i u_i.
$$

These weights sum to one. They are neither 64 independent random field elements nor consecutive powers of one random element. They are the interpolation weights of a six-variable MLE.

For a fixed nonzero error vector $e=(e_0,\ldots,e_{63})$, its MLE

$$
E(\Gamma)=\sum_i\operatorname{eq}_6(\Gamma,i)e_i
$$

is nonzero by uniqueness of multilinear interpolation, and has total degree at most six. Therefore, with independent uniform fresh coordinates,

$$
\Pr_\gamma[E(\gamma)=0]\leq\frac{6}{q}.
$$

The old points may be correlated. What matters for this local batching argument is that the error vector is fixed before $\gamma$ is sampled. This is a reduction argument conditional on the underlying tables/claims being fixed and ultimately authenticated.

## 3. Tiny example: four claims, two coordinates

In $K=\mathbb F_4$, with $\alpha^2=\alpha+1$, choose $\gamma=(\alpha,\alpha)$. In little-endian bit-index order,

$$
(w_{00},w_{01},w_{10},w_{11})
=\bigl((1-\alpha)^2,\alpha(1-\alpha),(1-\alpha)\alpha,\alpha^2\bigr)
=(\alpha,1,1,\alpha+1).
$$

The subscripts in this displayed list are ordinary binary numerals: index $01$ has low coordinate one. For errors $(1,1,0,0)$, the unweighted sum vanishes in characteristic two, but the weighted error is

$$
\alpha\cdot1+1\cdot1=\alpha+1\ne0.
$$

This particular challenge exposes this particular error. Random batching gives a probability bound, not a guarantee for every challenge.

## 4. The actual shared sumcheck polynomial

The selector aggregate becomes a Boolean sum of the polynomial

$$
F_{\mathrm{sel}}(X)=
\sum_{i=0}^{63}w_i\operatorname{eq}_{\ell}(r_i,X)
\left[1+\widetilde b_i(X)(\widetilde A(X)-1)\right].
$$

Thus $S=\sum_x F_{\mathrm{sel}}(x)$ when the input selector claims are correct. Define

$$
F_C(X)=\operatorname{eq}_{\ell}(r_C,X)
\widetilde C_{\mathrm{lo}}(X)\widetilde C_{\mathrm{hi}}(X).
$$

The ordinary batch driver samples a separate scalar $\theta$ and checks

$$
S+\theta c
=\sum_{x\in\{0,1\}^{\ell}}
\left(F_{\mathrm{sel}}(x)+\theta F_C(x)\right).
$$

Each summand has individual degree at most three: equality contributes one and a product of two MLEs contributes two. The selector sum does not increase the degree. Because the old $r_i$ differ, one common equality indicator cannot be factored out of the entire selector aggregate.

The local error bounds are distinct: at most $6/q$ for the first tensor batch; at most $1/q$ for mixing a fixed nonzero pair of aggregate errors with $\theta$; and at most $3\ell/q$ for ordinary degree-three sumcheck with authenticated terminal values. These statements do not constitute the full IntMul or Fiat–Shamir security proof.

## 5. Closing the sumcheck at one new point

Let $r^\star$ be the new row point. The prover sends

$$
\beta_i=\widetilde b_i(r^\star)\quad(0\leq i<64),\qquad
a^\star=\widetilde A(r^\star),\qquad
c_{\mathrm{lo}}^\star=\widetilde C_{\mathrm{lo}}(r^\star),\qquad
c_{\mathrm{hi}}^\star=\widetilde C_{\mathrm{hi}}(r^\star).
$$

The verifier checks its terminal sumcheck target against

$$
\sum_i w_i\operatorname{eq}_{\ell}(r_i,r^\star)
\left[1+\beta_i(a^\star-1)\right]
+\theta\operatorname{eq}_{\ell}(r_C,r^\star)
c_{\mathrm{lo}}^\star c_{\mathrm{hi}}^\star.
$$

All columns are now evaluated at the same row point. This scalar consistency check still requires later authentication of the sent column evaluations.

The code receives all 64 bit values, the shared base value and the two output-side values before sampling another fresh six-coordinate point $r_I^b$. It compresses the bit outputs to

$$
B^\star=\sum_i\operatorname{eq}_6(r_I^b,i)\beta_i
=\widetilde b(r^\star,r_I^b),
$$

where the combined table uses low row coordinates and high bit-index coordinates. This is a second tensor compression, for carrying the exponent claim into later phases. For a fixed incorrect bit-value vector at the new row point, fresh recombination misses that vector with probability at most $6/q$, assuming the later scalar claim is correctly authenticated. Sampling $r_I^b$ itself does not authenticate the bits.

Actual order: sample $\gamma$; sample the batch coefficient; exchange sumcheck rounds; receive bit/base/output values; sample $r_I^b$; form the bit recombination and check the terminal identity. The terminal check uses the earlier values. Sumcheck challenges arrive in high-variable binding order and are reversed into ordinary coordinate order.

## 6. How the selector prover computes a round

SelectorMlecheckProver keeps the shared base column, 64 selectors, 64 equality trackers and 64 internal prime claims, while exposing one weighted round polynomial to the outer driver.

For one selector, strip the current variable's equality factor. The remaining prime round $R(T)$ is quadratic. For the halves $A_0,A_1,b_0,b_1$, the composition samples needed inside its Boolean sum are

$$
\begin{aligned}
\text{at }1 &: 1+b_1(A_1-1),\\
\text{at }\infty &: (b_0+b_1)(A_0+A_1).
\end{aligned}
$$

The symbol $\infty$ means the coefficient of $T^2$, not a field element. In characteristic two, differences equal sums. The highest coefficient drops the constant and linear terms of the selector composition.

The kernel weights these samples by equality factors for the other variables and sums over their Boolean indices. It interpolates each prime round, restores its linear equality factor and accumulated prefix factor, then combines the resulting full round polynomials with $w_i$. Those restored round polynomials have degree at most three.

After the challenge, it evaluates each stored prime polynomial to obtain the next internal claim, folds each equality tracker, folds selector state and folds the shared base along the highest remaining variable. At finish it returns the 64 bit-column values followed by the one shared base value. The outer driver also writes the two $C$ column values.

## 7. The engineering choices in the inspected bodies

- ChunkedEqTracker factors the remaining equality table into an inner chunk on at most eight low coordinates and a suffix indexed by chunk. The current highest coordinate stays separate until interpolation. Previously bound coordinates contribute an accumulated equality prefix.
- The kernel loops over chunks, then selectors. Its source comments explain the aim of reusing small buffers in cache; this is source-level intent, not a measured speedup.
- Equality-times-composition products accumulate in a wide unreduced representation, with reduction once per chunk. The composition's own field product is reduced normally because it feeds the wide multiplication. It is incorrect to say every multiplication in the kernel postpones reduction.
- Before any fold, selector values are bits, so masks replace their field multiplications. After one fold their values generally belong to the extension field. The code's is_binary test checks that no challenge has yet been bound.
- Packed storage persists longer than Boolean-valuedness. BinarySwitchover retains bit-transposed words for the first $h=\min(\ell,6)$ bindings and reads partial field values through byte-table lookup against challenge tensor weights. The $h$th fold materializes a field buffer, then ordinary folds continue.
- With $s=64$ selectors, the materialized buffer stores selector $i$ at offset $\mathrm{lo}\,s+i$. Thus selectors occupy low buffer-index bits in this internal layout. This differs from the semantic combined witness table's row-low/bit-index-high convention above; do not identify the two layouts.

The switchover constructor/fold/finish contracts and selector read interfaces were inspected. The full transpose and lookup backend, allocator behavior, SIMD execution and measured performance remain unvalidated.

## 8. A precise exceptional-coordinate issue to revisit

The optimized prime-round interpolation computes its missing value as

$$
R(0)=\frac{s-\alpha R(1)}{1-\alpha},\qquad
s=(1-\alpha)R(0)+\alpha R(1).
$$

This determines $R(0)$ when $\alpha\ne1$. RoundEvals::eval_at_zero_eq uses invert_or_zero for the denominator, and its nearby property tests exclude $\alpha=1$. At $\alpha=1$, the weighted claim supplies only $R(1)$ and does not determine $R(0)$.

For example, take $R(T)=1+T^2$. With $\alpha=1$, the claim and $R(1)$ are both zero, and the leading coefficient is one. The helper recovers zero for the missing constant and reconstructs $R'(T)=T+T^2$, which agrees at one but is a different prime polynomial.

This is a local information-loss example in the prover shortcut. The earlier division-free verifier coefficient recovery from Lesson 008 is a different algorithm. We have not traced the concrete challenge sampler or established the full execution/completeness consequence of this exceptional case. Inverse-Frobenius points also satisfy $(r_i)_j=1$ simultaneously when their original coordinate equals one, so these are not 64 independent exceptional events. Track sampler policy and full error accounting before drawing a protocol-level conclusion.

## 9. Source anchors and validation

- [IntMul verifier, verify_phase_3](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/intmul/verify.rs): six-coordinate aggregation, degree-three batch, receive order, terminal check and exponent recombination.
- [IntMul prover orchestration](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/intmul/prove.rs): matching selector weights and two-prover batch.
- [SelectorMlecheckProver](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/intmul/selector_mle.rs): constructor and execute/fold/finish bodies.
- [ChunkedEqTracker](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip-prover/src/sumcheck/eq_tracker.rs): factorization, interpolation and folding.
- [RoundEvals](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip-prover/src/sumcheck/round_evals.rs): one/infinity samples, interpolation and the exceptional denominator.
- [BinarySwitchover](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/intmul/switchover.rs): transparent/field storage states, materialization, bit masks and value interfaces.

Independent scalar checks enumerate all 255 nonzero four-entry error tables over $\mathbb F_4$ at all 16 challenge points; the maximum zero count is seven. In 24 two-row-variable fixtures over $\mathbb F_{16}$, a six-coordinate batch combines 64 selectors with correlated Frobenius-shaped points. Explicit polynomial expansion verifies individual degree at most three, the initial Boolean sum, both ordinary sumcheck rounds and the terminal closure for all 256 challenge pairs per fixture: 6,144 complete pairs. Recombination agrees with the combined eight-variable bit/row MLE. All 61,440 quadratic interpolation cases with $\alpha\ne1$ are checked over $\mathbb F_{16}$; the $\alpha=1$ information-loss example is checked separately.

These are independent mathematical models. They do not execute Rust, packed kernels, channels, a full proof or a security audit.

**Remember:** six coordinates weight 64 old claims; one sumcheck moves their input columns to one row point; later reductions still authenticate those columns.

**Recall:** why must the 64 claimed bit values be received before sampling the recombination point?

Next: actual fixed-base exponentiation with four 16-bit limbs, and how lookup index binding connects the exponentiated tables to the integer words.
