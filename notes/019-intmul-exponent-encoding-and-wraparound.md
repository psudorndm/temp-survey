# Lesson 019 — IntMul through exponents, and the wraparound correction

Introduced: 2026-10-23. Understanding unchecked. Code revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf; Blueprint revision: 05061c6f9eca316b6844396883afb9ae37ff2fe9.

**One idea:** move unsigned integers into exponents of a full-order field generator. The resulting field identity checks the product modulo the group order; a low-bit condition removes the only endpoint ambiguity at the default field size.

Prerequisites: [binary fields](001-binary-field-arithmetic.md), [constraint language](009-binius64-constraint-language.md), and [sumcheck/zerocheck](008-zerocheck-and-mle-check.md). This begins the IntMul implementation track. Full exponentiation proofs are subsequent lessons.

## 1. Three distinct arithmetic layers

One native IntMul row asserts

$$
ab=2^{64}c_{\mathrm{hi}}+c_{\mathrm{lo}}.
$$

where all four symbols are unsigned64-bit integers and the product is the full128-bit product. The stored operand order is [a,b,c_lo,c_hi]. This is not the GHASH multiplication constraint BinMul.

Simply interpreting the words as binary-field coordinates and multiplying them gives carryless polynomial multiplication, not integer multiplication with carries. Interpreting an integer by repeatedly adding the field's1 is also insufficient: in characteristic two it retains only parity.

Instead, use the **multiplicative group** of $K=\mathbb{F}_{2^{\lambda }}$. Its elements are the nonzero field elements; its order is

$$
M=|K^*|=2^\lambda-1.
$$

The integers in the exponent, the field elements being exponentiated, and the bit-valued witness coordinates remain separate objects.

## 2. What a generator buys us

Choose a fixed public multiplicative generator g: its successive powers cycle through every nonzero element before repeating. Thus g has order exactlyM, and

$$
g^u=g^v\quad\Longleftrightarrow\quad u\equiv v\pmod M.
$$

To see the forward direction, divide u−v byM. Only a zero remainder can give the identity, since the powers $g^{0},\ldots ,g^{M-1}$ are distinct. The converse follows from $g^M=1$.

It is essential that g have full order. An arbitrary nonzero element may live in a smaller subgroup and lose more information. This encoding requires forward exponentiation and algebraic proof machinery; it does not require computing a discrete logarithm or assume discrete-log hardness for this identity.

The variable base $g^a$ used below need not itself generate the group. The global equivalence uses the full-order original g.

## 3. Turn the integer product into a field identity

Integer multiplication in the exponent becomes repeated exponentiation:

$$
(g^a)^b=g^{ab}.
$$

The claimed high and low words produce

$$
(g^{2^{64}})^{c_{\mathrm{hi}}}\,g^{c_{\mathrm{lo}}}=g^{2^{64}c_{\mathrm{hi}}+c_{\mathrm{lo}}}.
$$

So check

$$
(g^a)^b=(g^{2^{64}})^{c_{\mathrm{hi}}}\,g^{c_{\mathrm{lo}}}.\tag{1}
$$

All multiplication outside the exponents is field multiplication. All exponent products/sums in this derivation are ordinary integer arithmetic. Equation(1) is equivalent to the desired integer equality **moduloM**, before applying range information.

The high-half base $g^{2^{64}}$ is computed by64 repeated field squarings. It is not the field element whose raw representation is the integer $2^{64}$.

## 4. A full two-bit example

Shrink to two-bit input words and $K=\mathbb{F}_{16}=\mathbb{F}_{2}[u]/(u^{4}+u+1)$. Take the full-order generator $g=u$, whose order is15. Let $a=2$, $b=3$. The ordinary product is6, so the two-bit output halves are $c_{\mathrm{hi}}=1$ and $c_{\mathrm{lo}}=2$:

$$
2\cdot3=4\cdot1+2.
$$

The field check becomes

$$
(g^2)^3=(g^4)^1g^2=g^6.
$$

Both sides equal $u^{3}+u^{2}$ (raw polynomial coordinates0xc). This equality was computed using field arithmetic, while 6 was the integer exponent.

The field identity can hold for a wrong claimed product too. Let $a=0$, $b=2$, but claim $c_{\mathrm{hi}}=c_{\mathrm{lo}}=3$, so $C=15$. Then

$$
(g^0)^2=1,\qquad(g^4)^3g^3=g^{15}=1.
$$

It accepts the same residue as zero, despite $0\neq 15$ as integers.

## 5. Why there is only one ambiguity at 128 bits

Return to64-bit inputs. Their product P and candidate full output C obey

$$
\begin{aligned}0\leq P&\leq(2^{64}-1)^2=2^{128}-2^{65}+1<2^{128}-1,\\0\leq C&=2^{64}c_{\mathrm{hi}}+c_{\mathrm{lo}}\leq2^{128}-1.\end{aligned}
$$

At the default $\lambda =128$, the group order is $M=2^{128}-1$. Therefore $P\in [0,M-1]$ and $C\in [0,M]$. If $C\equiv P$ modM, either

1. $C=P$, the desired equality; or
2. $P=0$ and $C=M$, the sole extra endpoint.

There is no other multiple ofM that fits the difference range. In the second case $a=0$ or $b=0$ and both output words are all ones. The false output has least significant bit1, while the actual product is even.

The exact correction is one bit per row:

$$
a_0b_0=c_{\mathrm{lo},0}.\tag{2}
$$

Here $a_{0},b_{0},c_{\mathrm{lo},0}$ are the least significant bits, embedded as field0 or1. An integer product is odd exactly when both inputs are odd. Every correct product satisfies(2); the sole false endpoint violates it. Equations(1) and(2), together with the specified word ranges and full-order g, are therefore equivalent to the native IntMul relation.

For $\lambda >128$, the whole candidate output range is strictly below the group order, so the exponent identity alone has no wraparound alias. The inspected generic verifier nevertheless includes the parity subcheck; it does not branch it away for a larger field.

Parity alone is far from sufficient. Its power here comes from the range proof showing that only one false case survives the group identity. Likewise, parity cannot generally repair a non-generator: in the $\mathbb{F}_{16}$ example $h=g^{3}$ has order5, and the false product0 versus candidate10 passes both $h^{0}=h^{10}$ and even parity.

## 6. What the implementation actually checks

The pinned verifier requires a binary field with at least128 bits. The standard configuration uses GHASH128. Its field declaration supplies the multiplicative generator

$$
\mathtt{0x494ef99794d5244f9152df59d87a9186}
$$

in GHASH polynomial coordinates, with modulus $X^{128}+X^{7}+X^{2}+X+1$. This generator is not inferred from the fact that a nonzero field element exists: the declaration's second macro argument becomes Field::MULTIPLICATIVE_GENERATOR, and the witness builder explicitly uses that constant.

Parity is enforced across all Boolean rows by a weighted zerocheck, batched with other checks. Let $\widetilde{a}_{0},\widetilde{b}_{0},\widetilde{c}_{\mathrm{lo},0}$ be the row MLEs of the low bits. The initial parity claim is

$$
\sum_{x\in\{0,1\}^\ell}\operatorname{eq}(r_b,x)\bigl(\widetilde a_0(x)\widetilde b_0(x)-\widetilde c_{\mathrm{lo},0}(x)\bigr)=0.
$$

The prover builds low-bit columns with bit index0 and initial claim0. In verify_phase_5, the parity contribution to the closing identity is

$$
\operatorname{eq}(r_b,r_{\mathrm{out}})\bigl(a_{\mathrm{evals}}[0]b_{\mathrm{evals}}[0]-c_{\mathrm{lo\_evals}}[0]\bigr).
$$

This is compared as one term of the batch against the sumcheck's terminal target. The verifier does **not** separately require the parenthesized residual to be zero at the random off-cube output point. Valid Boolean constraints can have nonzero composition evaluations off the cube, as Lesson008 explained. Subsequent witness authentication is still necessary for the sent bit-column evaluations.

The prover sends final per-bit arrays on the wire in the order a,c_lo,c_hi,b in this phase. IntMulOutput rearranges the named fields, and operand_claims supplies [a,b,c_lo,c_hi] to re-randomization. Distinguish wire order from the downstream operand order.

## 7. Exponentiate on the rows, then interpolate

Define the exponentiation table $A[x]=g^{a[x]}$ (with integer $a[x]$) at Boolean row indices x, then take its MLE $\widetilde{A}$. At a random row point r, $\widetilde{A}(r)$ is not obtained by interpreting $\widetilde{a}(r)$ as a new integer exponent and computing g to that exponent.

Tiny counterexample in $\mathbb{F}_{4}$: $g=\alpha$ and integer input rows $a=[0,1]$. Then A=$[1,\alpha ]$ and

$$
\widetilde A(X)=1+(\alpha+1)X,\qquad\widetilde A(\alpha)=0.
$$

Each Boolean-row A value is nonzero, but its MLE can be zero off the cube. The raw-coordinate interpretation of $\widetilde{a}(\alpha )=\alpha$ as integer2 would give $g^{2}=\alpha +1$, a different value. This is why the protocol proves identities through correctly interpolated tables and sumchecks. Notation resembling $g^{\widetilde{a}(X)}$ in overview comments must not be read as that off-cube equality.

## 8. Code/spec boundary for the coming lessons

The algebra above matches the Blueprint's multiplying-in-the-exponent argument. Its fixed-base proof presentation uses64 individual bit selectors. The inspected code takes a different fixed-base implementation route:

- $\mathtt{LOG\_N\_LIMBS}=2$ gives4 limbs of16 bits per exponent word.
- a,c_lo,c_hi yield12 looked-up limb columns.
- The witness builder forms Frobenius-related power tables and product trees over the limbs.
- The verifier's phase5 uses a transparent logup* lookup, point switching of b and the parity zerocheck in its final unification.

This is a material implementation difference. Do not present the Blueprint's fixed-base64-leaf tree as the exact current code path. The variable-base b exponentiation still uses64 per-bit leaves. This lesson inspects the encoding, constants, selected witness construction and parity closure; the GKR, Frobenius and lookup mechanisms are not yet fully derived or validated.

## 9. Evidence and verification

Primary sources at the pinned revisions:

- [Blueprint source](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex), §4.5.1 Multiplying in the Exponent and wraparound argument, plus the exponentiation-subprotocol outline.
- [Native word arithmetic](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/core/src/word.rs) and [constraint definition](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/core/src/constraint_system/constraint.rs), previously inspected in Lesson009.
- [IntMul verifier](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/intmul/verify.rs): field-width requirement, main phase call order and verify_phase_5 parity/wire-order lines.
- [IntMul prover](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/intmul/prove.rs): low-bit extraction and zero-initialized overflow/parity MLE-check.
- [IntMul common definitions](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/intmul/common.rs): limb constants, output fields and operand order.
- [Witness builder](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/intmul/witness.rs): public generator, repeated-squaring table bases and limb-tree construction.
- [GHASH declaration](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/field/src/fields/ghash.rs) and [binary-field macro](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/field/src/binary_field.rs): actual generator value and its assignment to MULTIPLICATIVE_GENERATOR.

Independent scalar verification checked4368 exhaustive input/candidate cases for word widths1,2,3 in corresponding binary fields; every false alias was a zero product versus the maximal candidate, and parity eliminated all of them. Also checked the non-generator counterexample and wider-field case.

For the actual GHASH parameters, verified irreducibility using the degree 128 Rabin criterion, factored M exactly as

$$
3\cdot5\cdot17\cdot257\cdot641\cdot65537\cdot274177\cdot6700417\cdot67280421310721.
$$

verified every listed factor is prime by trial division, and checked $g^M=1$ and $g^{M/p}\neq 1$ for each prime factor p. This proves the declared element has full multiplicative order in the modeled field. Checked45 full-width edge/candidate pairs, the high-half base from64 squarings, and the off-cube interpolation counterexample.

These are independent mathematical checks, not execution of the Rust field kernels or IntMul proof. The full lookup, GKR, channel and downstream authentication remain future work; retaining the source files is not a whole-file audit.

**Remember:** exponentiation checks the product modulo the field group's order; the exact word ranges plus one low-bit check recover the full integer relation.

Next: variable-base exponentiation as a product of64 bit-selected powers, organized as a depth-six product tree.
