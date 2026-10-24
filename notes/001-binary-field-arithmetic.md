# Lesson 001 — A bit string can be a field element

2026-10-23 · Binary fields, part 1 · Introduced; understanding not yet checked.

## The idea to retain

A bit string does not determine its arithmetic. In a polynomial basis, its bits are coefficients of a polynomial. Addition is XOR. Multiplication is polynomial multiplication followed by a polynomial remainder operation.

## A complete tiny example

Build a field with four elements. Write a two-bit string as `b₁b₀`, representing `b₁α + b₀`, where each coefficient is 0 or 1.

| Bits | Field element |
|---|---|
| 00 | 0 |
| 01 | 1 |
| 10 | α |
| 11 | α + 1 |

We choose the rule `α² + α + 1 = 0`, equivalently `α² = α + 1`. The second equality uses the fact that adding and subtracting are the same in characteristic two.

Addition: `(α + 1) + α = 1`, represented by `11 XOR 10 = 01`. The two α terms cancel. There is no carry.

Multiplication:

\[
(\alpha+1)^2=\alpha^2+2\alpha+1=\alpha^2+1
=(\alpha+1)+1=\alpha.
\]

The term `2α` vanishes because `1 + 1 = 0`. So `11 × 11 = 10` in this field.

Compare the same input bit patterns:

| Operation | Result |
|---|---|
| Bitwise AND: `11 & 11` | `11` |
| Ordinary integer multiplication: 3 × 3 | 9, or `1001` |
| Field multiplication in this example | `10` |

The field is F₄ (equivalently, \(\mathbb F_{2^2}\)), with four elements, not arithmetic modulo the integer 4. In particular `1 + 1 = 0`, and every nonzero element has a multiplicative inverse.

## Why the rule produces a field

The formal construction is

\[
\mathbb F_4=\mathbb F_2[X]/(X^2+X+1),\qquad
\alpha=[X].
\]

Read this as: take polynomials with bit coefficients, and identify two polynomials when their difference is a multiple of `X² + X + 1`. Each element has a unique remainder of degree less than two, giving exactly four possible coefficient strings.

The chosen degree-two polynomial is **irreducible**: it cannot be factored into two positive-degree polynomials over F₂. A quadratic is irreducible exactly when it has no root in its coefficient field. Substituting 0 gives 1; substituting 1 also gives 1. Neither is a root.

Irreducibility matters because it guarantees division by every nonzero element. Here we can see all inverses directly:

- `1 · 1 = 1`.
- `α(α + 1) = α² + α = (α + 1) + α = 1`.

Thus α and α + 1 are each other's inverses. For a reducible modulus the quotient can have nonzero elements whose product is zero; it would fail to be a field. For example, modulo `X² + X = X(X + 1)`, the nonzero classes of X and X + 1 multiply to zero.

## Connection to Binius64

Blueprint §2.1 uses this same general construction at larger sizes. Its challenge field is the GHASH field

\[
K=\mathbb F_2[X]/(X^{128}+X^7+X^2+X+1).
\]

A field element has 128 bit coefficients. The defining relation lets us replace `X¹²⁸` with `X⁷ + X² + X + 1`. Multiplication mixes coefficients; it is not bitwise AND. The toy two-bit field explains the construction, not Binius64's literal field parameters or complete multiplication implementation.

At pinned revision `4428e759dd27956bd5b9ae5025d67eb8ccc1accf`, `crates/field/src/fields/ghash.rs` names that modulus. Its `test_ghash_mul` checks that raw bit patterns `3 × 3` produce `5`: `(X + 1)² = X² + 1`. No degree-128 reduction is needed for that product. This differs from the toy F₄ result because the defining modulus is different.

The same source checks that `(1 << 127) × 2` produces `0x87`, whose set bits are positions 7, 2, 1, and 0. This is the defining GHASH reduction relation appearing directly in code. These are source assertions inspected, not Rust tests executed in this project. Low-level multiplication kernels remain to be traced.

## Recall

1. Why do equal polynomial terms cancel when adding?
2. In our toy field, what is `10 × 10`?
3. Why can't we use an arbitrary polynomial as the modulus?

Answers: coefficients add in F₂; `11` because α² = α + 1; the modulus must be irreducible to guarantee a field.

Next: explain why the protocol needs both bit-valued data and large-field random challenges. Then develop subfields, embeddings, and binary towers.

Sources: [Blueprint §2.1](https://www.binius.xyz/spec.pdf); [pinned whitepaper source](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex); [GHASH definition](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/field/src/fields/ghash.rs#L4); [multiplication examples in source](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/field/src/fields/ghash.rs#L346).

Validation: independently computed all 16 toy-field multiplication results and verified an inverse for each nonzero element using polynomial multiplication and remainder. This checks the worked arithmetic, not the Binius implementation.
