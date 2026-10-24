# Lesson 009 — The Binius64 constraint language

2026-10-23 · Constraint language · Introduced; understanding unchecked.

## Words and operands

Binius64 describes computation as relations on 64-bit words. In the pinned Rust implementation, Word is a transparent u64 wrapper. At the relation level, an operand is an XOR of references to words, each with up to two fixed shifts applied in sequence:

\[
A=\bigoplus_{t\in L}\operatorname{outer}_t(
\operatorname{inner}_t(w[\operatorname{index}_t])).
\]

The list L is circuit data. Referenced word contents are supplied through the public/prover data. Empty lists evaluate to zero; duplicate identical terms cancel. Shift types and amounts in these references are fixed by the circuit, not arbitrary runtime secret shift amounts. Variable shifts need an additional construction.

## A relation with concrete values

Consider

\[
((x\ll1)\oplus y)\mathbin{\&}z=t.
\]

For x=5, y=3, z=15, t=9, the left operand is (10 XOR 3)=9 and 9 AND 15=9. All are actual 64-bit words; the small integers simply have zero high bits. Logical left shift discards bits beyond position 63.

This is one BitAnd constraint with three operands:

- A lists x shifted left by one, and y unshifted.
- B lists z unshifted.
- C lists t unshifted.

At this constraint-language level, no extra witness word is required just to store x<<1 or its XOR with y. The frontend may make its own choices about materializing and reusing expressions; its optimization passes have not yet been traced.

The native constructor shape uses `AndConstraint::abc` with arrays/iterators of `ShiftedValueIndex`. For example, its A argument can be `[ShiftedValueIndex::sll(x, 1), ShiftedValueIndex::plain(y)]`, where x and y are ValueIndex references, not the raw integers 5 and 3.

## Four native relations

Every capital operand below is an XOR list of shifted word references.

| Specification name | Rust type | Required relation |
|---|---|---|
| Zero | ZeroConstraint | D = 0 |
| BitAnd | AndConstraint | A AND B = C, bit by bit |
| IntMul | ImulConstraint | A·B = LO + 2⁶⁴·HI over unsigned integers |
| BinMul | BmulConstraint | A·B = C in GHASH F₂¹²⁸; each element uses low/high word operands |

Zero expresses linear relations: to require z=x XOR y, require x XOR y XOR z=0. A Zero constraint is a native relation type; zerocheck from Lesson 008 is a proof technique. These names refer to different layers.

A BitAnd constraint represents 64 equations aᵢbᵢ+cᵢ=0 over bits, one for each position i. It is not one GHASH multiplication. A nonzero word residual `(A & B) ^ C` records exactly which bit positions fail.

IntMul retains the full product: (2⁶⁴−1)·2 gives HI=1 and LO=2⁶⁴−2. Treating it as wrapping multiplication alone would omit required information. The constraint operand storage order is [a,b,lo,hi], whereas Word::imul returns (hi,lo); keep those interfaces distinct.

BinMul assembles an element from a low and high word, where their bit positions supply polynomial coefficients 0–63 and 64–127. These bits are multiplied as a GHASH field element. For example, raw patterns 3 and 3 multiply to 5 in GHASH, versus the integer product 9 and word AND 3.

## Why two shifts per reference?

`ShiftedValueIndex` stores a ValueIndex plus `[inner, outer]`. Its evaluation reads the word and applies `outer(inner(word))` in that order. Two shifts can express useful operations without an intermediate constrained word.

Example: logically shift left 40, then arithmetically shift right 40. This sign-extends the original low 24 bits into a 64-bit word. Arithmetic right shift repeats the current sign bit; it differs from logical right shift, which inserts zeros.

The eight shift families cover logical left/right, arithmetic right, and rotate right, each at full 64-bit width or independently on the two 32-bit halves. In the pinned Rust constructors full-width shift amounts must be below 64 and half-width amounts below 32. The specification models half-width amounts with five effective bits; the Rust Shift type restricts the stored range directly.

These fixed shifts and XOR combinations are linear over F₂ on the word's bits. They need no separate nonlinear gate in this language, but their wiring still must be tied to the committed witness. “Built into an operand” is not “zero CPU work” or “no proof obligation.” The shift reduction will handle that connection.

## Data layout and direct checks in code

The Blueprint's prover data contains public constants, instance-specific public inputs/outputs, and private witness/intermediate words. The code uses segmented ValueIndex references: the top two bits of its u32 encode Constant, InOut, Private, or Scratch; the bottom 30 bits encode the index within the segment.

Scratch is for circuit-evaluation temporaries. ValueSegment::is_referenceable marks it as unavailable to constraint operands. End-to-end system validation remains to be traced. Calling data “private” in this layout is also not a proof that the transparent proof transcript hides it; zero knowledge remains a separate protocol layer.

ValueVec::eval_operand delegates to shift::eval_operand, which XORs the evaluated terms from an initial zero word. The native constraint verify methods read this full value vector and directly check their relation. They are witness-validation helpers, not the succinct cryptographic verifier: they have access to values the external verifier does not receive.

This lesson inspected reference construction/evaluation, range bounds, operand evaluation, and the native direct checks. It did not compile a circuit, run Rust, trace all shift normalization cases, or establish proof-system soundness from these helpers.

## Next step

Next: organize BitAnd constraints into a constraint-index × bit-position table and derive the baseline polynomial check. Then explain why actual Binius64 treats the 64-position bit axis specially and how the shift reduction reconnects operand claims to witness data.

Sources: [Blueprint §3](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex); [constraint types and direct checks](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/core/src/constraint_system/constraint.rs); [shifted references](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/core/src/constraint_system/shift.rs#L736); [segmented indices](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/core/src/constraint_system/value_index.rs).

Validation: independently checked the concrete shifted/XOR/AND example and detection of each of its 64 single-bit output mutations; empty/duplicate operands; the overflow product; GHASH examples; and double-shift sign extension at representative boundaries. These checks validate the lesson arithmetic, not Rust execution or every shift variant.
