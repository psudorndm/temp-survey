# Lesson 030 — A shift is a small routing matrix

Introduced: 2026-10-23. Understanding unchecked. Code revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf. Blueprint revision: 05061c6f9eca316b6844396883afb9ae37ff2fe9.

**One idea:** each output bit of a shift copies one input bit or zero. Encode that routing rule as a multilinear polynomial, then compose two shifts by summing over an intermediate bit index.

Prerequisites: [native shifted operands](009-binius64-constraint-language.md), [MLEs](005-multilinear-extensions.md) and [sumcheck](007-sumcheck-a-complete-transcript.md).

## 1. The Boolean indicator

For word width $W=64$, define

$$
S_v(i,j,s)=
\begin{cases}
1,&\text{output bit }i\text{ of variant }v\text{ at amount }s\text{ copies input bit }j,\\
0,&\text{otherwise}.
\end{cases}
$$

The four full-word variants satisfy:

| Variant | Source-bit condition |
|---|---|
| Logical left | $i=j+s$, with indices in range |
| Logical right | $j=i+s$, with indices in range |
| Arithmetic right | $j=\min(i+s,63)$ |
| Rotate right | $j=(i+s)\bmod64$ |

The arithmetic sign-fill may copy the sign bit into several output positions, but each particular output position still has one source. Zero-filled positions have no source.

For fixed variant and amount,

$$
\operatorname{shift}_{v,s}(w)_i=\sum_{j=0}^{63}S_v(i,j,s)w_j.
$$

The sum is in the binary field. Since a row selects at most one input bit, it exactly implements copying or zero fill.

## 2. Lift routing into polynomials

Each bit index and amount has six Boolean coordinates. The unique MLE $\widetilde S_v(I,J,S)$ therefore has18 variables. Adding three variant-selector coordinates gives one21-variable MLE

$$
\widetilde S(I,J,S,V)
=\sum_{v=0}^7\operatorname{eq}_3(V,v)\widetilde S_v(I,J,S).
$$

The shifted-word identity extends to arbitrary output/amount/word points:

$$
\widetilde{\operatorname{shift}}_v(I,Y,S)
=\sum_{j\in\{0,1\}^6}
\widetilde S_v(I,j,S)\widetilde w(j,Y).
$$

The right side is multilinear in its remaining free variables and matches the actual shifted bits on the Boolean cube, proving the identity by uniqueness. This is not interpreting a field-valued amount as an ordinary machine shift.

## 3. Two shifts need an intermediate index, not an intermediate witness

For inner shift $(v_1,s_1)$ and outer shift $(v_2,s_2)$,

$$
\operatorname{shift}_{v_2,s_2}
\bigl(\operatorname{shift}_{v_1,s_1}(w)\bigr)_i
=\sum_{k,j}S_{v_2}(i,k,s_2)S_{v_1}(k,j,s_1)w_j.
$$

Here $k$ is a bit of the intermediate word. It is only a summation index: the protocol does not add the whole intermediate word to the witness.

Order matters. In a four-bit illustration with $w=\mathtt{1101}$,

$$
(w\gg1)\ll1=\mathtt{1100},
\qquad
(w\ll1)\gg1=\mathtt{0101},
$$

where every operation keeps only four bits. The two routing matrices must be composed in the stated inner/outer order.

## 4. Transparent evaluation by a two-state carry recurrence

The verifier does not enumerate a $64^3$ truth table. The pinned evaluator checks the integer addition of bit indices one binary position at a time.

For logical right, the desired predicate is $i+s=j$. Keep two accumulated field values $N,C$, initially $(1,0)$, for correct lower positions with carry-out zero or one.

At one bit position, let $i,j,s$ now denote the three field-valued coordinates. Define

$$
b=js,\qquad a_j=j-b,\qquad a_s=s-b,\qquad
a_{\mathrm{eq}}=\operatorname{eq}_1(j,s).
$$

The next states are

$$
\begin{aligned}
N'&=(1-i)(a_{\mathrm{eq}}N+a_jC)+i(a_jN),\\
C'&=(1-i)(a_sC)+i(a_sN+a_{\mathrm{eq}}C).
\end{aligned}
$$

On Boolean coordinates this is the ordinary carry truth table. Each step introduces only its own three fresh coordinates, so both outputs remain multilinear in every coordinate. After six steps,

$$
N=\widetilde{\boldsymbol1[i+s=j]},\qquad
C=\widetilde{\boldsymbol1[i+s=j+64]}.
$$

Logical left uses the no-carry evaluator with input/output indices exchanged. Rotate right adds the no-wrap and wrapped predicates. Arithmetic right adds logical right to

$$
\left(\prod_{b=0}^5J_b\right)\widetilde{\boldsymbol1[i+s\geq64]}(I,S).
$$

The product selects the sign position $j=63$. A separate carry recurrence computes the overflow predicate without keeping the result index.

The source uses field interpolation at each bit, not integer carries on fractional inputs. Correct Boolean behavior plus multilinearity identifies the desired MLE.

## 5. Parallel half-word variants

The four32-bit variants apply the same rules independently in each half. Their indicators multiply the lower-five-bit evaluator by

$$
\operatorname{eq}_1(I_5,J_5).
$$

This prevents a bit from crossing halves. The evaluator ignores the top amount bit, effectively reading the amount modulo32. Native constructors may restrict half-word amounts more tightly; the polynomial amount axis still spans64 positions.

## 6. Push weights backward through a shift

If output bits carry weights $\psi_i$, the corresponding input weights are

$$
\psi'_j=\sum_i\psi_i S_v(i,j,s).
$$

This is the transpose of the shift routing map. In shift_operator_row, logical left moves these weights in the direction opposite to moving the word's bits; arithmetic right adds all sign-fill weights to the sign-bit position.

For two slots, the indicator weights factor across the inner and outer variant/amount axes. The prover can retain separate small tensors rather than a single table over all $512^2$ slot pairs. This describes the source's factorization, not a benchmark.

## Sources and validation

- [Blueprint §4.2](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex): indicator definition, composition and variant interpolation.
- [Verifier shift evaluator](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/shift/shift_ind.rs): evaluate_shift_inds, evaluate_variants, evaluate_sum_inds and evaluate_overflow_ind bodies.
- [Prover transpose operation](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/shift/monster.rs): OuterSlotWeights and shift_operator_row bodies. Other large operator-building helpers are not fully reviewed.
- [Native shifts](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/core/src/constraint_system/shift.rs): word operations and validated constructors.

Independent checks exhaust584 Boolean index triples at widths2,4,8 for all eight variants, compare112 off-cube recurrence points with direct interpolation over $\mathbb F_{16}$, including16 points at the actual64-bit width, check16,384 four-bit two-shift compositions against independent machine-style operations, and match512 width64 transpose-weight rows to direct routing.

No Rust kernel, full shift proof or CPU performance was executed.

**Remember:** shifts route bits; transpose routing moves their weights; an intermediate index composes two shifts without an intermediate witness word.
