# Lesson 047 — Sampling and grinding must preserve the exact state schedule

Introduced: 2026-10-23. Understanding unchecked. Binius64 revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** field challenges, query-index bits and proof-of-work checks consume the same challenger stream through different readers. Their exact widths and ordering affect every later challenge.

Prerequisites: [challenger state](045-transcript-tape-and-challenger-state.md), [query counts](044-fri-query-counts-arities-and-proof-size-accounting.md) and [exceptional interpolation](041-basefold-butterflies-and-the-terminal-check.md).

## 1. Full-field challenges accept every binary representation

[CanSample for the transcript](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/transcript/src/transcript.rs#L165) deserializes a field element directly from the challenger stream. For GHASH, the inspected underlier decoder reads16little-endian bytes. Every128-bit string represents one field element.

There is no prime-modulus reduction or rejection sampler needed for this binary field representation. Under ideal uniform bytes, the resulting field value is uniform:

$$
\Pr[\rho=a]=2^{-128}\qquad\text{for every }a\in K.
$$

In particular, zero and one are permitted. The native IP sample methods forward to this reader without excluding either value.

This resolves the sampler-policy part of the earlier $\alpha=1$ interpolation question: the inspected native path has no rejection/retry rule removing that coordinate. The optimized honest-prover shortcut still has its exceptional event; a pinned Rust reproduction remains pending. No soundness exploit follows from this observation.

## 2. Query bits use a32-bit reader

[sample_bits_reader](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/transcript/src/fiat_shamir/sampling.rs#L35) always consumes four bytes, interprets them little-endian and keeps the requested low bits, clamping the width to32.

For $0\le b\le32$:

$$
\operatorname{sample\_bits}(b)
=\operatorname{LE32}(\text{next four bytes})
\mathbin{\&}(2^b-1).
$$

The mask is a bit operation, not field arithmetic. Even requesting zero bits consumes four bytes at this helper level.

The native WordIP channel converts this result to a64-bit Word; that conversion does not restore extra random bits. A request above32still has at most32bits of support.

Therefore a full-domain-uniform FRI query analysis through this native reader requires index_bits at most32. The generic FRI shape code inspected here does not itself establish that bound for every possible rate/shape. Typical concrete configurations and all larger configurations need to be evaluated against it; no large-configuration attack or runtime failure was demonstrated.

## 3. What a grind asks the prover to find

[ProverTranscript::grind](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/transcript/src/transcript.rs#L354) searches sequential64-bit nonces. For each trial it clones the current challenger, observes the nonce's eight little-endian bytes and checks whether the next $b$ sampled bits are zero.

In an ideal hash model, one trial succeeds with probability $2^{-b}$, giving expected trial count

$$
\mathbb E[T]=2^b.
$$

Failed trials do not alter the real transcript. On success, the prover writes the nonce as an observed message and consumes the same four-byte bit sample on its real state.

VerifierTranscript::verify_grind reads the nonce as a message, samples the same bits and rejects unless they are zero. The helper caps difficulty at32bits.

## 4. Grinding changes search cost, not a conditional probability for free

Suppose a subsequent bad event has probability $\varepsilon$ under independent ideal challenge bits. Requiring both a valid grind and that event gives approximately

$$
\Pr[\text{one trial meets both}]
=2^{-b}\varepsilon.
$$

A prover searching for both therefore pays about $2^b/\varepsilon$ trials. Conditional on already passing the grind, the later honest challenge does not automatically become $b$ bits “more random.”

Security gains must be stated against a computation/random-oracle query budget, with the grind at the right fixed transcript prefix. The source comment about gaining bits is an engineering summary, not a stand-alone theorem for all protocol terms.

The inspected ordinary native Binius64/BaseFold path does not insert a grinding step. Having a grinding-capable Merkle channel does not activate it. Other protocol variants and their call sites require separate inspection.

## 5. Zero difficulty differs between the raw helper and channel contract

At the direct transcript level, grind(0) still chooses nonce zero, writes its eight bytes and consumes the four-byte sample.

The [GrindingProverChannel implementation](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop-prover/src/merkle_channel.rs#L204) instead returns immediately at zero difficulty; its verifier counterpart also does nothing. That capability's contract is: zero difficulty leaves the tape and challenger unchanged.

Mixing these two call paths at zero would desynchronize prover and verifier. The distinction is visible in the actual function bodies despite the shared method names.

## Validation and recall

[Independent model](../../work/verify_lessons045_048.py) checks40nonce round trips at difficulties one through ten, zero-difficulty raw/channel behavior and requested widths from zero through64. It reproduces accepted raw zero/one encodings and exact sample consumption. No billions-of-trials test, Rust proof or full grinding security reduction.

**Takeaway:** a sampling API's output type does not determine how many random bytes it consumes or which values it excludes.

**Recall:** why can a64-bit Word returned by this native query sampler contain only32random bits?
