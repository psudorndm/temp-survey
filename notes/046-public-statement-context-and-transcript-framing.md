# Lesson 046 — Which statement is actually bound into the transcript?

Introduced: 2026-10-23. Understanding unchecked. Binius64 revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** algebraically checking a configured relation and hashing its identity into the challenge prefix are different mechanisms. Trace both instead of assuming every public parameter is automatically hashed.

Prerequisite: [transcript operations](045-transcript-tape-and-challenger-state.md).

## 1. Native public values are observed before the trace root

[IOPProver::prove](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/prove.rs#L98) observes witness.inout(), then commits the packed private trace.

[IOPVerifier::verify_statement](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/verify.rs#L292) observes the supplied concrete input/output words, delegates to verify and discharges its returned native wiring claim. That verification checks the supplied word count against the configured n_inout.

Each word contributes eight little-endian bytes. These public bytes affect challenges but are not copied into the raw proof tape.

The public segment used by the shift check is the configured constants followed by those input/output words. The constants come from the verifier's constraint system rather than a second instance-supplied list.

## 2. Configured parameters versus observed prefix

For the inspected plain entry points:

| Item | How it participates |
|---|---|
| Instance input/output words |Explicitly observed before the trace root|
| Constraint-system constants and wiring |Used by the configured verifier's checks|
| Code rate, query count and oracle shapes |Used by the configured compiler and proof grammar|
| Merkle roots and algebraic messages |Serialized and observed as messages|
| Merkle opening advice |Serialized, then authenticated against observed roots|
| Caller-supplied context prefix |Observed only if the caller inserts it|

The plain Prover/Verifier APIs accept an already constructed transcript. Their inspected bodies do not automatically hash a constraint-system digest, configuration digest or Binius protocol/version tag at entry.

This does not mean the verifier ignores the circuit: it computes checks for that circuit. It means “fixed verifier relation” and “explicit challenge-domain binding across applications/circuits” must be distinguished. No cross-circuit attack has been demonstrated here.

For an application that requires an explicit domain binding, a possible agreed prefix is a canonical encoding of

$$
(\mathrm{application\ tag},H(\mathrm{CS}),
\mathrm{configuration\ identifier}).
$$

Both sides would observe it before the protocol. This is an illustrative application construction; the current plain entry point does not insert it automatically. Its security and encoding requirements depend on the intended application.

The SHA-256 parent-compression tag in Lesson043 separates Merkle hash roles. It does not identify a circuit or replace such an application prefix.

## 3. Byte framing is part of the protocol grammar

TranscriptWriter::write_slice serializes each item in sequence without adding a count. recv_many and read_scalar_slice consume a count supplied by the verifier's protocol.

Consequently, consecutive writes of byte runs $A,B$ and a write of $A\Vert B$ can produce the same observed run. There are no automatic per-call type tags:

$$
\operatorname{Ser}_{64}(a)\Vert\operatorname{Ser}_{64}(b)
=\operatorname{Ser}_{128}(a+2^{64}b)
$$

for the inspected little-endian integer/field representations. The addition on the right describes bit placement in an integer representation, not field addition.

This is unambiguous when the agreed protocol fixes types, lengths, configuration and sampling boundaries. Caller-defined variable-length context needs its own unambiguous framing.

Do not confuse this with SerializeBytes for a slice or vector: that serializer writes a length prefix. TranscriptWriter::write_slice deliberately iterates items instead. [The serializer bodies](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/utils/src/serialization.rs#L203) make the difference explicit.

## 4. An existing optional message-binding mode

[signature::observe_message](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/signature.rs#L74) hashes an external message with the selected leaf hash and observes that digest first.

ZKProver::prove_sig and ZKVerifier::verify_sig invoke this helper before their ordinary proof logic. The signed message remains out of band; it is not stored in the raw proof tape.

Binding an empty message observes $H(\varepsilon)$, whereas selecting no message observes nothing. These modes have different prefixes. The mode is not automatically recorded in the raw proof, so callers must select matching methods.

The helper supplies message binding. Claims such as unforgeability/non-malleability still require the underlying knowledge/Fiat–Shamir/hash assumptions; changing a prefix alone is not their complete proof.

## 5. Metadata does not silently seed the challenger

The transport Proof container stores a format version, challenger-name string and raw transcript bytes. The example verifier checks its expected challenger name outside the transcript and then starts a default challenger on the extracted data.

Those metadata checks establish format/algorithm compatibility. They do not automatically absorb the string or version into the challenge state, and the container's name/documentation does not make a plain proof zero knowledge.

[Lesson048](048-proof-finalization-transport-and-setup-validation.md) separates transport decoding from transcript exhaustion and deferred-opening completion.

## Validation and recall

[Independent checks](../../work/lessons045-048-validation.json) cover30context/signature prefix fixtures, raw word/field byte equivalence, concatenated write boundaries and sampling-cursor differences. These validate concrete byte behavior, not an application-wide context-binding theorem. No Rust proof execution.

**Takeaway:** specify both the relation being verified and the context used to derive its challenges.

**Recall:** which public data does the native plain entry point explicitly observe, and which identity remains configured outside that prefix?
