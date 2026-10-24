# Lesson 048 — Finishing an opening and finishing a proof parser are separate checks

Introduced: 2026-10-23. Understanding unchecked. Binius64 revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** acceptance has several layers: validate configured data, verify the algebra and deferred openings, then confirm the proof parser consumed exactly what was expected.

Prerequisites: [queued obligations](039-linear-oracle-queues-and-relation-batching.md), [transcript tape](045-transcript-tape-and-challenger-state.md) and [statement framing](046-public-statement-context-and-transcript-framing.md).

## 1. Setup fixes structure before reading a proof

[Verifier::setup](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/verify.rs#L332) validates the constraint system, derives the oracle specifications and creates the BaseFold compiler.

Oracle specifications are obtained by structurally executing the IOP verifier against [OracleSetupChannel](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/channel/oracle_setup.rs#L151). Its dummy arithmetic performs no real cryptographic verification. It records oracle lengths, witness-dependence/ZK flags and commitment-round boundaries.

This is a layout dry run, not evidence that a witness satisfies the circuit. The real verifier later uses those shapes and checks supplied input/output length, algebraic equations, public wiring and commitments.

ConstraintSystem::validate checks segment bounds, references and shift legality. It is not a complete bound on every user-selected FRI rate or every possible allocation/serialization cost; Lesson047 records the separate native query-width limit.

## 2. Channel finish discharges deferred work

[Native Verifier::verify](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/verify.rs#L409) calls verify_statement, then channel.finish.

The first runs reductions and checks the returned public wiring claim. The second verifies queued linear-oracle relations through the combined BaseFold/Merkle opening.

If either fails, the proof has not verified. Returning successfully from an inner reduction that merely queued an oracle relation is insufficient.

## 3. Transcript finalize checks exhaustion

The native verify API borrows the transcript and returns after channel finish. Its caller must separately call [VerifierTranscript::finalize](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/transcript/src/transcript.rs#L72), which rejects if any bytes remain.

The condition is simply

$$
\text{consumed proof bytes}=\text{raw proof-tape length}.
$$

The transcript destructor logs a warning for unread bytes; dropping it is not an error-returning substitute for finalize.

[The example check_proof helper](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/examples/src/lib.rs#L179) and verifier CLI perform both verification and finalization. Truncated reads return errors through the relevant parser/channel. Valid algebraic checks alone do not establish tape exhaustion.

## 4. The transport envelope has its own grammar

[Proof serialization](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/core/src/constraint_system/proof.rs#L84) writes:

1. Format version, currently one, as a32-bit integer.
2. Challenger-name string, with its length.
3. Raw proof-tape byte vector, with its length.

The generic usize serializer uses a32-bit length representation, independent of host pointer width. The decoder rejects an incompatible version. The example CLI separately requires the stable name HasherChallenger<Sha256>.

These bytes wrap the transcript. They are not all fed to its challenger: verification extracts the raw tape and creates the chosen challenger separately.

The Proof type also transports ordinary non-ZK proofs. Its documentation's “zero-knowledge proof” wording is not an enforced privacy property.

## 5. Inner and outer trailing bytes are different

Appending bytes **inside the raw transcript vector** leaves unread transcript bytes, caught by finalize.

Appending bytes **after the serialized Proof object** concerns the outer decoder. DeserializeBytes reads one object from its input and does not generally require that the surrounding stream be exhausted; this supports composing serialized objects.

The inspected example CLI deserializes the envelope from a slice without an explicit final outer-slice exhaustion check. Its later transcript finalization checks the extracted tape, not unrelated bytes after the envelope. This is a parser-boundary observation, not evidence that the algebraic proof can be forged.

Applications requiring a unique whole-file encoding must specify that outer parsing contract too.

## 6. Debug markers and compatibility

Transcript Options controls debug-string writes/reads, defaulting to the build's debug_assertions setting. Where protocol code uses those markers, both sides must agree on the option and call order. A marker is ordinary bytes through its selected writer, not a self-describing container feature.

The explicit field/word widths, proof-envelope version, configured shape and optional context prefix each solve different compatibility/binding problems. None should silently be substituted for the others.

## Validation and recall

[Independent byte model](../../work/lessons045-048-validation.json) checks80matching tape parses and240advice/trailing/truncation fixtures, plus envelope round trips, version mismatch and inner-versus-outer trailing distinctions. These are concrete parser/state models. No full malformed-input audit or Binius Rust proof execution.

Rust1.94.1 and Cargo1.94.1 are available locally. A pinned checkout/build and end-to-end run remain scheduled in the implementation stage; toolchain availability alone is not an executed test.

**Takeaway:** channel finish proves queued obligations; transcript finalize proves parser exhaustion.

**Recall:** which layer catches an extra byte inside the transcript, and which layer must address an extra byte outside its serialized envelope?
