# Lesson 012 — Eight bit errors, no cancellation

2026-10-23 · Deterministic zerocheck · Introduced; understanding unchecked.

## Begin with two bits

In F₄ with basis (1,α), map two bit errors to e₀+αe₁. The four possible bit pairs map to 0,1,α,1+α, all distinct. Thus the packed value is zero exactly when both errors are zero. This is an injective linear encoding of bits into a field, not ordinary addition of indistinguishable errors.

The AES field is an eight-dimensional vector space over F₂. Given any F₂-basis w₀,…,w₇,

\[
\sum_{u=0}^7 e_u w_u=0\quad\Longleftrightarrow\quad e_0=\cdots=e_7=0
\qquad(e_u\in\mathbb F_2).
\]

This lets us combine eight Boolean constraint errors exactly, with no random collision probability for that group.

## Choose weights compatible with the equality polynomial

Let β be the class of X in AES's polynomial-basis presentation F₂[X]/(X⁸+X⁴+X³+X+1). The monomials 1,β,…,β⁷ form an F₂-basis. β is a polynomial-basis generator here; no claim that it generates the multiplicative group is needed.

Choose three fixed coordinates

\[
\rho=(\beta,\beta^2,\beta^4),\qquad
w_u=\operatorname{eq}_3(\rho,u),\quad u\in\{0,1\}^3.
\]

For example w₀=(1+β)(1+β²)(1+β⁴), while w₇=β·β²·β⁴=β⁷. In little-endian order, the eight weights are these AES byte patterns:

`ff, aa, cc, 88, f0, a0, c0, 80`.

They are not equal to the usual single-bit monomial basis, but they form another basis.

## Why these weights are independent

Products selecting β, β², and β⁴ give βʲ for every j from 0 through 7. The equality weights replace a factor 1 by 1+ρⱼ in each coordinate. This is an invertible basis change: the pair (1+ρⱼ,ρⱼ) spans the same space as (1,ρⱼ), since adding its two entries recovers 1.

Taking the three-fold tensor preserves invertibility, so the eight equality weights span the same space as 1,β,…,β⁷. More explicitly,

\[
w_u=\sum_{v:\operatorname{supp}(u)\subseteq\operatorname{supp}(v)}\beta^{v_0+2v_1+4v_2}.
\]

The Boolean-subset transformation is triangular with ones on its diagonal when indices are ordered compatibly with inclusion. It is invertible over F₂. Therefore no nonzero bit pattern can have weighted sum zero.

The fixed AES-to-GHASH embedding from Lesson 003 preserves all these field operations and F₂-linear independence. Write σⱼ=ι(ρⱼ) for the corresponding challenge-field coordinates; the eight eq₃(σ,u) remain independent over F₂ inside K.

## Which Binius64 coordinates are fixed?

These are **constraint-row coordinates**, not the bit axis changed in Lesson 011. At a fixed actual bit position, group eight consecutive constraint rows by their first three Boolean row-index bits. Write row index x=(u,v), where u has three coordinates and v has ℓ−3 coordinates.

For bit-valued errors e(u,v), define

\[
\overline e(v)=\sum_{u\in\{0,1\}^3}e(u,v)\operatorname{eq}_3(\sigma,u).
\]

Every group is zero exactly when all eight original errors in it are zero. Now only the group index needs a random point τ̄ in K^(ℓ−3):

\[
\sum_{u,v}e(u,v)\operatorname{eq}_{\ell}((\sigma,\overline\tau),(u,v))
=\sum_v\overline e(v)\operatorname{eq}_{\ell-3}(\overline\tau,v).
\]

If any bit error exists, the grouped error table is nonzero. Its MLE has total degree at most ℓ−3, giving at most (ℓ−3)/|K| initial random-test error. This is one component of the full protocol's soundness error, not its total security bound.

For fewer than three row coordinates, use the available prefix of ρ. Its 2^ℓ equality weights are still independent, so this initial grouping test is exact. The number of remaining random row coordinates is max(ℓ−3,0), matching the code.

## Essential limits

The errors must be bits before this packing step. Independence over F₂ does not prevent cancellation with arbitrary AES- or GHASH-valued coefficients. For example, take two nonzero weights w₀,w₁ and field-valued errors e₀=w₁, e₁=w₀. Their weighted sum is w₀w₁+w₁w₀=0.

For the oblong AND composition, errors at designated bit-domain points d in D and Boolean row indices are bits. That is where the independence argument applies. At an arbitrary off-domain bit-axis point, the composition may be general field-valued; do not apply the bit-independence claim there without further reasoning.

Fixing three coordinates of the **initial zerocheck weighting point** does not remove three fresh sumcheck-round challenges. The latter serve another role in validating the weighted claim. This trick is also distinct from the univariate skip, which changes how the bit axis is handled.

## Actual code

At pinned revision `4428e759dd27956bd5b9ae5025d67eb8ccc1accf`, `verifier/src/config.rs` defines `PROVER_SMALL_FIELD_ZEROCHECK_CHALLENGES` as Rijndael8b values `0x02`, `0x04`, `0x10`, exactly β,β²,β⁴ in the AES polynomial basis.

`verify_bitand_reduction` in verifier/src/verify.rs takes as many of these coordinates as the row count allows, embeds them, and samples only the remaining coordinates. `protocols/bitand/prove.rs` uses saturating subtraction to draw the same number of large-field coordinates. `UnivariateRoundProver::fold` concatenates the embedded fixed prefix with those sampled coordinates for its MLE-check point.

The tensor weights and linear independence explain why fixed choices can be sound here. Their small-field structure also supports efficient prover calculations; the actual byte-lookup/message kernel will show how. This inspection covers constants and coordinate assembly, not the full message kernel or transcript security.

Recall: why would replacing bit errors with arbitrary challenge-field elements invalidate the no-cancellation argument? The weights are independent only over F₂, not over the entire challenge field.

Next: the univariate-skip message. Explain how prescribed zeros on D and evaluations outside D describe the first-round polynomial, then how a random bit-axis challenge and row sumcheck authenticate it.

Sources: [Blueprint §4.3.1](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex); [fixed constants](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/config.rs#L29); [verifier coordinate assembly](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/verify.rs#L467); [prover coordinate assembly](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/bitand/prover.rs).

Validation: exhaustively checked all 256 eight-bit patterns map to distinct AES values, zero only for the zero pattern. Verified every shorter coordinate prefix, the Boolean-subset basis relation, and preservation by the actual retained AES-to-GHASH conversion table. Also checked a counterexample with field-valued errors. Independent arithmetic; no Rust execution.
