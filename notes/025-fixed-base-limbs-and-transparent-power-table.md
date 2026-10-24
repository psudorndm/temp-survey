# Lesson 025 — Four limbs replace 64 fixed-base bit leaves

Introduced: 2026-10-23. Understanding unchecked. Binius64 code: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf. Blueprint source: 05061c6f9eca316b6844396883afb9ae37ff2fe9.

**One idea:** when the exponentiation base is public and fixed, precompute powers for blocks of exponent bits. Multiply four looked-up limb values to obtain a 64-bit exponentiation.

Prerequisites: [integer exponent encoding](019-intmul-exponent-encoding-and-wraparound.md), [product trees](020-bit-selected-powers-and-product-trees.md), [GKR](021-one-gkr-layer-and-line-reduction.md), [Frobenius/MLE twists](023-frobenius-twists-of-mle-claims.md), and [selector outputs](024-selector-batching-and-common-point.md).

## 1. What is fixed, and what still varies

The fixed base is the public field generator $g$. Every constraint row may have a different unsigned integer exponent $e[x]$. Fixed-base does not mean the exponent or output table is constant across rows.

In the pinned implementation,

$$
w=16,\qquad n_{\mathrm{limbs}}=4,\qquad
e=\sum_{j=0}^3 2^{16j}e_j,\qquad 0\leq e_j<2^{16}.
$$

The limb extraction implemented by limb_index is

$$
e_j=\left\lfloor\frac{e}{2^{16j}}\right\rfloor\bmod 2^{16}.
$$

All sums and shifts in this integer decomposition concern ordinary integers. Exponentiation and products below are field operations.

Define four public tables, each with $2^{16}=65{,}536$ rows:

$$
T_j[k]=\left(g^{2^{16j}}\right)^k.
$$

Then the exact arithmetic identity is

$$
g^e
=\prod_{j=0}^3 g^{2^{16j}e_j}
=\prod_{j=0}^3 T_j[e_j].
\tag{1}
$$

The four gathered values become a four-leaf product tree. It has two multiplication layers; the 64-bit variable-base tree from Lesson 020 has six layers and 64 leaves. This comparison is about the tree shapes, not total proof cost: the fixed-base path also needs a lookup proof.

## 2. Tiny two-limb example

Use two-bit limbs only for illustration:

$$
13=1+4\cdot3,\qquad
g^{13}=T_0[1]T_1[3]
=g\cdot(g^4)^3.
$$

One lookup selects the low limb's contribution and one selects the high limb's contribution. The base and limb-position powers are precomputable; the addresses vary with the exponent.

For a complete four-limb toy, take $w=2$, $e=182=\mathtt{0xB6}$, and $g=u$ in $\mathbb F_{16}=\mathbb F_2[u]/(u^4+u+1)$. The low-to-high limbs are

$$
(e_0,e_1,e_2,e_3)=(2,1,3,2),\qquad
182=2+4+48+128.
$$

In raw polynomial coordinates, the leaves and actual high-half pairing are

$$
[4,3,8,5]\longrightarrow[4\cdot8,\;3\cdot5]
=[6,15]\longrightarrow4.
$$

These multiplications are in $\mathbb F_{16}$, not integer multiplication. The root is $g^{182}=g^2$, since this toy field's multiplicative group has order 15. This small-field example checks decomposition and layout; it is not a sound parameter choice for injectively encoding an eight-bit integer.

## 3. Why IntMul has three fixed-base trees and eight tables

The IntMul identity needs these row tables:

$$
\begin{aligned}
A[x]&=g^{a[x]},\\
C_{\mathrm{lo}}[x]&=g^{c_{\mathrm{lo}}[x]},\\
C_{\mathrm{hi}}[x]&=\left(g^{2^{64}}\right)^{c_{\mathrm{hi}}[x]}.
\end{aligned}
$$

The high output word uses a shifted base because its integer contribution is $2^{64}c_{\mathrm{hi}}$. The other input word $b$ is used in the variable-base operation $A[x]^{b[x]}$.

The prover constructs eight tables

$$
T_s[k]=\left(g^{2^{16s}}\right)^k,\qquad 0\leq s<8.
$$

Here $T_s$ extends the notation from equation (1). Table usage is:

| Tree | Four tables used | Root on a Boolean row |
|---|---|---|
| $a$ | $T_0,T_1,T_2,T_3$ | $g^{a[x]}$ |
| $c_{\mathrm{lo}}$ | $T_0,T_1,T_2,T_3$ | $g^{c_{\mathrm{lo}}[x]}$ |
| $c_{\mathrm{hi}}$ | $T_4,T_5,T_6,T_7$ | $g^{2^{64}c_{\mathrm{hi}}[x]}$ |

Thus there are 12 gathered limb columns but eight distinct precomputed power tables. At the default 128-bit field size, the logical scalar payload is $65{,}536\cdot16=1$ MiB per table, or 8 MiB for eight tables. This excludes tree layers, buffers, allocation overhead and other proof state; it is not a measured peak-memory claim.

## 4. Actual witness construction and table generation

Witness::new constructs bases by repeated squaring, taking every sixteenth element:

$$
g,\quad g^{2^{16}},\quad g^{2^{32}},\quad\ldots,\quad g^{2^{112}}.
$$

It allocates the eight table buffers before filling them in parallel. It calls limb_leaves for each family and builds each product tree with LOG_N_LIMBS equal to two.

The leaf buffer has row coordinates low and the two limb-index coordinates high. For $N=2^\ell$ padded rows, limb $j$ and row $x$ sit at scalar offset $jN+x$. The constructor pairs low and high halves, so the first layer combines limbs $(0,2)$ and $(1,3)$, then multiplies those two results.

Absent exponent rows are zero words, which gather table row zero. Since $T_s[0]=1$, padded rows have all-one leaves and root one. Zero-padding the integer word therefore does not mean zero-padding its exponentiation table.

power_table_into uses a sequential prefix and a strided continuation. With packed width $P$, LOG_STRIDE equal to four gives a block of $B=16P$ logical scalars. For a table larger than a block, it generates

$$
1,h,\ldots,h^{B-1}
$$

and retains the increment $h^B$. It extends later packed entries by multiplying the packed entry one block earlier by the broadcast increment. At the scalar level this is

$$
h^{i+B}=h^i h^B.
$$

The source comments explain the intent of exposing independent multiply chains. Actual speed, register allocation, lane execution and allocator behavior have not been benchmarked or validated here. Small tables use the sequential branch instead.

## 5. What phase 4 checks, and where it stops

Phase 3 left claims about the MLEs of $A$, $C_{\mathrm{lo}}$ and $C_{\mathrm{hi}}$. Phase 4 batches their three four-leaf trees using two tree-selector coordinates, with a fourth zero slot for interpolation.

The initial combined claim is

$$
\sum_{t=0}^{2}\operatorname{eq}_2(\sigma,t)s_t,
$$

where $s_t$ are the three root claims. This zero tree slot is separate from the all-one padding of absent exponent rows inside each real tree.

The GKR call runs two layers and returns a point partitioned by the caller as

$$
(\sigma^\star,\;r_{\mathrm{row}}^\star,\;r_{\mathrm{limb}}^\star).
$$

The prover sends four limb-column evaluations per tree, all at the new row point:

$$
v_{t,j}=\widetilde L_{t,j}(r_{\mathrm{row}}^\star).
$$

The verifier reconstructs each stacked leaf-table evaluation and checks the combined result:

$$
\sum_{t=0}^{2}\operatorname{eq}_2(\sigma^\star,t)
\sum_{j=0}^{3}\operatorname{eq}_2(r_{\mathrm{limb}}^\star,j)v_{t,j}
\stackrel{?}{=}s_{\mathrm{GKR}}.
$$

The prover re-gathers each limb column from the corresponding table to calculate these 12 evaluations. This is how the honest prover obtains them; the verifier must still check their lookup semantics later.

Reading the phase4 caller and verifier establishes this interface and receive/recombination check. It does not amount to reviewing every internal branch of the large batched product-tree implementation. A scalar closing equality also does not separately authenticate every sent value.

## 6. Collapse the eight tables to one shared lookup table

Let $\varphi(z)=z^2$. Every table is a Frobenius image of the first:

$$
T_s[k]=\varphi^{16s}(T_0[k]),\qquad T_0[k]=g^k.
$$

For one gathered column, let $k[x]$ be its integer limb index and define

$$
U[x]=T_0[k[x]],\qquad
L[x]=T_s[k[x]]=\varphi^{16s}(U[x]).
$$

The deterministic claim transformation from Lesson 023 gives

$$
\widetilde U\bigl(\varphi^{-16s}(r)\bigr)
=\varphi^{-16s}\bigl(\widetilde L(r)\bigr).
$$

Both the claimed value and every row-point coordinate change. The index column does not change: the same integer address $k[x]$ selects the table row on every Boolean witness row.

In wire-family order $a,c_{\mathrm{lo}},c_{\mathrm{hi}}$, limb_column_twists returns

$$
(0,16,32,48,\;0,16,32,48,\;64,80,96,112).
$$

All 12 columns can therefore read the one shared public table $T_0$. Their row points now differ, and their index columns still refer to different limbs/word families. Reusing one table does not make the 12 looked-up columns identical.

## 7. Why the verifier can evaluate that table without expanding it

For the 16-bit table address $i=\sum_{j=0}^{15}2^j i_j$,

$$
T_0[i]=g^i=\prod_{j=0}^{15}\left(g^{2^j}\right)^{i_j}.
$$

Each factor depends on just one bit. Hence the unique MLE is

$$
\widetilde T_0(Y_0,\ldots,Y_{15})
=\prod_{j=0}^{15}
\left[1+\left(g^{2^j}-1\right)Y_j\right].
\tag{2}
$$

The right side is multilinear and agrees with the table at every Boolean address, proving the identity by uniqueness. This is a formal MLE identity, not exponentiation by a field-valued address.

eval_power_table_mle implements this factorization using repeated squaring of the public base, coordinate scaling and accumulated products. Work is linear in the 16 address variables; the verifier does not materialize 65,536 entries. The source's multiplication-count shorthand should not be read as a complete field-operation cost model.

The phase5 entry passes this evaluator to the transparent logup* routine. The table itself needs no commitment. Other lookup-related oracles can still be committed; “transparent table” does not mean the entire lookup protocol has no commitments.

## 8. The binding obligation that remains

The protocol must tie the looked-up value to the correct limb index of the original word:

$$
U_{t,j}[x]=T_0[e_{t,j}[x]].
$$

Showing that a value appears somewhere in the table would not enforce this address. A prover could choose different valid addresses, build a consistent product tree for those values and leave the original integer operand disconnected.

The lookup path therefore produces index evaluation claims, and later bit-column relations must connect those indices to the actual operand limbs. The phase5 entry was inspected through the transparent lookup call and returned index-claim boundary; the logup* reduction and complete index-binding proof are the next lesson. Parity and final witness/shift authentication also remain necessary.

## 9. Sources and independent checks

- [Blueprint source, §4.5.2 exponentiation subprotocols](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex): uses 64 bit-selector leaves for the fixed-base presentation. The four-limb lookup path above is grounded in the pinned implementation.
- [IntMul common definitions](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/intmul/common.rs): limb constants, limb_column_twists, twist_limb_claim and eval_power_table_mle.
- [Witness construction](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/intmul/witness.rs): Witness::new, power_table_into, sequential_powers, limb_index and limb_leaves.
- [Prover phase4](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/intmul/prove.rs): three-tree batch and re-gathered column evaluations.
- [Verifier phases4/5](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/intmul/verify.rs): closing interpolation, claim twists and transparent-table lookup interface.

Independent scalar checks cover all 256 four-two-bit-limb exponent decompositions; 4,384 small-field table-MLE evaluations; 120 logical strided table shapes/base choices; 32 actual-width low/high exponentiation cases using the pinned GHASH generator; all 12 twisted gathered columns across three real rows and one padded row; row/limb leaf recombination; and two full 65,536-row GHASH table-MLE evaluations against equation (2).

These models check arithmetic, layout and identities. They do not run Rust, a complete phase4/5 transcript, SIMD kernels or performance benchmarks, and do not establish the full proof's security.

**Remember:** four limb lookups compute the fixed-base power; a product tree checks their combination; the lookup/index proof must connect them to the original exponent.

**Recall:** why does an absent zero exponent row contribute one rather than zero to each limb column?

Next: derive the logup* lookup reduction and trace how its index claims bind these gathered powers to the operand bits.
