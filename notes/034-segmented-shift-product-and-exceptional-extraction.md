# Lesson 034 — Fold the public/private selector without concatenating

Introduced: 2026-10-23. Understanding unchecked. Code revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** keep public and hidden arrays separate until their selector coordinate is bound. The first product round can be computed from the dense hidden pass plus a short public-prefix correction.

Prerequisites: [five-factor endpoint](031-five-factor-shift-sumcheck-and-witness-closure.md), [sparse keys](032-sparse-shift-keys-and-witness-weighted-rows.md) and [early shift rounds](033-implicit-shift-weights-and-sparse-sumcheck-rounds.md).

## 1. What remains after the bit/slot rounds

At fixed witness-bit point $r_J$, the prover folds each word into a field value. Let $P,H$ be the public and hidden folded word arrays. Let $Q,M$ be their corresponding wiring-weight arrays, already scaled by

$$
f=\widetilde L_z(r_I)\,
\widetilde S_{\mathrm{outer}}\,
\widetilde S_{\mathrm{inner}}.
$$

Each segment is padded with zeros over a common word-axis span. The selector variable $T$ gives

$$
W(T,y)=(1-T)P[y]+TH[y],\qquad
A(T,y)=(1-T)Q[y]+TM[y].
$$

The remaining claim is the sum of $W(T,y)A(T,y)$ over Boolean $T,y$.

## 2. Form the first quadratic round sparsely

The selector round is

$$
R(T)=\sum_yW(T,y)A(T,y).
$$

Its value at one and quadratic coefficient are

$$
R(1)=\langle H,M\rangle,\qquad
R(\infty)=\langle P+H,Q+M\rangle.
$$

Suppose the public arrays are supported on a short prefix $U$. Outside that prefix $P=Q=0$. In characteristic two,

$$
R(\infty)
=\langle H,M\rangle
+\langle H|_U,M|_U\rangle
+\langle(P+H)|_U,(Q+M)|_U\rangle.
\tag{1}
$$

The second term cancels the prefix part of the dense pass, then the third replaces it with the correct prefix composition. This is an exact algebraic correction, not a heuristic.

The source accumulates wide products, reduces them and sums packed lanes. It uses these two samples and the incoming sum claim to recover the ordinary quadratic round.

## 3. Reuse the hidden buffers after sampling

After receiving selector challenge $\eta$, SegmentPair::fold overwrites its hidden buffer with

$$
H\longleftarrow\eta H+(1-\eta)P_{\mathrm{padded}},
$$

and similarly folds the wiring-weight pair. This equals folding the highest coordinate of a fully concatenated public/hidden array.

The combined arrays are never materialized at twice the common segment span. Remaining word-coordinate rounds use the standard dense bivariate-product prover.

The phase2 body takes the maximum of public and hidden spans and explicitly zero-extends the hidden arrays if public data are longer. Older “hidden is always wider” comments are not the contract followed here.

## 4. Rebuild the wiring segment scalars from keys

For one word, build_monster_segments adds every key's constraint accumulation, weighted by its inner and outer slot interpolation weights and by $f$.

The sequence weight factors:

$$
\operatorname{eq}(r_{S_1},s_1)\operatorname{eq}(r_{V_1},v_1)
\operatorname{eq}(r_{S_2},s_2)\operatorname{eq}(r_{V_2},v_2).
$$

Two small slot tensors suffice. Operand weights occupy the inner stride of a per-active-sequence scalar table, so each key can index a contiguous block. Wide key accumulations are combined before a word scalar is reduced.

The code builds public and hidden wiring arrays separately. It also folds both word segments against one reused bit-query table. This is source-level allocation and reuse behavior, not a memory or throughput measurement.

## 5. The current prover still uses two quotient extractions

Let $t_{\mathrm{trace}}$ be the dense prover's final value for the combined witness, and $t_{\mathrm{scaled}}$ its value for the scaled wiring table. The current phase2 code recovers:

$$
g^\star=t_{\mathrm{scaled}}\,f^{-1},
\qquad
h^\star=
\left(t_{\mathrm{trace}}-(1-\eta)p^\star\right)\eta^{-1}.
\tag{2}
$$

Here $p^\star$ is the directly evaluated padded public segment. Both inverses use invert_or_zero, which returns zero when its input is zero.

This is a correction in scope to Lesson031's division-free endpoint discussion: the **verifier** receives the private value and does not solve for it, but the **honest prover implementation** derives it from the folded value by equation (2). The earlier bit-index stages deliberately avoided division; these final extractions still contain it.

At $\eta=0$, the combined witness reveals only $p^\star$, not $h^\star$. For example $p^\star=1,h^\star=\alpha$ gives $t_{\mathrm{trace}}=1$, yet invert_or_zero extraction returns zero. The shift endpoint can still hold, but a later authentication of that incorrect private claim may reject.

At $f=0$, the scaled wiring value is zero regardless of the true bare wiring value. Extraction returns zero; native wiring discharge can then disagree with the true public wiring evaluation.

The source explicitly records these as negligible-probability incompleteness events. These are honest-prover extraction limitations, not an established soundness attack, and no implementation fix was applied.

## 6. A conservative extraction-event bound

Under ideal independent uniform sumcheck challenges, the segment selector is uniform, so

$$
\Pr[\eta=0]=1/|K|.
$$

The Lagrange-weight MLE is nonzero because its Boolean values sum to one and has total degree at most six. Each interpolated shift indicator is a nonzero21-variable MLE of total degree at most21.

Consequently their product $f$, considered as a polynomial in the jointly sampled coordinates, is nonzero and has total degree at most

$$
6+21+21=48.
$$

Schwartz–Zippel and a union bound give the extraction-specific exceptional-set bound

$$
\Pr[f=0\ \text{or}\ \eta=0]
\leq\min(1,49/|K|).
$$

This does not require assuming the two events are independent. It is conditional on the ideal uniform challenge model and the stated coordinate/field setup. It does not account for earlier interpolation shortcuts, Fiat–Shamir security, commitment failure or the full protocol's completeness/security budget. An exceptional coordinate need not reject every witness.

## 7. Return the private claim, then discharge the separate obligations

The source reverses binding-order challenges and returns witness coordinates $r_J\Vert r_Y\Vert\eta$ internally, with the segment selector last in the word run. It sends the extracted private value, while its wiring scalar is returned for the caller to send at the matching public-input/wiring stage.
The top-level prover explicitly removes that final segment coordinate before constructing the private ring-switch point. The private query is at $r_J\Vert r_Y$; it must not include the public/private selector.

The verifier separately authenticates public values, checks the five-factor terminal product and discharges the wiring claim. It then sends the bit-level private claim to ring-switching and the packed commitment opening. The scalar equation alone is not an opening proof.

## Sources and checks

- [Phase2](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/shift/phase_2.rs): prove_phase_2, SegmentPair::fold, first_round_coeffs and run_sumcheck bodies, including both quotient extractions.
- [Segment scalar construction](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/shift/key_collection/collection.rs): build_monster_segments.
- [Verifier endpoint](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/shift/verify.rs): check_eval and outstanding wiring/private-value claims.
- [Top-level prover handoff](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/prove.rs): point split, public claim, wiring scalar and private ring-switch call.

Independent models check320 segmented first-round challenge cases, including public-longer-than-hidden shapes, against direct concatenated-array folding and product evaluation. All3,840 scalar private extractions with nonzero selector recover the correct value over $\mathbb F_{16}$. Separate exact zero-selector and zero-scale examples exhibit the lost information.

These are scalar reference calculations. They do not execute Rust, packed lanes or final authentication. The degree-based bound above is a mathematical derivation, not a measured failure rate.

**Remember:** segment separation saves a combined buffer; quotient extraction still has exceptional zero factors.

Next: ring-switching converts the resulting bit-level claim into a linear relation on packed field elements.
