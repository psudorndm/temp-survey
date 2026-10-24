# Lesson 033 — Keep the huge weight table implicit

Introduced: 2026-10-23. Understanding unchecked. Code revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** pushing weights through a shift is linear. Fold one small operator table first, then apply the remaining shift only to the sparse rows the circuit actually uses.

Prerequisites: [shift transpose](030-shift-indicators-and-bit-carry-evaluation.md), [sparse keys](032-sparse-shift-keys-and-witness-weighted-rows.md).

## 1. Where the large formal table comes from

Let $d(i)$ be the64 oblong bit weights. The early shift sumcheck is

$$
s=\sum_{q_1,q_2,j}g(j,q_1,q_2)\,h(j,q_1,q_2),
$$

where $g$ is the sparse witness-weighted table and

$$
h(j,q_1,q_2)=\sum_{i,k}d(i)S_{q_2}(i,k)S_{q_1}(k,j).
$$

Its table would have $64\cdot512^2=2^{24}$ field entries: 256 MiB of scalar payload at128 bits per field element, before buffers and other state. The implementation does not materialize that table.

## 2. Apply the same operator twice

For an output weight vector $\psi$, define

$$
T[\psi](j,q)=\sum_k\psi(k)S_q(k,j).
$$

One full operator table has $64\cdot512=2^{15}$ entries. First compute

$$
\eta(k,q_2)=T[d](k,q_2).
$$

At a Boolean inner slot,

$$
h(j,q_1,q_2)=T[\eta(\cdot,q_2)](j,q_1).
$$

The outer slot is bound first. Since $T$ is linear in its weight input,

$$
T[(1-r)\eta_0+r\eta_1]
=(1-r)T[\eta_0]+rT[\eta_1].
$$

Therefore OuterShiftStage can fold $\eta$ and derive an $h$ row on demand instead of folding the full $h$ table.

While the inner slot is Boolean, that derivation is a literal transpose shift of64 weights, with sign-fill accumulation where required. Reversing the two slots would change the composition, and would also lose this cheap fixed-inner-shift access pattern.

## 3. A sparse round needs no matched pair of stored rows

Write the product round as

$$
R(t)=\sum_vG_t(v)H_t(v).
$$

The two nontrivial samples are

$$
R(1)=\sum_vG_1(v)H_1(v),\qquad
R(\infty)=\sum_v(G_0+G_1)(v)(H_0+H_1)(v).
$$

Here infinity means the coefficient of $t^2$. If a stored $G$ row at label $a$ has values $c$, its contributions are:

- To $R(1)$: $\langle c,H[a]\rangle$ only if $a$ is in the upper half.
- To $R(\infty)$: $\langle c,H[a]+H[a\mathbin{\oplus}\mathrm{half}]\rangle$ from either half.

Every contribution is linear in $c$. Facing $G$ rows never need to be paired or even both present. Repeated labels also work additively.

The claim supplies $R(0)=s-R(1)$. The quadratic coefficients follow from these samples; this ordinary-sumcheck interpolation uses no division by an equality coordinate.

## 4. Sparse folding is just weighted relabeling

For a stored row $(a,c)$, binding the highest label bit to $r$ does:

$$
(a,c)\longmapsto
\begin{cases}
(a,(1-r)c),&a\text{ in lower half},\\
(a\mathbin{\oplus}\mathrm{half},rc),&a\text{ in upper half}.
\end{cases}
$$

The list length does not shrink. Rows may collide, but their sum is exactly the dense polynomial fold. After all slot coordinates are bound, the rows are finally added into one dense64-entry bit table.

It is incorrect to describe this optimization as halving the number of sparse rows each round. It halves the logical address space while retaining the row list.

## 5. The actual stages

The first24 rounds comprise nine outer-slot rounds, nine inner-slot rounds and six witness-bit rounds:

1. OuterShiftStage keeps folded $\eta$, deriving own/facing $h$ rows into reusable scratch arrays.
2. After the outer slot is fixed, its64 weights $\psi$ are retained.
3. shift_operator_table builds the inner-slot table $T[\psi]$.
4. Phase1SumcheckProver handles inner-slot coordinates using sparse rows and dense weights.
5. After those nine coordinates are fixed, both inputs are dense64-entry bit tables and the standard bivariate-product prover takes over.

One $2^{15}$ table has512 KiB logical scalar payload at128-bit field width. This is a table-size comparison, not a peak-memory benchmark: allocation capacity and overlapping buffers may persist.

## 6. Expose the two remaining bit indices without dividing

The outer contraction still represents a sum over intermediate/output bit indices. The top-level prover runs ShiftIndSumcheck twice: first over $K$, then over $I$.

For a generic run, the claim has the form

$$
\beta=G\sum_i w_i a_i.
$$

The prover runs a dense product sumcheck on columns $Gw$ and $a$. At the endpoint it has $(G\widetilde w(r),\widetilde a(r))$. It separately reevaluates the original unscaled weights to get $\widetilde w(r)$ rather than dividing by $G$.

This preserves correctness even when $G=0$. Each run returns weight and indicator evaluations separately so the next stage can carry the correct scalar product.

The assembly/new/prove bodies were inspected. The specialized partial_eval helper recurrences below that interface were retained but are not all derived in this note.

## Sources and checks

- [Outer stage](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/shift/outer.rs): eta layout, linear fold, on-demand own/facing rows.
- [Phase1](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/shift/phase_1.rs): SparseShiftRows fold/round_coeffs/run_phase_1_sumcheck and Phase1SumcheckProver execute/fold/finish.
- [Operator table](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/shift/monster.rs): shift_operator_row/table.
- [Remaining bit checks](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/shift/shift_ind.rs): ShiftChallengePoint indicator assembly and ShiftIndSumcheck.
- [Blueprint §4.7.2](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex): factorization and successive operator tables.

A four-bit/two-variant model checks16 fixtures with duplicate sparse labels. All96 sparse slot rounds match dense product-round evaluation at every point of $\mathbb F_{16}$, and all48 outer folds match pushing the folded small table through the fixed inner operator. The miniature deliberately constructs the full dense table only as a reference oracle; production does not.

No Rust, packed execution, allocator behavior or speed measurement was performed.

**Remember:** fold the small operator, keep sparse rows additive, become dense only after the slot axes are gone.
