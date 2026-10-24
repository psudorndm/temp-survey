# Lesson 015 — Subspace polynomials make recursive evaluation possible

2026-10-23 · Additive NTT foundations · Introduced; understanding unchecked.

## Why use additive domains?

A conventional length-64 roots-of-unity transform needs an element of multiplicative order 64. In $\mathbb{F}_{2^{m}}$, the nonzero multiplicative group has odd order $2^{m}-1$, so such an element does not exist. Equivalently, $Z^{2^r}-1=(Z-1)^{2^r}$ in characteristic two and has only the root 1. This rules out that particular power-of-two multiplicative transform, not fast transforms in general.

Instead use an additive subspace spanned by an ordered independent list $\beta _{0},\ldots ,\beta _{n-1}$. The actual AES lookup path uses a polynomial-basis binary subspace and generic normalized constants; do not assume every domain is a specially chosen Gao–Mateer basis.

## Grow the vanishing polynomial with the subspace

Define $V_{j}=\operatorname{span}_{\mathbb{F}_2}(\beta _{0},\ldots ,\beta _{j-1})$, with $V_{0}=\{0\}$, and

$$
W_j(Z)=\prod_{v\in V_j}(Z-v),\qquad \deg W_j=2^j.
$$

The next space is the disjoint union $V_{j+1}=V_{j} \cup (\beta _{j}+V_{j})$. These polynomials are additive ($\mathbb{F}_2$-linear): $W_{j}(x+y)=W_{j}(x)+W_{j}(y)$. Thus

$$
\boxed{W_{j+1}(Z)=W_j(Z)\bigl(W_j(Z)+W_j(\beta_j)\bigr).}
$$

Why additivity holds: $W_{0}(Z)=Z$ is additive. The recurrence is $W_{j}^{2}+cW_{j}$, and squaring preserves additivity in characteristic two. Induction proves both additivity and the product formula: translating $V_{j}$ by $\beta _{j}$ replaces $W_{j}(Z)$ by $W_{j}(Z)+W_{j}(\beta _{j})$.

Only powers Z, $Z^{2}$, $Z^{4}$, … occur. Such a polynomial is called linearized. Its ordinary polynomial degree can be large even though its induced map is linear over $\mathbb{F}_{2}$.

## The four-element example

Use $\beta _{0}=1$ and $\beta _{1}=\alpha$ in $\mathbb{F}_{4}$, where $\alpha ^{2}=\alpha +1$. Then

$$
W_0(Z)=Z,\quad W_1(Z)=Z^2+Z,\quad W_1(\alpha)=1,
\quad W_2(Z)=W_1(Z)^2+W_1(Z)=Z^4+Z.
$$

$W_{1}$ is zero on {0,1} and one on $\{\alpha ,\alpha +1\}$. For a polynomial of degree below four, write $f=A+W_{1}B$ with A and B of degree below two. On the first half-domain we evaluate A; on the second half-domain we evaluate A+B. That is the recursive splitting idea in its smallest useful form.

## Normalize to make sibling cosets differ by one

In a general basis, $c_{j}=W_{j}(\beta _{j})$ is nonzero but need not equal 1. Define

$$
s_j(Z)=W_j(Z)/c_j.
$$

Then $s_{j}$ vanishes on $V_{j}$ and $s_{j}(\beta _{j})=1$. On any translated coset $a+V_{j}$ it has the constant value $t=s_{j}(a)$, because it is additive and vanishes on the directions $v\in V_{j}$. On the sibling $a+\beta _{j}+V_{j}$ the value is t+1.

For $\beta _{0}$ not equal to one, even $s_{0}$ is $Z/\beta _{0}$, not simply Z. Normalization must be derived from the actual ordered basis; the AES path's first basis element is one.

The source's arbitrary-point helper also uses a normalized recurrence. If $d=s_{j}(\beta _{j+1})$, then

$$
s_{j+1}(Z)=\frac{s_j(Z)(s_j(Z)+1)}{d(d+1)}.
$$

The denominator is nonzero for an independent basis. All its inverses can be precomputed from the domain; evaluating at a fresh Z then needs no new inversion.

## The novel polynomial basis

For an integer i with binary digits $i_{j}$, define

$$
N_i(Z)=\prod_{j:i_j=1}s_j(Z).
$$

Its degree is $\sum _{j}i_{j}2^{j}=i$. Therefore $N_{0},\ldots ,N_{2^{n}-1}$ form a basis for all polynomials of degree below $2^{n}$: their distinct degrees give a triangular invertible change from the monomial basis.

The forward additive NTT interprets its inputs as coefficients $m_{i}$ in $f(Z)=\sum _{i}m_{i}N_{i}(Z)$, and outputs evaluations on the additive domain. The inverse takes those evaluations back to novel-basis coefficients. Neither vector should automatically be interpreted as monomial coefficients.

In the $\mathbb{F}_{4}$ example, the four basis polynomials are 1, Z, $Z^{2}+Z$, and $Z^{3}+Z^{2}$. A degree-below-four polynomial can be grouped as $f=A+s_{1}B$, with $A=m_{0}+m_{1}s_{0}$ and $B=m_{2}+m_{3}s_{0}$.

## The butterfly identity

On sibling cosets where $s_{j}$ takes t and t+1, recursively evaluate the two smaller polynomials

$$
A+tB,\qquad A+(t+1)B.
$$

For coefficient pair (a,b), one butterfly is

$$
u=a+tb,\qquad v=u+b.
$$

The inverse is $b=u+v$, then $a=u+tb$. It requires no division or 1/N scaling. In characteristic two, the butterfly matrix has determinant one.

For $N=2^{n}$ novel-basis coefficients, each of n layers has N/2 butterflies, giving O(N log N) arithmetic for the transform with available domain constants. This is not a claim that arbitrary monomial-basis input is converted for free, nor a benchmark of the implementation.

## Actual code alignment

At pinned revision `4428e759dd27956bd5b9ae5025d67eb8ccc1accf`:

- `ntt/mod.rs` explicitly defines the transform's novel-basis coefficient input and evaluation output, with references to Lin–Chung–Han and the Binius ring-switching paper.
- `domain_context.rs::generate_evals_from_subspace` starts with basis values, applies $W_{i}$ recurrence on the remaining basis elements, and divides each row by its first entry to normalize it. GenericOnTheFly then computes block twiddles from subset sums of normalized polynomial values on the higher basis elements.
- `reference.rs` forwards a pair with `u += v * twiddle; v += u`, and inverts it with `v += u; u += v * twiddle`, traversing layers in reverse order. This is precisely the butterfly above. It processes the highest split first; neighbors become adjacent in the last layer.
- `subspace_polys.rs` documents $N_{i}$ as the product of normalized subspace polynomials and provides the arbitrary-point normalized recurrence.

The mathematical $V_{j}$ here are nested prefix spans of the ordered basis. Do not identify them blindly with every `DomainContext::subspace(i)` returned by the API: those spaces describe transformed domains used by different-sized NTT/FRI folds and need not be nested. Exact indexing/coset conventions will be made explicit in the next walkthrough.

The BitAnd lookup builder from Lesson014 uses separate generic domain contexts for the 64-point input and the 128-point output. Their compatible ordered basis prefixes let low-degree novel-basis coefficients be zero-padded and re-evaluated as the same polynomial. It uses NeighborsLastReference for table precomputation; this is distinct from running a reference transform on every witness word. Optimized NTT implementations exist but have not yet been inspected.

## Recall and next step

- What makes $W_{j}$ useful? It vanishes on a subspace and is constant on its cosets.
- Why normalize it? To make its values on sibling cosets differ by exactly one.
- Which coefficients enter the transform? Novel-basis coefficients, not arbitrary monomial coefficients.

Next: a full four-point forward/inverse butterfly computation, then map each index and twiddle to the actual reference code and explain its use in the $64\to 128$ LDE.

Sources: [NTT contract](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/math/src/ntt/mod.rs); [normalization/domain constants](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/math/src/ntt/domain_context.rs); [reference butterflies](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/math/src/ntt/reference.rs); [novel-basis evaluation](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/math/src/ntt/subspace_polys.rs). Background: [Lin–Chung–Han 2014](https://arxiv.org/abs/1404.3458); abstract and revision metadata reviewed, not the whole paper.

Validation: independently checked subspace products/recurrence and additivity over $\mathbb{F}_{16}$, novel-basis degrees, and forward/inverse reference-shaped transforms at dimensions1–4 against direct polynomial evaluation on all coefficient basis vectors plus representative inputs. Verified the $\mathbb{F}_{4}$ two-coset example. No Rust execution, optimized backend testing, or full API/coset convention validation.
