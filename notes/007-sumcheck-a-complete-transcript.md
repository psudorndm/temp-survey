# Lesson 007 — Sumcheck, from a sum to one evaluation

2026-10-23 · Sumcheck, part 1 · Introduced; understanding unchecked.

## The claim

Let F(X,Y) = 1 + X + XY over a binary field K. Its Boolean values are [1,0,1,1], indexed by X + 2Y. The claimed field sum is

\[
S=\sum_{x,y\in\{0,1\}}F(x,y)=1+0+1+1=1.
\]

This is a field sum (XOR for bits), not a count of ones. F has individual degree at most one. We bind Y first, matching the highest-index-first convention in the Blueprint.

## Round 1: remove Y from the sum

The honest prover sends the univariate polynomial

\[
R_1(T)=\sum_{x\in\{0,1\}}F(x,T)=1+T.
\]

The verifier checks R₁(0) + R₁(1) = 1 + 0 = S and that the degree bound is respected. Only then does it draw a fresh uniform challenge s in K.

The new claim is

\[
\sum_{x\in\{0,1\}}F(x,s)=R_1(s)=1+s.
\]

There is now just one variable left to sum over. The prover's message is supposed to be the partial sum polynomial, but checking two endpoints alone cannot establish that; random challenges and the final evaluation supply the soundness argument.

## Round 2: remove X

After seeing s, the honest prover sends

\[
R_2(T)=F(T,s)=1+(1+s)T.
\]

The verifier checks R₂(0) + R₂(1) = 1 + s = R₁(s), then samples a fresh uniform r in K. The remaining claim is

\[
\boxed{F(r,s)=R_2(r)=1+r+rs.}
\]

All challenges must come after the relevant messages. The second message may depend on s, but it must be fixed before r.

For a fully numerical toy transcript, suppose both random challenges happen to equal α in F₄. The first updated claim is 1 + α; the second polynomial is 1 + (1 + α)T; its value at α is zero. Directly F(α,α) = 1 + α + α² = 0, so the terminal check accepts. Choosing α here is for illustration; a real verifier samples challenges.

## What the final step requires

In this toy example, the verifier knows F's explicit formula and can compute F(r,s) directly. In Binius applications, F is often a composition of witness-dependent multilinears. If F = C(f̃₀,…,f̃ₘ₋₁), the verifier needs authenticated values f̃ᵢ(r,s) and checks their composition against the final claim. More reductions and commitment openings supply that authentication.

Sumcheck alone outputs a reduced claim, not a complete proof. It does not authenticate the underlying polynomial or establish that every constraint vanishes.

## Why random challenges preserve soundness

Suppose the current claim is false. Any degree-at-most-d round polynomial passing its endpoint-sum check must differ from the true partial-sum polynomial: otherwise its endpoint sum would equal the true claim. Their difference is a nonzero polynomial of degree at most d, fixed before the next random challenge. It has at most d roots.

Except with probability at most d/|K|, a false claim therefore stays false after a round. If it stays false throughout, an exact terminal evaluation rejects. A union bound over n rounds gives at most min(1,nd/|K|). This assumes the stated degree bounds, conditional fresh uniform challenges, a fixed underlying polynomial, and a valid terminal check. Commitment and Fiat–Shamir security require additional analysis.

A concrete failure mode if the terminal check is skipped: falsely claim S=0, send R₁(T)=1, then R₂(T)=1+T. Both endpoint equations hold for every first challenge. The final claimed value 1+r agrees with the actual F(r,s) only when rs=0. Over F₄ this occurs for 7 of the 16 challenge pairs. The tiny field is intentionally insecure; over a field of size q, this strategy succeeds with probability 2/q − 1/q², within the 2/q bound. Without the terminal check, every one of these internally consistent transcripts would pass.

## The implementation detail: one coefficient is omitted

At pinned revision `4428e759dd27956bd5b9ae5025d67eb8ccc1accf`, `RoundProof` in `crates/ip/src/sumcheck/common.rs` omits the highest coefficient of a degree-bound-d round polynomial, for positive d. If

\[
R(T)=a_0+a_1T+\cdots+a_dT^d,
\]

and the current claim is c, then the endpoint equation determines

\[
a_d=c-a_0-\sum_{j=0}^{d-1}a_j.
\]

In characteristic two, the two copies of a₀ cancel. For degree bound one, the slope is simply c; only the constant is transmitted. For the honest example, both rounds transmit constant 1, and the verifier recovers slopes 1 and 1+s respectively.

`verify.rs` reads exactly d field coefficients, samples a challenge, reconstructs the omitted coefficient using the already-fixed current claim, evaluates, and pushes the challenge into its output. Reconstruction enforces endpoint consistency by construction; the absence of a separate endpoint rejection branch is intentional. Polynomial length encodes the degree bound. The actual challenge channel and message authentication still depend on other code.

Challenges are pushed in sampling order: s for the high variable, then r for the low variable. The mathematical input point is (r,s). Do not confuse transcript order with coordinate order; each caller's ordering convention must be checked when tracing integration.

This example uses a multilinear F for clarity. In characteristic two, summing a pure multilinear over the full Boolean cube isolates its highest multilinear monomial coefficient. Actual sumchecks become more useful on compositions/products, whose individual degrees may exceed one. Computing honest prover messages can still require work linear in witness size; a short interaction is not a claim of constant proving cost.

## Recall and next step

- Why must R₁ be sent before s? A discrepancy chosen after s could be made to vanish there.
- What remains after n rounds? An evaluation claim on the underlying polynomial at the n challenges.
- Why is sumcheck not a zerocheck? A zero sum can contain cancelling nonzero constraint errors.

Next: turn “every constraint is zero” into a random equality-weighted sum claim. Derive why an unweighted zero sum is insufficient and introduce zerocheck/MLE-check.

Sources: [Blueprint §2.2](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex); [round coefficient recovery](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip/src/sumcheck/common.rs); [sumcheck verifier](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip/src/sumcheck/verify.rs).

Validation: checked the honest transcript at all 16 F₄ challenge pairs, the 7/16 dishonest example with a terminal check, and compressed coefficient recovery for all F₄ polynomials at degree bounds 1–3. Independent mathematical verification; no Binius Rust execution or end-to-end proof test.
