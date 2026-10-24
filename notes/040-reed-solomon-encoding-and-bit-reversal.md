# Lesson 040 — The committed codeword is an encoding, not the witness table

Introduced: 2026-10-23. Understanding unchecked. Implementation: Binius64 at 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** the message entries are treated as bit-reversed coefficients of a univariate polynomial. Its evaluations form a redundant codeword, arranged so later folds match multilinear witness folds.

Prerequisites: [additive NTT](015-additive-ntt-subspace-polynomials.md), [reference transform conventions](016-four-point-ntt-and-lde.md), [MLEs](005-multilinear-extensions.md) and [linear-oracle interface](039-linear-oracle-queues-and-relation-batching.md).

## 1. Two polynomials with different jobs

Let the packed message $\pi$ have length $M=2^n$. Its multilinear polynomial is

$$
\widetilde\pi(X_0,\ldots,X_{n-1})
=\sum_{u\in\{0,1\}^n}\pi[u]\operatorname{eq}(X,u).
$$

For coding, choose inverse rate $R=2^r$ and an additive evaluation domain $E_{n+r}$ of size $RM$. Let $N_j(Z)$ denote the normalized novel basis from Lesson015, and let $\operatorname{rev}_n(j)$ reverse the $n$ index bits. Define

$$
f_\pi(Z)=\sum_{j=0}^{M-1}
\pi[\operatorname{rev}_n(j)]N_j(Z).
\tag{1}
$$

Then

$$
\operatorname{Enc}_n(\pi)[e]=f_\pi(z_e),
\qquad z_e\in E_{n+r}.
$$

The same message values are Boolean evaluations of $\widetilde\pi$ but novel-basis coefficients of $f_\pi$. These are not interchangeable polynomial representations.

The encoding is generally not systematic: its first $M$ symbols need not equal $\pi$. For $\pi=[1,0]$, equation (1) is the constant univariate polynomial one, so every code symbol is one, even though the second message entry is zero.

## 2. Redundancy gives distance

Because $\deg N_j=j$, every nonzero difference polynomial has degree at most $M-1$. It can vanish at at most $M-1$ distinct domain points. Thus two different messages produce codewords differing in at least

$$
RM-M+1
$$

positions. The relative distance is at least $1-1/R+1/(RM)$.

This is a deterministic distance statement for genuine codewords. It does not say a few samples alone certify an arbitrary vector, or give the full adaptive FRI soundness bound. The rate here is $M/(RM)=1/R$; it is unrelated to the binary extension degree used to pack witness bits.

## 3. Which domain the implementation fixes

[ReedSolomonCode::new/subspace](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/math/src/reed_solomon.rs#L36) derives its domain from the Gao–Mateer basis. Prover and verifier rebuild it from dimensions; the prover does not choose an arbitrary basis in a proof message.

This basis $(b_0,b_1,\ldots)$ satisfies

$$
b_0=1,\qquad b_{j+1}^2+b_{j+1}=b_j.
$$

[gao_mateer_basis](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/math/src/ntt/domain_context.rs#L170) starts from a fixed trace-one field element and repeatedly applies $T(x)=x^2+x$. For GHASH that starting element is the polynomial-basis bit $2^{121}$, read in its field declaration.

The coding-domain basis $b_j$ is distinct from the polynomial basis $\beta_i=Z^i$ used for packing. “Both are binary bases” does not make their coordinates equal.

For a miniature implementation-shaped example over $\mathbb F_{16}$ modulo $U^4+U+1$, choosing trace-one element $U^3$ gives raw basis bytes $(1,7,4,8)$. This is a toy basis choice, not the GHASH basis.

## 4. Why the Gao–Mateer chain helps folding

For the span of $b_0,\ldots,b_{j-1}$, its monic vanishing polynomial is

$$
W_j=T^{\circ j},\qquad W_j(b_j)=1.
$$

Indeed $T^{\circ j}$ has degree $2^j$ and vanishes on that $2^j$-element span. The chain elements are independent: applying an appropriate iterate of $T$ to a putative highest-index dependence exposes its nonzero top coefficient.

Consequently the normalized subspace polynomial is already $W_j$, and

$$
W_j(T(Z))=W_{j+1}(Z).
$$

The novel basis therefore satisfies

$$
N_{2j}(Z)=N_j(T(Z)),\qquad
N_{2j+1}(Z)=Z\,N_j(T(Z)).
\tag{2}
$$

These identities make an inverse butterfly expose two smaller code polynomials. Bit reversal makes their coefficient vectors the low and high halves of the original message. Lesson041 uses exactly this fact.

## 5. Read encode_batch as three concrete operations

[ReedSolomonCode::encode_batch](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/math/src/reed_solomon.rs#L117) checks the message/domain shapes, then:

1. Reverses all logical message-index bits, including any interleave dimensions.
2. Repeats that permuted buffer $R$ times.
3. Runs the additive NTT while skipping its first $r$ layers and its final interleave layers.

Why repeat? Had the code instead appended zero coefficients, each of the first $r$ butterflies would pair a live value with zero:

$$
(c,0)\longmapsto(c,c).
$$

Repeating the initial buffer performs exactly those skipped layers. This is a coefficient-domain NTT optimization, not repetition of the final Reed–Solomon codeword.

The bit-reversal operation permutes entries or packed lanes. It does not reverse the bits inside a field element.

## 6. Interleaving changes memory layout, not field arithmetic

With $B=2^b$ message blocks, the output contains $B$ independent codewords interleaved at each domain position. If the original block number is $\eta$, output lane $j$ corresponds to $\eta=\operatorname{rev}_b(j)$:

$$
\mathrm{out}[eB+j]
=\operatorname{Enc}_n(\pi_{\operatorname{rev}_b(j)})[e].
$$

For two blocks this is the familiar alternating sequence of their symbols. With four blocks, the lane block order is $0,2,1,3$. This matters when applying high-to-low multilinear challenges to the batch lanes.

[encode_interleaved](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop-prover/src/fri/encode.rs#L35) encodes each oracle at its own resolved dimension and common rate. The corresponding Merkle leaf holds one interleaved coset. Zero-knowledge oracles additionally interleave a message and random mask; their privacy analysis is deferred.

## Validation and recall

[Independent checks](../../work/lessons039-041-validation.json) compare80skipped-layer encodings against direct novel-polynomial evaluation, check21interleave/lane fixtures and42zero-pad/duplicate identities. The complete tiny rate-half $\mathbb F_4$ code has minimum distance three, matching the bound. Packed permutation kernels and Rust were not executed.

**Takeaway:** message MLE evaluation and codeword polynomial evaluation are linked by a carefully chosen encoding, not by being the same polynomial.

**Recall:** why does repeating the input before a skipped-layer NTT differ from repeating a finished codeword?
