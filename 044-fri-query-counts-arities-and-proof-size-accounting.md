# Lesson 044 — Queries, rate and proof bytes pay for different things

Introduced: 2026-10-23. Understanding unchecked. Binius64 revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** lower code rate improves the query test but expands the codeword. Folding arities trade path hashes against opened field values. Concrete parameters balance those costs.

Prerequisites: [code distance](040-reed-solomon-encoding-and-bit-reversal.md), [terminal tests](041-basefold-butterflies-and-the-terminal-check.md), [unequal sizes](042-unequal-oracles-lift-repeat-and-first-fold-routing.md) and [Merkle batches](043-merkle-leaves-domain-separation-and-shared-layers.md).

## 1. The query target is not the whole security claim

[Verifier configuration](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/verify.rs#L37) declares SECURITY_BITS equal to96. Its setup computes the FRI query count from this target and the selected inverse rate.

For code rate $\rho=2^{-r}$, [calculate_n_test_queries](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/fri/common.rs#L506) uses

$$
p_{\mathrm{query}}=\frac{1+\rho}{2},
\qquad
q=\left\lceil
\frac{t}{-\log_2p_{\mathrm{query}}}
\right\rceil.
\tag{1}
$$

This makes $p_{\mathrm{query}}^q\le2^{-t}$. It uses the code's unique-decoding-radius regime in the cited FRI analysis. A complete derivation of that adaptive proximity theorem has not been checked in this notebook.

At fixed rate this formula grows linearly with the requested security bits $t$. Blueprint §5.2's parameter prose calls the query count logarithmic in its security parameter; that wording does not describe this implementation's formula.

For a fixed bad set occupying fraction $\delta$ of positions and independent uniform queries with replacement, the elementary miss probability is $(1-\delta)^q$. Connecting an adversarial folding transcript to the required bad-set fraction is the deeper FRI argument; code distance alone does not prove it.

The query term excludes folding-round algebraic errors, other reductions, commitment/hash assumptions and Fiat–Shamir losses. A128-bit field and a96-bit query target describe different quantities.

## 2. The actual counts

For $t=96$:

| Code rate | Queries |
|---|---:|
| $1/2$ |232|
| $1/4$ |142|
| $1/8$ |116|
| $1/16$ |106|
| $1/32$ |101|
| $1/64$ |99|

The improvement flattens as $\rho$ approaches zero: equation (1) approaches96queries. Meanwhile each additional inverse-rate bit doubles the encoded symbol count for a fixed message.

Holding the interleave decomposition fixed, an oracle of own dimension $2^d$ and batch width $2^b$ executes

$$
d\,2^{d+b+r-1}
$$

logical NTT butterflies. The first $r$ zero-padding layers are skipped. This is a derived operation count, not a timing measurement; retuning the batch dimension can also change $d$.

## 3. Grouping a folds changes leaf and path sizes

A grouped fold of width $a$ consumes $2^a$ symbols per queried coset. If its input codeword has $2^L$ symbols, its Merkle tree has depth $L-a$.

Let $v$ be bytes per field value and $H$ bytes per digest. For $q$ queries, a layer reduction costs

$$
C(L,a)=q\,v\,2^a
+H\left(2^h+q(L-a-h)\right),
\quad
h=\min(L-a,\lceil\log_2q\rceil).
\tag{2}
$$

Larger $a$ reduces later codewords and shortens paths, but the opened coset payload grows exponentially. Counting tree depth as $L$ would charge $a$ extra hashes per branch and optimize the wrong layout.

For GHASH and the standard suite, $v=16$ and $H=32$.

## 4. Read the selector as a cost-model search

[ReductionOptimizer](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/fri/common.rs#L534) builds a dynamic-programming table. At message exponent $i$, it compares sending the whole terminal vector, costing $v2^{i+r}$, with a reduction of width $a$ plus the recorded cost at $i-a$.

It also chooses the common reduced dimension $D$, accounting for each input oracle's initial batch/lift layout. Masked inputs fix their batch exponent to one; unmasked inputs have flexible batch sizes.

The helper min_concave scans candidate costs until the first increase and keeps the later candidate on ties. Its stated assumption is unimodality along that scan. We have not proved this for every possible parameter shape.

The constructor [BaseFoldVerifierCompiler::new](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/basefold/compiler.rs#L43) accepts an arity-strategy argument but does not consult it on this combined path; it calls optimal_for_batch instead. Reading only the top-level setup's ConstantArityStrategy calculation would therefore give the wrong impression of which arities are actually selected.

The selection estimate excludes root digests. The exact size adds one digest for each input root and each subsequent committed codeword; that count depends on the selected fold sequence. Our168single-oracle comparisons found the source-selected plans also minimized the full cost, but this finite check is not a general optimality proof.

## 5. Account for the FRI component exactly

[fri::proof_size](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/fri/size_estimation.rs#L20) counts:

- One root per input oracle.
- One root for the first combined codeword and one after each subsequent committed reduction.
- Each input oracle's coset values and Merkle batch proof, separately.
- The coset values and Merkle proof for each later reduction.
- The entire terminal vector.

For $k$ input oracles and $m$ later fold widths, root bytes total $H(k+m+1)$. Outer oracle-batching coordinates add no separate decommitment payload: they combine values already opened.

This function counts the **FRI component**, including initial roots. The BaseFold relation-reduction messages, degree-one MLE coefficients and original Binius reductions contribute additional proof bytes.

## 6. A concrete rate trade-off

The independent model reproduces these pinned FRI byte counts for one unmasked oracle, with arities and batch widths reselected at each rate:

| Rate | Message length $2^{17}$ | Message length $2^{24}$ |
|---|---:|---:|
| $1/2$ |205,152|472,480|
| $1/4$ |160,128|340,128|
| $1/8$ |154,240|307,264|
| $1/16$ |161,088|307,008|
| $1/32$ |172,544|318,240|
| $1/64$ |186,304|336,192|

Among these candidates, the smaller message favors rate $1/8$, while the larger favors $1/16$. Lower rate does not make proof size fall indefinitely.

These are calculated sizes using current source conventions, not measured proof files or proving-time benchmarks. The pinned test's label zk/24+21 denotes one masked length-$2^{24}$ oracle and one **unmasked** length-$2^{21}$ oracle.

## Validation and recall

[Independent report](../../work/lesson044-validation.json) matches all18pinned size fixtures, the six query counts and168full-cost comparisons. It records chosen dimensions/arities and the FRI-only scope. No Rust proof generation, full soundness theorem, optimizer-wide proof or performance measurement.

**Takeaway:** field size controls algebraic error, query count controls a proximity-test term, and rate/arity determine concrete work and bytes.

**Recall:** why can halving the code rate make the proof larger even though fewer queries are needed?
