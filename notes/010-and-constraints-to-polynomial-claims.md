# Lesson 010 — From AND words to polynomial claims

2026-10-23 · BitAnd, baseline construction · Introduced; understanding unchecked.

## A table with two kinds of index

For each word-level constraint row x, evaluate its operands A[x], B[x], C[x]. Correctness means A[x] AND B[x] = C[x]. Unpack their bits conceptually:

\[
a(i,x)=\operatorname{bit}_i(A[x]),\quad
b(i,x)=\operatorname{bit}_i(B[x]),\quad
c(i,x)=\operatorname{bit}_i(C[x]).
\]

There is one row index and one bit-position index. For 2^ℓ padded rows, x has ℓ Boolean coordinates and the 64 bit positions have six Boolean coordinates i. The conceptual bit tables have 2^(ℓ+6) entries. They need not be expanded to one large-field element per bit in actual memory.

## A small four-bit illustration

Use two rows and only four low bits for the teaching example:

| Row x | A | B | C |
|---|---|---|---|
| 0 | 1010 | 1100 | 1000 |
| 1 | 0111 | 0011 | 0011 |

Bit position is counted from the least-significant end. For example, row 0 bit 2 has a=0, b=1, c=0, so its error is zero. Row 0 bit 3 has a=b=c=1, so its error is also zero because 1·1+1=0.

Each location has a bit residual

\[
e(i,x)=a(i,x)b(i,x)+c(i,x).
\]

All eight residuals in this example vanish. Changing C's row 0 word from 1000 to 1100 flips bit 2, producing exactly one nonzero residual. In actual Binius64 words this reasoning covers all 64 positions.

## Interpolate operands, then form a composition

Let ã,b̃,c̃ be multilinear extensions of the bit tables. Form

\[
G(I,X)=\widetilde a(I,X)\widetilde b(I,X)+\widetilde c(I,X).
\]

On the Boolean cube, G equals the bit residual table. Therefore the constraints hold exactly when G vanishes on that cube. G's individual degree is at most two, even though ã,b̃,c̃ each have individual degree at most one.

Do not confuse G with E, the multilinear extension of the residual table itself. They agree on Boolean inputs but can differ elsewhere. For the valid example, E is the zero polynomial. Yet, using four-bit coordinates (i₀,i₁,x) and the F₄ point (1,α,1), the operand composition G evaluates to 1. This is consistent with Lesson 008: an honest terminal constraint-polynomial value need not be zero.

## Apply the previous lesson

For a uniform challenge τ in K^(ℓ+6), the baseline zerocheck claim is

\[
0\stackrel?=\sum_{(i,x)\in\{0,1\}^{\ell+6}}
\operatorname{eq}_{\ell+6}(\tau,(i,x))\,G(i,x).
\]

This is E(τ)=0. If some constraint fails, E is a nonzero multilinear; the initial random test has error at most (ℓ+6)/|K|. MLE-check then reduces that weighted-sum claim to an evaluation of G at a fresh point ρ. Since G has individual degree at most two, a generic factored MLE-check has degree-two rounds. Ordinary sumcheck on the whole eq·G product instead has degree-three rounds. These are baseline bounds and degrees, not the concrete optimized Binius64 parameter calculation.

The prover supplies values a*,b*,c* claimed to equal ã(ρ),b̃(ρ),c̃(ρ). The terminal composition check compares the reduced claim with

\[
a^*b^*+c^*.
\]

Authenticating those three operand claims is still necessary. The operands are XORs of shifted witness references, so the shift reduction must connect these values to the committed witness and public data. The verifier cannot simply trust the three scalars.

## How actual Binius64 changes the baseline

The specification's BitAnd reduction keeps the row axis multilinear but replaces the six Boolean bit coordinates by one univariate coordinate over a fixed 64-point domain D inside the AES field. Its operand polynomials have degree below 64 on that bit axis. It also uses fixed small-field challenge coordinates on the row axis, a univariate-skip message, and batching with multiplication-operand re-randomization claims. Those changes are separate derivations; the baseline above explains the relation they ultimately establish.

Next lesson develops this mixed univariate/multilinear representation, called oblong-multilinearization. It is not the same as storing 64 independent SIMD lanes or treating word AND as one field multiplication.

## Concrete implementation observations

At pinned revision `4428e759dd27956bd5b9ae5025d67eb8ccc1accf`:

- In `prover/src/prove.rs`, `build_operation_columns` evaluates a requested operand position across every constraint. Actual rows are materialized; padding to the next power of two is handled by the reduction. An empty AND set has a standing zero row rather than an omitted AND phase.
- The BitAnd call requests only A and B columns. Its entry point `protocols/bitand/prove.rs` accepts exactly two columns, samples the non-fixed zerocheck challenges, computes/sends the univariate message, samples the bit-axis challenge, and calls the re-randomization prover.
- `UnivariateRoundProver::fold` builds bit-axis Lagrange weights and calls `BitAxisFolder::fold_bitand_operands`, then constructs `quadratic_mlecheck_prover` with composition a·b−c. In characteristic two this is a·b+c.

The source documents that C is derived as A AND B. This saves evaluating/storing the actual C column in this path. It does not make an arbitrary actual C operand acceptable: the claimed derived-C evaluation must later match the circuit's actual C expression through the wiring/shift check. In the changed-word example above, derived C is 1000 while the erroneous referenced C is 1100, and their bit-table evaluations differ. Full end-to-end authentication is a later trace, not established solely by this interface inspection.

Inspected: table assembly, the BitAnd entry point, and UnivariateRoundProver's orchestration/fold method. Still untraced: BitAxisFolder internals, detailed univariate-message computation, re-randomization, and the complete shift-verifier path. No Rust execution.

## Recall

- What does an AND table index identify? Both a constraint row and a bit position.
- Why can the constraint composition have degree two? It multiplies two multilinear operand polynomials.
- Does a terminal a*b+c check authenticate a,b,c? No; their connection to witness operands is another obligation.

Sources: [Blueprint §§3.5 and 4.3](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex); [operation-column assembly](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/prove.rs#L500); [BitAnd entry point](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/bitand/prove.rs); [UnivariateRoundProver](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/bitand/prover.rs).

Validation: independently checked the two-row/four-bit table, every Boolean residual, a single-bit mutation's residual MLE at all 64 F₄³ points, and a valid operand composition with a nonzero off-cube value. Mathematical checks, not an execution of the Rust BitAnd proof.
