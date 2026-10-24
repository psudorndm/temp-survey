# Lesson 045 — The proof tape and challenge state are different objects

Introduced: 2026-10-23. Understanding unchecked. Binius64 revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** some bytes are stored, some are hashed, and some do both. Replaying the same sequence of these operations makes prover and verifier derive the same challenges.

Prerequisites: [BaseFold message order](041-basefold-butterflies-and-the-terminal-check.md) and [Merkle authentication](043-merkle-leaves-domain-separation-and-shared-layers.md).

## 1. One physical tape, three logical operations

[ProverTranscript and VerifierTranscript](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/transcript/src/transcript.rs) wrap one byte buffer and one challenger. Their accessors have these effects:

| Accessor | Stored in/read from proof tape | Feeds challenge state |
|---|---|---|
| observe |No|Yes|
| message |Yes|Yes|
| decommitment |Yes|No|

Public input/output words use observe: the verifier already has them. Algebraic round messages and Merkle roots use message. Merkle paths, opened leaf values and terminal vectors use decommitment, because they are checked against earlier observed roots.

These are not three separate physical proof files. Message and decommitment bytes can occupy successive positions in the same tape. The verifier's protocol determines which accessor consumes each region.

The [FiatShamirBuf body](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/transcript/src/fiat_shamir/buffer.rs) observes exactly the bytes advanced through a message read/write. A decommitment read advances the underlying tape directly.

## 2. Why not hash opening advice again?

Consider the sequence: observe public data, send root, sample query indices, then send leaf/path advice. The query must depend on the root before the prover sees it. The advice is constrained by that root and the selected index.

Changing an advice byte does not change the challenger's state. It should instead make authentication fail. Lesson043 traces that check. Sending unchecked witness-dependent values as decommitment advice would not be justified merely by choosing that accessor.

This separation also lets FRI sample its query indices up front and batch openings without making later challenges depend on the serialization of authentication paths.

## 3. The default challenger is a state machine

[HasherChallenger](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/transcript/src/fiat_shamir/hasher_challenger.rs#L32) alternates between observing bytes and sampling a digest stream.

Let $H$ be SHA-256 and $d_0=H(\varepsilon)$. Initially the sampling buffer contains $d_0$, its cursor is zero, and the running hasher is seeded with $d_0$.

When sampling switches to observing, it appends the current sampling cursor as an eight-byte little-endian integer. If the cursor is $\ell$ and the next observed byte run is $M$, the next digest is

$$
d'=H(d_{\mathrm{current}}\Vert\operatorname{LE64}(\ell)\Vert M).
\tag{1}
$$

After this digest is exposed for sampling, the hasher is reseeded with it. If more than32bytes are requested without another observation, subsequent blocks are

$$
d_{\mathrm{next}}=H(d_{\mathrm{current}}).
$$

The cursor in (1) is the position within the current digest buffer, not the lifetime count of all sampled bytes. Digest feed-forward carries the preceding history.

## 4. A byte-exact tiny transcript

Starting from default state, sampling eight bytes returns the first eight bytes of $d_0$. Sampling another24returns its remainder. If the parties then observe bytes $48_{\mathrm{hex}},55_{\mathrm{hex}}$, the next sampling block is

$$
H\bigl(d_0\Vert\operatorname{LE64}(32)\Vert
\operatorname{bytes}(48_{\mathrm{hex}},55_{\mathrm{hex}})\bigr).
$$

This is an inspected source test's transition, reproduced independently. It shows why consuming a different number of challenge bytes before the next message changes subsequent challenges.

Buffering observed data in64-byte SHA blocks is an implementation detail: writing one contiguous observed run in different chunks gives the same digest. Switching between observe and sample is a protocol event and must match.

An explicit observe accessor can switch the mode even if its payload is empty. Removing an empty operation is therefore safe only after checking the surrounding call sequence.

## 5. What enforces receive-before-sample

The algebraic protocols themselves call receive/send before sample; the transcript wrapper hashes their serialized messages as they pass. For example, BaseFold reads its round coefficient and any scheduled root before drawing the next folding challenge.

The default initial sampling bytes are publicly predictable. Unpredictability against a prover's already fixed message comes from absorbing that message and the agreed preceding context before sampling—not from keeping the challenger implementation secret.

Fiat–Shamir challenges are deterministic. The interactive bound $d/|K|$ is an ideal-coin statement; an adversary trying many transcript prefixes requires a random-oracle/query-budget analysis. Matching source call order is necessary evidence, not that complete security theorem.

## Validation and recall

[Independent model](../../work/verify_lessons045_048.py) reproduces two pinned challenger transition sequences,70chunk/refill fixtures and80matching prover/verifier tape runs. Advice mutation preserves the modeled challenge state while changing its committed hash. Truncation and unread-byte checks are recorded separately. No Rust or complete Binius proof execution.

**Takeaway:** “present in the proof” and “absorbed into Fiat–Shamir” are separate properties.

**Recall:** why can a modified authentication path leave the challenge stream unchanged and still cause rejection?
