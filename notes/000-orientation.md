# Lesson 000 — Binius orientation

2026-10-23 · Scope: first intuition, not a derivation.

## The problem

A prover supplies a witness: inputs and intermediate values for a computation. A verifier wants evidence that these values satisfy the computation's constraints.

## The binary-field intuition

For bits, field addition in F₂ is XOR and field multiplication is AND:

- 1 + 1 = 0.
- 1 · 1 = 1; every product with 0 is 0.

Thus a bitwise AND relation can be expressed for every bit position i as

`a_i · b_i + c_i = 0`.

Example: `1010 AND 1100 = 1000`. At each of the four positions, the bit equation holds. Binius64 uses 64-bit words in its constraint language, so one word AND relation expresses 64 such bit relations.

## The distinction to retain

Packing 64 bits into a word does not make word AND equal to a single multiplication in F₂⁶⁴. Coordinatewise bit multiplication and extension-field multiplication are different operations. Binius64 also supports unsigned integer multiplication, which is a third operation.

## Where the protocol enters

The bit equations define correctness. Polynomial reductions, commitments, and transcript challenges provide evidence about that correctness. The zero-knowledge composition adds privacy.

Recall: What is special about 1 + 1 in a binary field?

Next: build a tiny extension field by treating bit strings as polynomials.

Sources: [Blueprint §§1–3](https://www.binius.xyz/spec.pdf); [AndConstraint::verify](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/core/src/constraint_system/constraint.rs#L188). Example is pedagogical; actual words have 64 bits.
