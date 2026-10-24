# Lesson 011 — One long axis instead of six Boolean axes

2026-10-23 · BitAnd representation · Introduced; understanding unchecked.

## Re-label a word's 64 bit positions

The baseline table a(i,x) from Lesson 010 has a six-bit position index i and a Boolean constraint-row index x. Choose six linearly independent AES-field elements ζ₀,…,ζ₅. Map i to a field point

\[
d_i=i_0\zeta_0+\cdots+i_5\zeta_5.
\]

The 64 distinct dᵢ form a six-dimensional F₂-linear subspace D inside F₂⁸, then embedded in the GHASH challenge field K. This labels 64 bit positions by 64 distinct field elements. It does not turn the six Boolean variables into their arithmetic sum inside the old polynomial by substitution.

D is generally not a subfield. In fact F₂⁶ cannot be a subfield of F₂⁸ because 6 does not divide 8 (Lesson 003). Addition closure is enough for the domain; its elements need not be closed under multiplication.

## Interpolate along that axis

For each dᵢ define the univariate Lagrange selector

\[
L_i(Z)=\prod_{j\ne i}\frac{Z-d_j}{d_i-d_j}.
\]

Every denominator is nonzero because the domain points are distinct. Lᵢ(dⱼ) is 1 when i=j and 0 otherwise. The degree is 63 for a 64-point domain.

If ã(i,X) denotes the multilinear extension along rows for the bit position i, set

\[
\boxed{\widehat a(Z,X)=\sum_{i=0}^{63}L_i(Z)\widetilde a(i,X).}
\]

At Z=dᵢ, it recovers that bit's row polynomial. It has degree at most 63 in Z and individual degree at most one in each row coordinate. The degree bounds and values on D×{0,1}^ℓ determine it uniquely by univariate interpolation followed by MLE uniqueness. This mixed-degree representation is called **oblong-multilinearization**.

This is a new interpolation representation of the same table, not a field embedding, SIMD packing operation, or compression of the table's information content.

## A four-position example

Use the embedded F₄ domain D={0,1,α,α+1} from Lesson 003, and assign the four values [1,0,1,1] to those points in that order. The unique degree-below-four interpolant is

\[
P(Z)=1+Z+Z^2+Z^3=(Z+1)^3.
\]

Check its values at all four domain points. For the two nonzero domain points other than 1, Z+1 is a nonzero F₄ element and its cube equals 1. This cubic and the earlier two-variable MLE 1+X+XY encode the same four table entries on differently labeled domains; they are different formal polynomials.

The polynomial can also be evaluated outside D. In the polynomial-basis F₁₆ from Lesson 003, α is represented by u²+u and u is outside D. Then P(u)=1+u+u²+u³. These off-domain values are determined by the chosen interpolation, not newly supplied table entries.

## What happens to the AND relation?

Interpolate all three operand tables this way and define

\[
\widehat G(Z,X)=\widehat a(Z,X)\widehat b(Z,X)+\widehat c(Z,X).
\]

It vanishes at every (dᵢ,x) in D×{0,1}^ℓ exactly when every bitwise AND relation holds. Its degree in Z is at most 63+63=126, and its individual row degrees are at most two.

As before, vanishing on the checking grid does not mean the product polynomial is identically zero everywhere. The degree-126 composition need not equal the degree-below-64 interpolation of the Boolean error table. A later protocol must prove vanishing on D, not simply demand zero at an arbitrary Z outside D.

After a verifier challenge r_Z, each operand specialization â(r_Z,X) is an ordinary multilinear polynomial in the remaining row variables. The data for it is one field value per row, computed as

\[
\widehat a(r_Z,x)=\sum_i\operatorname{bit}_i(A[x])L_i(r_Z).
\]

Thus every source word is folded using the same vector of 64 weights. This fold is F₂-linear on bits. It does not preserve coordinatewise bit multiplication: folding A AND B generally differs from multiplying the two folded field values.

## Why this helps, and what remains to prove

The long axis can be handled by one univariate message instead of six successive Boolean-variable rounds. With the domain and input bits in a small field, substantial interpolation/message work can stay in that small field before the large-field bit-axis challenge is introduced. Row-weight aggregation may still involve large-field challenges, so do not claim that the entire first message uses only AES arithmetic. The deterministic row-coordinate trick and the actual lookup kernel explain the detailed savings later.

Changing representation alone does not establish soundness or a speed factor. The next steps are deterministic zerocheck and the univariate-skip protocol, followed by detailed kernels and authenticated operand claims.

## Implementation details inspected

At pinned revision `4428e759dd27956bd5b9ae5025d67eb8ccc1accf`, `UnivariateRoundProver::fold` obtains the 64-point subspace and calls `lagrange_evals` at the bit-axis challenge. It passes those weights to BitAxisFolder.

In `math/src/univariate.rs`, the BinarySubspace implementation of `lagrange_evals` exploits a shared denominator. For every i, the differences dᵢ−dⱼ for j≠i run over the nonzero elements of D, so

\[
L_i(z)=w\prod_{j\ne i}(z-d_j),\qquad
w=\left(\prod_{d\in D\setminus\{0\}}d\right)^{-1}.
\]

The code computes prefix/suffix products to produce all weights in linear work in |D|. It never divides by z−dᵢ, so z landing on a domain point is handled naturally. The shared weight is computed by a subspace-polynomial recurrence, inverted in the domain's field, and then embedded into the arithmetic field. The recurrence's full derivation is a later coding-theory topic; the body and its invariant were inspected, not exhaustively runtime-tested.

In `prover/src/fold_word/bit_axis.rs`, `fold_bitand_operands` streams A and B and writes three folded output buffers. The third uses `self.tables.fold(a & b)`, both for full packed chunks and for the short tail. This is the concrete source of the derived C described in Lesson 010. It does not compute folded C by multiplying folded A and folded B. The byte lookup table builder and allocation/output helpers remain to be traced.

## Recall and next step

- How can six Boolean coordinates become one variable? Interpolate on 64 distinct field points, allowing degree up to 63 on that axis.
- Are the 64 points a field? No; D is an additive subspace inside the AES field.
- What is the product degree bound on that axis? At most 126 for two degree-at-most-63 operands.

Next: how fixed AES-field challenges can pack eight bit-valued constraint errors without cancellation, reducing the number of random row-coordinate challenges. Then the univariate message and its soundness.

Sources: [Blueprint §2.2.1 and §4.3](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex); [Lagrange weights](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/math/src/univariate.rs#L94); [bit-axis folding](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/fold_word/bit_axis.rs#L128).

Validation: checked the toy cubic and shared-weight algorithm at all F₁₆ points; for an explicit 64-point AES subspace, checked shared denominators, selector and sum-of-weights identities, and agreement with independent textbook Lagrange products at eight representative points including on/off-domain values. Confirmed the domain is not multiplicatively closed. No Rust execution or performance measurement.
