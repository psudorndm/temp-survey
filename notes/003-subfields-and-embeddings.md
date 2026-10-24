# Lesson 003 — Preserving arithmetic across fields

2026-10-23 · Subfields and embeddings · Introduced; understanding unchecked.

## The point

A field embedding is an injective map ι that preserves 0, 1, addition, and multiplication:

\[
\iota(a+b)=\iota(a)+\iota(b),\qquad
\iota(ab)=\iota(a)\iota(b),\qquad \iota(1)=1.
\]

The image is a subfield: a collection of elements inside the larger field that itself supports the smaller field's arithmetic. The bit encoding may change. Merely copying a representation into a wider register generally preserves XOR but need not preserve field multiplication.

## A tiny embedding worked completely

Our F₄ uses α² = α + 1. For this lesson choose F₁₆ = F₂[U]/(U⁴ + U + 1), with u denoting the class of U. Thus u⁴ = u + 1.

We cannot send α to u: u² does not equal u + 1. Instead take β = u² + u. Then

\[
\beta^2=u^4+u^2=(u+1)+u^2=\beta+1.
\]

β obeys the same equation as α, so define ι(a₀ + a₁α) = a₀ + a₁β. Substitution preserves the addition and multiplication rules. Explicitly:

| F₄ bits | Element | F₁₆ image | Bits in basis 1, u, u², u³ |
|---|---|---|---|
| 00 | 0 | 0 | 0000 |
| 01 | 1 | 1 | 0001 |
| 10 | α | u² + u | 0110 |
| 11 | α + 1 | u² + u + 1 | 0111 |

Within these four images, multiplication works exactly as in F₄. For instance, ι(α²) = ι(α + 1) = 0111, and (0110)² = 0111 in F₁₆.

Zero-extending `10` to `0010` instead would map α to u. Squaring would give `0100`, whereas zero-extending α² = `11` gives `0011`. That map fails the multiplication requirement.

## When can such an embedding exist?

For finite binary fields, F₂ᵐ embeds into F₂ⁿ exactly when m divides n. The image is the unique subfield of that size, though different embeddings can label that image differently. In our example both β and β + 1 satisfy α's defining polynomial, yielding two embeddings onto the same four-element subfield.

A useful derivation of the divisibility condition: if a large field K contains a subfield E, then K is a finite-dimensional vector space over E. If the dimension is d, counting elements gives 2ⁿ = (2ᵐ)^d, so n = md. Conversely, when m divides n, the roots of T^(2ᵐ) − T in F₂ⁿ form the subfield with 2ᵐ elements. The Frobenius/root argument behind this converse will be developed later; it is not yet derived here in full.

## Binius64's concrete conversion

Blueprint §2.1 embeds the Rijndael field F₂⁸ into the GHASH challenge field F₂¹²⁸; 8 divides 128. These use different polynomial bases and defining polynomials.

At pinned revision `4428e759dd27956bd5b9ae5025d67eb8ccc1accf`, `impl From<Rijndael8b> for Ghash128b` in `crates/field/src/fields/ghash.rs` indexes a table with the input byte. Table entry 2 is

`0x0dcb364640a222fe6b8330483c2e9849`.

This is the image of the Rijndael polynomial-basis generator, not the integer 2. Calling that image b, the entries evaluate each input's bit polynomial at b, and b satisfies the AES relation b⁸ + b⁴ + b³ + b + 1 = 0. The source also includes a property test comparing conversion after multiplication with multiplication after conversion.

Why zero extension fails here: in the AES field, raw `0x80 × 0x02 = 0x1b` after degree-eight reduction. In GHASH, interpreting those same raw inputs as polynomial coefficients gives `0x100`: degree eight is below the degree-128 modulus, so no reduction occurs. The conversion table supplies the compatible subfield coordinates instead.

An embedding can be zero extension in a deliberately compatible tower basis. The lesson is not that bit-preserving embeddings are impossible; it is that their correctness depends on the chosen bases and field arithmetic.

## Validation and next step

Independently verified all addition and multiplication pairs for the four-element toy embedding, and all nonzero inverses in the chosen F₁₆ model. For the actual source table, checked all 256 images are distinct, match evaluation at b, and b satisfies the AES polynomial. This algebraically validates the retained table's embedding under the stated moduli. It does not test Rust execution, hardware kernels, or every call site.

Recall: what goes wrong if α is mapped to an element that does not satisfy α² + α + 1 = 0?

Next: binary towers. Build larger elements as pairs of smaller-field elements; distinguish embedding a single small-field value from packing several independent values.

Sources: [pinned Blueprint §2.1](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex); [actual conversion table](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/field/src/fields/ghash.rs#L70); [Rijndael field definition](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/field/src/fields/rijndael.rs). Toy construction and its arithmetic supplied here.
