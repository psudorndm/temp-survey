# Lesson 004 — A larger field element as a pair

2026-10-23 · Towers and packing · Introduced; understanding unchecked.

## Build one more level

Keep E = F₄ from Lesson 001, with α² = α + 1. Construct a larger field by adjoining θ with the rule

\[
\theta^2=1+\alpha\theta,\qquad
K=E[\Theta]/(\Theta^2+\alpha\Theta+1).
\]

Every element of K has a unique form a₀ + a₁θ, with a₀, a₁ in E. The pair (a₀,a₁) is its coordinate representation in the basis (1,θ). There are 4 choices for each coefficient, so K has 4² = 16 elements. Each coefficient takes two bits; the whole element takes four bits.

Why is K a field? The quadratic has no root in E. Evaluating at 0, 1, α, α + 1 gives 1, α, 1, α respectively, all nonzero. It is therefore irreducible. This is the second extension in the original Binius tower convention, renaming X₀ to α and X₁ to θ.

K is isomorphic to the F₁₆ from Lesson 003, but this is a different coordinate basis. Four-bit patterns cannot be equated across the two bases without the appropriate conversion.

## Arithmetic on pairs

Addition is coordinatewise:

\[
(a_0+a_1\theta)+(b_0+b_1\theta)
=(a_0+b_0)+(a_1+b_1)\theta.
\]

For multiplication, expand and substitute θ² = 1 + αθ:

\[
(a_0+a_1\theta)(b_0+b_1\theta)
=(a_0b_0+a_1b_1)
+(a_0b_1+a_1b_0+\alpha a_1b_1)\theta.
\]

Every operation on coefficients occurs in E, not in the ordinary integers. The coefficients mix during multiplication.

Example: θ is represented by (0,1). Its square is (1,α). Also θ(θ + α) = θ² + αθ = 1, so θ + α is its inverse.

Optional implementation idea: let p₀ = a₀b₀, p₁ = a₁b₁, and p₂ = (a₀ + a₁)(b₀ + b₁). The cross term a₀b₁ + a₁b₀ equals p₂ + p₀ + p₁. This is the characteristic-two Karatsuba identity: three general subfield products suffice, plus additions and multiplication by the fixed constant α. This algebraic observation is not a claim about which kernel a specific Binius build executes.

## Embedding, packing, and SIMD are different

| Operation | Meaning | Multiplication |
|---|---|---|
| Embed one E value | a → a + 0θ | Preserves the original field operations |
| Pack two E coordinates | (a₀,a₁) → a₀ + a₁θ | Produces one K value; coordinates mix |
| SIMD lanes over E | Store [a₀,a₁] as two independent values | [a₀b₀,a₁b₁], lane by lane |

The packing map is an E-linear bijection from E² to K, not a homomorphism from coordinatewise multiplication on E². It preserves addition and multiplication by an embedded E scalar, but not independent lane multiplication. It does not compress away information: two 2-bit coefficients still take four bits.

A sharp counterexample: the coordinate pair (0,1) represents θ in K and squares to (1,α). As two independent E lanes, the same pair squares to (0,1).

## Why it is a tower

Repeat degree-two extensions to get a chain

\[
\mathbb F_2\subset\mathbb F_4\subset\mathbb F_{16}
\subset\mathbb F_{256}\subset\cdots.
\]

The element widths double: 1, 2, 4, 8, 16, … bits. In the original notation, level Tᵢ has 2^(2ⁱ) elements and width 2ⁱ bits. The nested basis makes subfield inclusion compatible with zeroing higher coordinates. This is the setting in which a bit-preserving inclusion can be correct, unlike arbitrary zero extension between the unrelated polynomial bases in Lesson 003.

## Actual code connection and version boundary

The original Binius docs give precisely this tower construction. Current Binius64 instead uses the GHASH field representation for its challenge arithmetic; do not infer that its actual 128-bit multiplication is implemented recursively with the toy tower.

The pinned field library distinguishes the two abstractions:

- `ExtensionField<F>` in `field.rs` exposes `DEGREE`, `basis(i)`, `from_bases`, and coordinate access. The default `from_bases` delegates to `from_bases_sparse`; implementations determine the actual representation.
- `PackedField` in `packed.rs` describes a vector of scalar field elements, with `WIDTH` and per-lane iteration. Its SIMD packing vocabulary is distinct from assembling one extension-field element from base-field coordinates.

These interfaces were inspected. Concrete trait implementations, CPU kernels, and all conversions between packed representations remain untraced. We will return to them when studying commitment packing and performance.

## Recall and next step

1. How many E coordinates are needed for one K element? Two in this degree-two extension.
2. Does packing two small values preserve their independent multiplication? No; extension multiplication mixes coordinates.
3. How do you embed only a? Use (a,0), relative to this compatible basis.

Next: multilinear extensions. Turn a table of witness values into a polynomial that can be evaluated at the large-field challenges from Lesson 002. Further tower identities and Frobenius proofs remain on the roadmap.

Sources: [original Binius tower construction](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/docs/pages/basics/binius-v0/towers.mdx); [ExtensionField interface](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/field/src/field.rs#L154); [PackedField interface](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/field/src/packed.rs#L19).

Validation: checked irreducibility over E, all 256 products against the independent polynomial-basis F₁₆ model from Lesson 003 through a verified coordinate map, and inverses for all 15 nonzero elements. No Binius Rust execution or performance measurements.
