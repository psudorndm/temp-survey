# Lesson 039 — A queued relation is an obligation, not yet an opening

Introduced: 2026-10-23. Understanding unchecked. Implementation: Binius64 at 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** each reduction can request a linear check against a committed table. The BaseFold channel collects those requests and authenticates them together at the end.

Prerequisites: [ring-switch relation](036-ring-switch-transpose-and-fresh-row-batching.md), [public/private endpoints](038-ring-switch-tables-and-public-private-closure.md), [lookup commitments](027-logup-rational-identity-and-fraction-gkr.md) and [batching](017-batching-and-rerandomization.md).

## 1. What an oracle relation means

For a committed table $\pi_i:\{0,1\}^{n_i}\to K$, a relation requests

$$
\langle\pi_i,t_{ij}\rangle
=\sum_x\pi_i[x]t_{ij}[x]=s_{ij}.
\tag{1}
$$

The prover has the transparent coefficient table $t_{ij}$. The verifier owns a formula evaluating its MLE at later query points. “Transparent” means this formula can be computed from known data and prior challenges; it does not mean its evaluation is necessarily cheap.

The [IOP channel trait](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/channel/mod.rs) separates receiving an oracle handle from requesting its relations. The handle identifies which committed table must satisfy the check.

[BaseFoldVerifierChannel::verify_oracle_relation](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/basefold/channel.rs#L438) stores the handle's transparent closure and target, then returns. It does not read a Merkle path or run BaseFold at that call. The later finish method performs the combined opening. Returning successfully from a reduction without eventually calling finish would leave these obligations unauthenticated.

## 2. The concrete pending relations

The trace oracle carries the ring-switch query

$$
\langle\pi_{\mathrm{private}},A_{\mathrm{rs}}\rangle=s_{\mathrm{rs}}.
$$

Current IntMul also uses a committed pushforward $Y$ for its single shared power table. [logup_star::verify_transparent](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/logup_star.rs#L174) queues both

$$
\langle Y,\operatorname{eq}(a,\cdot)\rangle
=\widetilde Y(a),
\qquad
\langle Y,T_{\mathrm{power}}\rangle=s_{\mathrm{product}}.
$$

The first binds the fractional-reduction endpoint to this $Y$; the second binds the original lookup product to the same $Y$. Replacing either by an unrelated prover value would sever that connection.

The [IntMul phase5 caller](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/intmul/verify.rs#L293) passes one transparent table. Thus, when this branch runs, native Binius64 has the private trace and an additional pushforward commitment. A generic “only one oracle” comment in the plain prover/Blueprint is not a complete description of this pinned IntMul path.

## 3. First combine relations belonging to the same oracle

Once the queue is fixed, sample one scalar $\lambda$ and define

$$
T_i=\sum_{j=0}^{k_i-1}\lambda^j t_{ij},
\qquad
S_i=\sum_{j=0}^{k_i-1}\lambda^j s_{ij}.
$$

Linearity gives $\langle\pi_i,T_i\rangle=S_i$ for honest relations. In $\mathbb F_4$, take $\pi=[1,\alpha]$, coefficient tables $[1,0]$ and $[0,1]$, and $\lambda=\alpha$. Their combined table is $[1,\alpha]$ and their combined target is

$$
1+\alpha\cdot\alpha=\alpha.
$$

Both are computed from the same powers of $\lambda$.

For fixed discrepancies $\delta_{ij}=\langle\pi_i,t_{ij}\rangle-s_{ij}$, a false family can cancel only when the nonzero polynomial $\sum_j\delta_{ij}\lambda^j$ vanishes. A conservative union bound across oracles is

$$
\Pr[\text{a false family cancels}]
\le\frac{\sum_i(k_i-1)}{|K|}.
$$

The original targets and defining transcript data must already be bound before $\lambda$. This is an explicit prover-channel contract; queueing a scalar does not itself serialize it.

[The verifier batching body](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/basefold/channel.rs#L277) samples $\lambda$ even if every oracle has only one relation. The prover materializes the first transparent as its accumulator and adds later structured blocks without writing their zero padding.

## 4. Then reduce different oracles to evaluations at one point

Let $N=\max_i n_i$. In the non-ZK case, a second fresh scalar $\theta$ batches the per-oracle product sums. The polynomial actually represented by the padded prover is

$$
H(X)=\sum_i\theta^i
\widetilde\pi_i(X_{<n_i})\widetilde T_i(X_{<n_i})
\prod_{j=n_i}^{N-1}(1-X_j).
\tag{2}
$$

This has degree at most two in every coordinate. Each padding coordinate appears **once**. One equivalent factorization zero-pads the message table and repeats the transparent table along the extra coordinates.

Zero-padding both factors would give a squared padding factor off the Boolean cube. It agrees on Boolean points but defines a different sumcheck polynomial. For example at $r=\alpha$ in $\mathbb F_4$,

$$
D=1+\alpha,\qquad D^2=\alpha\ne D.
$$

The code uses [PaddedSumcheckDecorator](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip-prover/src/sumcheck/padded.rs), which pads the whole product relation. Blueprint §5.4's description in terms of two zero-padded operands should not replace this implemented off-cube convention.

After degree-two sumcheck, the prover sends one $\alpha_i=\widetilde\pi_i(r_{<n_i})$ per oracle. The verifier computes

$$
H(r)\stackrel?=
\sum_i\theta^i\alpha_i\widetilde T_i(r_{<n_i})
\prod_{j=n_i}^{N-1}(1-r_j).
\tag{3}
$$

The challenge vector returned by sumcheck is reversed into low-to-high coordinate order before these evaluations.

The scalar $\theta$ is distinct from $\lambda$. A further vector of outer oracle-batching challenges is sampled only after the $\alpha_i$ messages; its lift/repeat geometry is the next unequal-size lesson.

## 5. What this step still does not prove

The $\alpha_i$ values in (3) are still prover claims. BaseFold's combined MLE-check and FRI must authenticate them against the previously committed codewords.

Optional ZK oracles add a preceding mask-claim step and shared challenge $\gamma$. The unified channel branches on each OracleSpec::is_zk; its older “always applies blinding” comment is too broad. This lesson derives the unmasked branch. It does not claim the mask branch alone proves zero knowledge.

When any relation is queued, current combined-opening code requires at least one relation for every committed oracle. If every queue is empty, finish skips opening. Commitment count/shape checks and buffer-return invariants are separate code checks.

## Validation and recall

[Independent report](../../work/lessons039-041-validation.json): 2,048 honest relation batches, all336nonzero $\mathbb F_4$ discrepancy vectors through four relations, and120single-padding-factor fixtures. These are scalar models; no Rust or Merkle execution.

**Takeaway:** separate requesting a check, batching requests, and authenticating the resulting evaluations.

**Recall:** which two relations prevent a lookup pushforward from being replaced by unrelated values?
