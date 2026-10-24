# Lesson 002 — Tiny data, large random checks

2026-10-23 · Binary fields, part 2 · Introduced; user understanding not yet checked.

## Recall from Lesson 001

In our toy field F₄, α² = α + 1, so `10 × 10 = 11`. Field multiplication and integer multiplication differ.

## The problem with checking only bits

Suppose we need to distinguish two formal polynomials: A(T) = T² and B(T) = T. Their difference in characteristic two is P(T) = T² + T. P is a nonzero polynomial of degree two: it has nonzero coefficients, even though it evaluates to zero at both elements of F₂.

| Challenge r | P(r) |
|---|---|
| 0 | 0 |
| 1 | 0 |
| α | α² + α = 1 |
| α + 1 | (α + 1)² + (α + 1) = 1 |

Thus checking equality at a random bit never separates A from B. Checking at a uniformly random element of F₄ separates them half the time. Neither is an adequate cryptographic check; this is a small teaching example.

## The degree/root principle

A nonzero univariate polynomial of degree d over a field has at most d distinct roots. Therefore, for a polynomial P fixed independently of a uniform challenge r in K,

\[
\Pr_{r\gets K}[P(r)=0]\leq\min(1,d/|K|).
\]

Why: each distinct root a contributes a factor T − a. More than d roots would require more than d linear factors, exceeding the degree. The field property matters: the factorization and degree arguments rely on having no zero divisors.

This is the basic univariate ingredient behind polynomial identity testing. Later, multivariate tests and sumcheck require their own degree accounting and conditioning on earlier protocol messages.

For our P(T) = T(T + 1), the only roots in any characteristic-two field are exactly 0 and 1. Over K = F₂¹²⁸, the probability for this particular fixed-polynomial test is exactly 2/2¹²⁸ = 2⁻¹²⁷. That number is not Binius64's total soundness error or a claim of 127-bit end-to-end security.

## Two conditions that make the argument useful

1. **Bound the degree.** A prover must not be free to use arbitrary-degree polynomials. For example, T^q − T vanishes on every element of a field of size q, despite being a nonzero formal polynomial.
2. **Fix the relevant message before the challenge.** If the prover sees r first, it can choose P(T) = T − r and force the test to pass. In an interactive protocol, send the round polynomial first, then draw fresh randomness. How commitments and Fiat–Shamir enforce the required ordering will be studied separately.

Important: P vanishing on Boolean inputs is not itself a fraudulent computation. Many constraints intentionally vanish on the Boolean cube. The example illustrates failure of a test for *formal polynomial identity* over too small a sampling domain. A full proof protocol must establish the connection from a false statement to a nonzero polynomial discrepancy of bounded degree.

## Binius connection

Blueprint §2.1 separates bit-valued witness data from the challenge field K = F₂¹²⁸. Small witness entries support compact storage and small-field work; large random challenges make bounded-degree discrepancies hard to hide. This does not mean that every intermediate remains a bit: evaluating and folding with extension-field challenges produces extension-field elements.

Blueprint §2.2 states the usual n·d/|K| sumcheck error bound for n variables and maximum individual degree d. It is a reduction: its final evaluation claim must still be authenticated. The whole argument also has other soundness and cryptographic terms.

At pinned code revision `4428e759dd27956bd5b9ae5025d67eb8ccc1accf`, `crates/ip/src/sumcheck/verify.rs` receives round coefficients through `channel.recv_many(degree)` before `channel.sample()`. It evaluates the recovered round polynomial at that challenge and eventually returns an evaluation claim and challenges. It does not by itself discharge the final polynomial evaluation. The channel implementation and transcript security remain to be traced; the observed call order is not a standalone security proof.

## Recall

- Why does checking T² = T only at bits fail to establish a polynomial identity? Both 0 and 1 are roots of their nonzero difference.
- What controls the random-check error? The discrepancy's degree and the challenge-domain size, under the necessary independence and protocol conditions.
- Why sample after the prover message? To prevent the prover from choosing its discrepancy around the sampled point.

Next: subfields and embeddings. How can computations over different binary fields be made compatible, and why is copying raw bit patterns generally insufficient?

Sources: [pinned Blueprint source, §§2.1–2.2](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex); [sumcheck verifier](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip/src/sumcheck/verify.rs). Root-bound proof and F₄ example are elementary derivations supplied here. All four toy evaluations were independently checked. No Binius Rust tests were run.
