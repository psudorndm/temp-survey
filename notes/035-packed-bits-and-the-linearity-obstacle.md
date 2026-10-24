# Lesson 035 — Packing bits does not preserve every evaluation

Introduced: 2026-10-23. Understanding unchecked. Implementation: Binius64 at 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** storing bits inside a field element is lossless, but extracting those bits does not commute with multiplication by an arbitrary challenge-field scalar. Ring-switching bridges this algebraic mismatch.

Prerequisites: [field arithmetic](001-binary-field-arithmetic.md), [packing](004-binary-towers-and-packing.md), [MLEs](005-multilinear-extensions.md) and [the private shift endpoint](034-segmented-shift-product-and-exceptional-extraction.md).

## 1. Fix the two index axes

Let $K=\mathbb F_{2^d}$, with $d=2^\kappa$, and choose an $\mathbb F_2$-basis $\beta_0,\ldots,\beta_{d-1}$. Write the witness as bits $w(i,y)$, where $i$ indexes a bit inside a packed element and $y\in\{0,1\}^m$ indexes that element. Store

$$
\pi[y]=\sum_{i=0}^{d-1}\beta_iw(i,y).
$$

This is a bijection between $d$ bits and one field element. A binding commitment to this packed table therefore binds its underlying bits too; packing does not require a second Booleanity proof for the field element's binary coordinates.

The bit-polynomial claim is different from an ordinary evaluation of the packed polynomial:

$$
t=\widetilde w(r_{\mathrm{lo}},z)
=\sum_y\operatorname{eq}(z,y)
  \sum_i\operatorname{eq}(r_{\mathrm{lo}},i)w(i,y).
\tag{1}
$$

Here $r_{\mathrm{lo}}\in K^\kappa$ and $z\in K^m$. The basis weights $\beta_i$ used for storage are fixed; the equality weights in (1) depend on the query.

## 2. The smallest counterexample

Use $K=\mathbb F_4$ with $\alpha^2=\alpha+1$ and basis $(1,\alpha)$. Let $c_0(a_0+a_1\alpha)=a_0$. Coordinate extraction preserves XOR, but

$$
c_0(\alpha\cdot1)=0
\ne \alpha\,c_0(1)=\alpha.
$$

Thus $c_0$ is $\mathbb F_2$-linear, not $K$-linear.

For one packed cell, evaluation of its two bits at $r$ is

$$
F_r(a_0+a_1\alpha)=(1-r)a_0+ra_1.
$$

At $r=\alpha$, this gives $F_r(1)=\alpha+1$ and $F_r(\alpha)=\alpha$. If this were multiplication by one field scalar, that scalar would have to be $\alpha+1$, but

$$
(\alpha+1)\alpha=1\ne F_r(\alpha).
$$

Consequently, even the one-cell query cannot generally be replaced by “multiply the packed cell by a suitable scalar.” Nor can the verifier extract the bits of $\widetilde\pi(z)$ and pretend those are the individual bit-polynomial evaluations at $z$: field multiplication by the interpolation weights mixes coordinates.

## 3. What current Binius64 actually packs

The default [field configuration](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/config.rs#L12) sets B128 to GHASH. Thus $d=128$, $\kappa=7$, and the basis is the polynomial basis used by that field representation.

[pack_witness](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/prove.rs#L434) packs consecutive pairs of unsigned words:

$$
\pi[y]=\operatorname{bits}_{128}
  \bigl(\mathrm{word}_{2y}\;|\;(\mathrm{word}_{2y+1}\ll64)\bigr).
$$

The right side denotes bit placement into the field representation, not integer multiplication or field multiplication. A missing final word and later padding positions are zero. The function's packed-lane tail handling keeps real pairs consecutive; it does not insert gaps between the aligned portion and a final odd word.

The shift output point is $r_J\Vert r_Y$: six coordinates for a bit within a word, followed by word-index coordinates. Ring-switching takes its **first seven** coordinates as $r_{\mathrm{lo}}$. This includes the lowest word-index coordinate, since one field element contains two words. For a segment spanning $2^L$ words, the remaining packed-cell point has $L-1$ coordinates. The public helper spans at least one whole packed element.

The public/private segment selector from Lesson034 has already been removed on the private path. It is not an eighth packing coordinate.

## 4. The obligation left for the next reduction

We need a check that starts from (1), preserves the packed storage advantage, and ends with a relation linear in the committed $\pi[y]$ values. [Lesson036](036-ring-switch-transpose-and-fresh-row-batching.md) constructs that relation using a binary matrix transpose and fresh row batching.

This is current Binius64's interface. Historical tower-field commitments use related ideas with different field and protocol choices; a GHASH representation is not automatically a recursive tower implementation.

## Checks and recall

The independent model in [verify_lessons035_037.py](../../work/verify_lessons035_037.py) checks the $\mathbb F_4$ counterexamples, four GHASH-sized identities and 76 word-pair/tail layouts. It does not execute the Rust implementation.

**Takeaway:** packed storage is a basis encoding; moving a random evaluation through that encoding needs a protocol.

**Recall:** why is XOR linearity insufficient to pull a challenge-field interpolation weight through bit extraction?
