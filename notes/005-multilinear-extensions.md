# Lesson 005 — A table becomes a polynomial

2026-10-23 · Multilinear extensions, part 1 · Introduced; understanding unchecked.

## Start with two values

Given a table f(0) = a and f(1) = b, define

\[
\widetilde f(X)=(1-X)a+Xb=a+X(b-a).
\]

At X = 0 this selects a; at X = 1 it selects b. It is the unique degree-at-most-one polynomial with those endpoint values. In characteristic two, subtraction equals addition, so the last expression is a + X(b + a). A table [1,0] therefore extends to 1 + X. At X = α in our toy F₄, its value is 1 + α, which is not a bit.

This is interpolation over a field, not an average of real numbers. The variable X labels an evaluation point; it is not the polynomial-basis generator α used to represent a field element.

## Four values need two variables

Index a four-entry table by i = x₀ + 2x₁. This is little-endian: x₀ varies fastest, matching the Blueprint convention.

| i | (x₀,x₁) | Witness value wᵢ |
|---|---|---|
| 0 | (0,0) | 1 |
| 1 | (1,0) | 0 |
| 2 | (0,1) | 1 |
| 3 | (1,1) | 1 |

For arbitrary entries w₀,w₁,w₂,w₃, interpolation gives

\[
\widetilde w(X,Y)=w_0(1-X)(1-Y)+w_1X(1-Y)
+w_2(1-X)Y+w_3XY.
\]

Each weight selects one table entry at Boolean inputs. Substituting this example's values and simplifying in characteristic two gives

\[
\boxed{\widetilde w(X,Y)=1+X+XY.}
\]

Check the four rows directly. Outside the Boolean cube, the same polynomial still gives a well-defined field value: w̃(α,0) = 1 + α, and w̃(α,α) = 1 + α + α² = 0.

## What multilinear means

Each variable has individual degree at most one. XY is allowed; X² is not. Total degree can exceed one: 1 + X + XY has total degree two.

A table of 2ⁿ entries is indexed by n bits, hence gives an n-variable multilinear extension. Its formula is

\[
\widetilde w(r)=\sum_{u\in\{0,1\}^n}w(u)
\prod_{j=0}^{n-1}\begin{cases}1-r_j,&u_j=0,\\r_j,&u_j=1.\end{cases}
\]

The product is 1 at its designated Boolean index and 0 at the other indices. This proves existence. For uniqueness, a multilinear polynomial can be decomposed along its last variable as (1 − X)g₀ + Xg₁. Agreement on the Boolean cube determines both endpoint polynomials; induction on the number of variables determines the entire polynomial.

This restriction matters. In Lesson 002, X and X² agreed on both bits but were different formal polynomials. X² is excluded from the degree-one interpolation space, eliminating that ambiguity.

## Values, coefficients, and security boundaries

The table stores evaluations, not coefficients in the monomial basis. Here the evaluation table is [1,0,1,1], whereas coefficients of [1,X,Y,XY] are [1,1,0,1]. Both describe the same polynomial in different bases. Evaluation-table entries can also be called coefficients in the Lagrange basis; keep the basis explicit when reading code comments.

This correspondence does not compress the witness: there remain 2ⁿ independent table entries. Nor is a claimed evaluation automatically trustworthy. Commitments and reduction protocols must authenticate it.

Products also need care: multiplying two multilinear polynomials can raise individual degrees. For f̃(X) = X, its square is X², while extending the pointwise-squared Boolean table gives X again. These agree on bits but need not agree at an extension-field point. This distinction is essential for constraint checks and sumcheck later.

## Implementation connection

At pinned revision `4428e759dd27956bd5b9ae5025d67eb8ccc1accf`, `crates/math/src/multilinear/evaluate.rs` evaluates a polynomial from its hypercube evaluation buffer. No conversion to monomial coefficients is required.

- `evaluate_inplace_scalars` fixes coordinates from highest index to lowest. Each step replaces two endpoint values a,b by a + r(b − a), halving the active table.
- `evaluate_inplace` uses the corresponding packed fold operation. The concrete fold kernel has not yet been traced.
- The borrowed `evaluate` path builds a partial equality-weight tensor, computes inner products of chunks, and evaluates the residual table. Its source describes scratch memory on the order of the square root of the input count; this is an algorithmic observation, not a measured benchmark.

The first scalar loop's update and ordering were read directly. Equality-weight generation and concrete packed folds are next; reading this file alone does not validate every backend.

## Recall and next step

- Does multilinear mean total degree one? No; it means degree at most one in each variable.
- Must evaluating a bit-valued table's extension give a bit? Only at Boolean inputs; extension-field points can give other field values.
- Why are the Boolean table entries enough? Endpoint interpolation, applied recursively, determines a unique multilinear polynomial.

Next: equality indicators and folding. Understand the weights in the general formula and how fixing one variable halves the stored table. Then sumcheck.

Sources: [Blueprint §2, multilinear notation](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex); [official MLE explanation](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/docs/pages/blueprint/math/multilinears.mdx); [actual evaluation routines](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/math/src/multilinear/evaluate.rs).

Validation: for all 256 F₄-valued four-entry tables, verified interpolation equals direct monomial evaluation and highest-variable-first folding at all 16 F₄² points. Checked little-endian Boolean indexing and the worked examples. Independent mathematical verification; no Binius Rust execution.
