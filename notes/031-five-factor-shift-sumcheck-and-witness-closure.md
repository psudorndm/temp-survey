# Lesson 031 — Five factors connect operations to the witness

Introduced: 2026-10-23. Understanding unchecked. Code revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf. Blueprint revision: 05061c6f9eca316b6844396883afb9ae37ff2fe9.

**One idea:** write the claimed operand evaluations as a sum of five factors. Each summation coordinate appears in only two factors, so ordinary sumcheck has degree two despite the five-way product.

Prerequisites: [oblong weights](011-oblong-multilinearization.md), [unified operation claims](018-zero-reduction-and-unified-point.md), and [shift routing](030-shift-indicators-and-bit-carry-evaluation.md).

## 1. What arrives at the shift reduction

The native operation reductions leave14 operand slots in order Zero1, BitAnd3, IntMul4, BinMul6. Their bit axis is folded at the same univariate point $z$, and their row matrices are lifted to one padded constraint point $r_X$.

Sample a fresh operand-axis point $r_M\in K^4$ and form

$$
s=\sum_{m=0}^{13}\operatorname{eq}_4(r_M,m)\alpha_m.
$$

The remaining two slots of the16-entry cube are zero. The batch is sampled after the operand scalars are fixed. For a fixed nonzero padded error vector this local tensor test contributes at most $4/|K|$.

The earlier rerandomization and zero-padding scales are prerequisites; this reduction consumes the full padded claims, not unscaled prefix values.

## 2. Sparse wiring collects the operand terms

A native operand is an XOR of terms. One term names a witness word, an inner shift and an outer shift.

Let the Boolean wiring array be

$$
\mathcal W(m,x,s_1,v_1,s_2,v_2,y,\sigma),
$$

with $\sigma=0$ selecting public words and $\sigma=1$ private words. A set entry indicates that this term occurs in operand $m$ of constraint row $x$. Repeated identical terms cancel in characteristic two.

Contract its operand and constraint axes at the old points:

$$
G(s_1,v_1,s_2,v_2,y,\sigma)
=\sum_{m,x}\operatorname{eq}(r_M,m)\operatorname{eq}(r_X,x)
\mathcal W(m,x,s_1,v_1,s_2,v_2,y,\sigma).
$$

Its MLE is simply the wiring MLE with those two axes fixed. The constraint and operand indices are no longer variables of the upcoming sumcheck.

For the bit folding, let $d_i$ be domain point $i$ in the64-point AES additive domain and set

$$
L_z[i]=\delta_D(z,d_i).
$$

Write $\widetilde L_z(I)$ for the six-variable MLE of these64 weights. They are univariate Lagrange weights lifted to a Boolean-indexed table, not an arbitrary six-coordinate equality tensor.

## 3. The five-factor identity

Let $w(J,Y,\Sigma)$ be the full bit-level trace, with both public and private segments zero-padded to their common word span. The operand claim becomes

$$
s=\sum_{\substack{i,k,j,\ s_1,v_1,\ s_2,v_2,\\y,\sigma\ \text{Boolean}}}
\widetilde L_z(i)\,
\widetilde S(i,k,s_2,v_2)\,
\widetilde S(k,j,s_1,v_1)\,
\widetilde G(s_1,v_1,s_2,v_2,y,\sigma)\,
\widetilde w(j,y,\sigma).
\tag{1}
$$

The five factors are output-bit weights, outer routing, inner routing, wiring and witness. Equality (1) follows by expanding the sparse operand XORs, composing both shifts and applying the oblong weights. It is a finite-sum identity on Boolean summation indices.

Its extension off the cube is a product of those MLEs; it need not itself be multilinear. Ordinary sumcheck handles its coordinate degree.

## 4. Why five factors still mean degree two

| Coordinate | Factors containing it |
|---|---|
| Output bit $I$ | $L_z$, outer shift |
| Intermediate bit $K$ | Outer shift, inner shift |
| Witness bit $J$ | Inner shift, witness |
| Inner amount/variant $S_1,V_1$ | Inner shift, wiring |
| Outer amount/variant $S_2,V_2$ | Outer shift, wiring |
| Word/segment $Y,\Sigma$ | Wiring, witness |

Every factor is multilinear in its own coordinates, and each coordinate appears in at most two factors. Thus the summand has individual degree at most two.

The code has three six-coordinate bit-index axes and two nine-coordinate shift slots:

$$
\operatorname{SHIFT\_LOG\_VARS}=3\cdot6+2(6+3)=36.
$$

If each padded segment has $2^L$ words, the combined word axis has $L+1$ coordinates, including the segment bit. The verifier runs

$$
36+(L+1)
$$

ordinary sumcheck rounds of degree two. The Blueprint's $36+\ell_{\mathrm{words}}$ uses the full word-axis count, which includes this segment coordinate in the implementation.

Under authenticated terminal factors, the isolated sumcheck bound is $2(37+L)/|K|$. This is not the full constraint-system soundness bound.

## 5. Binding order enables the prover phases

Sampling binds the axes in this order:

$$
V_2,\ S_2,\ V_1,\ S_1,\ J,\ K,\ I,\ (Y,\Sigma).
$$

Within the last word run the highest segment coordinate is bound before the lower word coordinates. The source reverses sampled challenges into coordinate order and extracts each run explicitly.

The prover's top-level prove function groups this one transcript into stages: bind shift slots/witness-bit position using sparse key information, bind intermediate-bit position, bind output-bit position, then reduce the remaining word sum against wiring and witness data. The current top-level comments use inconsistent phase-count wording; the actual calls establish the sequence.

The first fixed shift is the outer one, peeling the composition from the output end inward. Efficient key collection and stage kernels still need a separate implementation walkthrough; this note covers the factorization, top-level call graph and verifier endpoint.

## 6. What the verifier computes at the endpoint

At the terminal points it computes:

$$
\ell^\star=\widetilde L_z(r_I),
\quad
s_2^\star=\widetilde S(r_I,r_K,r_{S_2},r_{V_2}),
\quad
s_1^\star=\widetilde S(r_K,r_J,r_{S_1},r_{V_1}).
$$

It receives a claimed wiring evaluation $g^\star$ and a claimed private witness evaluation $h^\star$. Let $p^\star$ be the public segment evaluation, including padding, verified by the caller. With segment coordinate $\eta$,

$$
w^\star=(1-\eta)p^\star+\eta h^\star.
$$

The terminal product check is

$$
t=\ell^\star s_2^\star s_1^\star g^\star w^\star.
$$

Reading $h^\star$ avoids solving this equation by dividing by random factors, which could be zero. The received value is still sent to subsequent witness authentication.
This describes the verifier interface. [Lesson034](034-segmented-shift-product-and-exceptional-extraction.md) subsequently traces the current honest prover, which still extracts the private and wiring values by quotient formulas using invert_or_zero; its exceptional zero-factor behavior is recorded there.

At a non-Boolean segment point, both $g^\star$ and $w^\star$ interpolate their public/private halves. Their product contains off-cube cross terms; that is consistent with a degree-two composition, not a reason to replace the product by separate segment products.

## 7. Public wiring is a real remaining check

WiringInfo stores public and hidden sparse bit vectors. Its address order is operand, inner shift, outer shift, word, constraint, all low-to-high. Segment selection interpolates their evaluations; a shorter public word span receives its explicit high-coordinate zero-padding factor.

check_eval returns a WiringEvalClaim carrying the public wiring function, its challenge inputs and the prover's claimed value. It does not silently authenticate that scalar by merely receiving it.

The native verify_statement path observes the public words, runs verification and calls check_native on the returned wiring claim. The symbolic/circuit interface may instead check it in constraints or export a deferred claim; an exported claim remains an obligation until discharged against the correct WiringInfo.

Direct sparse evaluation reads the circuit's operand terms. Hence this verifier path can have work proportional to circuit size even when the proof is small. Proof-size succinctness does not imply logarithmic verifier time.

The pinned sparse address representation also checks a concrete limit: the hidden wiring address has fewer than64 bits, yielding $L+\ell_{\mathrm{cons}}\leq41$ with the current22 lower term bits. This is a representation bound, not a cryptographic theorem.

## 8. Public values and private commitments are separate closures

The current verifier calls verify_public_eval before closing the shift product. That routine uses a ring-switch reduction and a sumcheck whose terminal factors the verifier evaluates from actual public words.

The private claim at $(r_J,r_Y)$ goes through ring-switching, then becomes an oracle relation against the committed hidden trace. The IntMul pushforward relations were already queued earlier. Concrete oracle opening verification and ring-switch identities are subsequent lessons; merely emitting these calls is not a proof that they have been fully analyzed here.

## Sources and independent checks

- [Blueprint §4.7](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex): sparse shift expression, five factors, coordinate degrees/order and direct verifier cost.
- [Shift verifier](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/shift/verify.rs): verify, check_eval, evaluate_words_mle and wiring-claim discharge methods.
- [WiringInfo](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/shift/wiring.rs): constructor, bit layout, segment padding and FieldFn evaluation interface.
- [Prover orchestration](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/shift/prove.rs): actual stage call sequence.
- [Verifier caller](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/verify.rs): public claim, wiring claim, private ring-switch/oracle relation and native statement discharge.
- [Public evaluation routine](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/reduction.rs): verify_public_eval and padding of the returned public value.

A two-bit-word miniature uses two variants, two shift slots, two operand columns, two constraint rows and two public/private word slots. Duplicate wiring terms cancel and the short public segment pads with zero. Across eight challenge fixtures, explicit sums over all512 Boolean points of the nine-variable product equal directly evaluated operands. There are144 quadratic coordinate-restriction checks and two complete nine-round sumcheck chains,18 rounds total. Public/private interpolation matches the direct bit trace.

These checks validate the factorization and miniature transcript algebra. They do not run Rust, the large sparse prover, ring-switching or BaseFold.

**Remember:** each coordinate touches two factors; public wiring, actual public values and private commitments must all close their respective claims.

Next: the sparse prover's key/phase implementation, then ring-switching and packed witness openings.
