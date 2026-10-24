# Lesson 042 — Unequal oracles: zero-lift some axes, repeat along others

Introduced: 2026-10-23. Understanding unchecked. Binius64 revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** the combined opening gives each smaller message a precise placement. Some added coordinates select its zero-padded block; others leave it unchanged by repetition. These two operations have different evaluation factors.

Prerequisites: [relation sumcheck](039-linear-oracle-queues-and-relation-batching.md), [encoding](040-reed-solomon-encoding-and-bit-reversal.md) and [synchronized folds](041-basefold-butterflies-and-the-terminal-check.md).

## 1. Resolve each input into one common code dimension

Let oracle $i$ have $2^{n_i}$ raw message entries and let $N=\max_i n_i$. The parameter selector chooses a common reduced Reed–Solomon dimension $2^D$.

For an unmasked oracle, the interleave width exponent is

$$
b_i=\max(n_i-D,0),\qquad
d_i=n_i-b_i,\qquad
\ell_i=D-d_i.
$$

It encodes $2^{b_i}$ blocks of length $2^{d_i}$. If its message is already smaller than $2^D$, then $b_i=0$ and $\ell_i=D-n_i$ supplies the lift.

A masked oracle instead commits its message and an equal-length mask, fixes $b_i=1$, and has $d_i=n_i$. The selector requires $D\ge n_i$ for these oracles. Its single interleave coordinate uses the shared mask challenge $\gamma$.

[CodewordSpec resolution](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/fri/common.rs#L421) records lift, early batch size and later batch size. OracleSpec describes the raw message and ZK flag; CodewordSpec describes the resolved encoded layout.

## 2. Define the combined message exactly

Let $\pi_i'$ be the unmasked message, or the mask-folded message

$$
\pi_i'=(1-\gamma)\pi_i+\gamma\omega_i.
$$

Define $h_i=N-n_i-\ell_i$. Its combined-buffer contribution is

$$
P_i(X)=
\widetilde\pi_i'(X_{<n_i})
\prod_{j=n_i}^{n_i+\ell_i-1}(1-X_j).
\tag{1}
$$

It ignores the last $h_i$ coordinates. In memory, put the original table in the first $2^{n_i}$ positions of a block of length $2^{n_i+\ell_i}$, fill the rest with zeros, and repeat that block $2^{h_i}$ times.

For example, with $N=3$, $D=2$ and a smaller two-entry oracle $[a,b]$, its placed table is

$$
[a,b,0,0,a,b,0,0].
$$

Its MLE is $\bigl((1-X_0)a+X_0b\bigr)(1-X_1)$, independent of $X_2$. At a point $r$, it receives a factor $1-r_1$, but no factor $1-r_2$.

[place_repeated and combined-buffer construction](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop-prover/src/basefold/channel.rs#L365) implement this placement, including blocks narrower than one packed element.

## 3. Two phases intentionally use different padding

Phase A pads each **linear relation** to $N$ variables, so its endpoint contains

$$
\alpha_i\widetilde T_i(r_{<n_i})
\prod_{j=n_i}^{N-1}(1-r_j),
\qquad
\alpha_i=\widetilde\pi_i'(r_{<n_i}).
$$

Phase B authenticates a different combined **message polynomial**. After receiving all $\alpha_i$, sample fresh outer coordinates $\zeta\in K^{\lceil\log_2k\rceil}$ and set $e_i=\operatorname{eq}(\zeta,i)$. The combined message and target are

$$
P(X)=\sum_{i=0}^{k-1}e_iP_i(X),
\qquad
s'=\sum_{i=0}^{k-1}e_i\alpha_i
\prod_{j=n_i}^{n_i+\ell_i-1}(1-r_j).
\tag{2}
$$

Only lift coordinates appear in (2); repeat coordinates contribute one. For a non-power-of-two oracle count, unused outer-index slots represent zero.

The [verifier's actual s_prime body](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/basefold/channel.rs#L238) takes exactly log_lift coordinates. Earlier comments in the same function use an abbreviated full-padding product; the body and repeat placement are authoritative.

## 4. Why a lift becomes adjacent codeword copies

From Lesson040's bit reversal and novel-basis identities,

$$
\operatorname{Enc}_{d+\ell}
(\operatorname{ZeroPadMSB}_\ell v)[j]
=\operatorname{Enc}_d(v)[j\gg\ell].
\tag{3}
$$

High zero coordinates of the message become low zero coefficient bits after reversal. Iterating the Gao–Mateer map $Z\mapsto Z^2+Z$ drops the low domain-index bits, giving $2^\ell$ consecutive copies of each original encoded symbol.

Thus a verifier can query the virtual lifted codeword at index $j$ by requesting original leaf index $j\gg\ell_i$. It does not need a new commitment containing all those copies.

[BrakedownOracle::open_queries](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/fri/batch.rs#L89) shifts indices, receives each interleaved coset, and folds its lanes. This is a codeword lift, distinct from repeating the message across high coordinates in (1).

## 5. Route challenges by coordinate meaning

The first codeword fold consumes three consecutive groups:

$$
[\text{early mask challenge}]
\;\Vert\;[\text{outer oracle coordinates}]
\;\Vert\;[\text{later ordinary coordinates}].
$$

The early group is present only when a masked oracle exists. Outer coordinates combine oracle identities. The later group has

$$
b_{\max}=\max_{i\text{ unmasked}}b_i,\qquad N=D+b_{\max}
$$

coordinates, taken from the first ordinary MLE-check rounds binding the highest message coordinates.

An unmasked oracle uses the **last $b_i$** later challenges. Earlier later-group challenges bind its repeat dimensions, so they have no effect. Masked oracles use $\gamma$ and ignore all later-group challenges at this first fold.

After those ordinary coordinates bind, the combined field table has dimension $D$. Each original encoded coset is folded using its routed challenges, its result is duplicated by $2^{\ell_i}$, and the results are combined with $e_i$. This equals encoding that reduced combined field table.

[BatchBrakedownFolder::fold](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop-prover/src/fri/fold.rs#L481) fuses the outer scalar into the lane tensor and writes copies directly. [FRIQueryVerifier::new_batch](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/fri/verify.rs#L76) constructs the matching suffix windows.

## 6. Fresh outer batching needs no division by padding factors

Suppose a false Phase-A terminal value passes the check using received $\alpha_i$. Relative to true oracle evaluations, at least one discrepancy must survive its full Phase-A padding factor. That discrepancy also survives the shorter Phase-B lift factor, since the former is the latter times the repeat-coordinate factors.

The resulting nonzero vector of lift-weighted discrepancies has a nonzero outer-index MLE. Fresh $\zeta$ hides it with probability at most

$$
\frac{\lceil\log_2k\rceil}{|K|},
$$

conditional on the fixed messages and ideal fresh coins. This argument never divides by a padding factor and remains valid when some factors are zero. The subsequent combined MLE/FRI authentication has its own error.

The mask transformation's hiding and security properties require the later ZK analysis; matching its algebra does not establish zero knowledge.

## Validation and recall

[Independent report](../../work/lesson042-validation.json):136mixed $\mathbb F_{16}$ batches,24GHASH batches and4,516per-oracle virtual query values. Checks cover combined targets, first encoded folds, suffix routing, non-power-of-two oracle counts and terminal equality. No Rust or ZK proof.

**Takeaway:** zero-padding contributes an equality-to-zero factor; high-coordinate repetition contributes one.

**Recall:** why does a smaller oracle ignore some of the first ordinary folding challenges?
