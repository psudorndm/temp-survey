# Lesson 038 — Compute ring-switching efficiently and close public claims

Introduced: 2026-10-23. Understanding unchecked. Implementation: Binius64 at 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** the same ring-switch relation serves two endpoints: a commitment-backed private oracle and a directly known public table. Their final authentication mechanisms differ.

Prerequisites: [packing](035-packed-bits-and-the-linearity-obstacle.md), [row batching](036-ring-switch-transpose-and-fresh-row-batching.md), [tensor indicator](037-tensor-algebra-and-the-ring-switch-indicator.md) and [public/private shift folding](034-segmented-shift-product-and-exceptional-extraction.md).

## 1. Factor the equality weights before folding witness bits

The prover must compute $v_i=\sum_yq_yw(i,y)$ for $128$ bit columns. Split the suffix point into its first $\min(m,7)$ coordinates and the rest:

$$
y=(y_{\mathrm{hi}}\ll m_{\mathrm{lo}})\;|\;y_{\mathrm{lo}},
\qquad
q_y=q_{\mathrm{lo}}[y_{\mathrm{lo}}]\,
q_{\mathrm{hi}}[y_{\mathrm{hi}}].
$$

Then

$$
v_i=\sum_{y_{\mathrm{hi}}}q_{\mathrm{hi}}[y_{\mathrm{hi}}]
\left(\sum_{y_{\mathrm{lo}}}
q_{\mathrm{lo}}[y_{\mathrm{lo}}]w(i,y_{\mathrm{hi}},y_{\mathrm{lo}})
\right).
\tag{1}
$$

[expand_tensor_factors and fold_1b_rows_for_b128_split](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/ring_switch.rs#L95) implement this regrouping. The complete suffix equality tensor is not written merely to consume it again in this fold.

The low factor covers at most $128$ packed witness rows. Group eight weights and precompute all $256$ subset sums. For each bit column, the corresponding eight witness bits form one byte indexing that table. A partial binary transpose extracts these byte masks for all columns.

[RowFoldTables::new and fold_group](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/bit_matrix.rs#L47) show the actual subset-sum and byte-index layout. There are $16$ tables, whose logical field payload is

$$
16\cdot256\cdot16\text{ bytes}=64\text{ KiB}.
$$

They are built once and reused across high blocks. Short low factors have zero trailing weights; unused rows/lanes contribute zero. The field-buffer alignment precondition is checked separately.

Each block's resulting $128$ field column sums is scaled by its high weight. This is $128$ multiplications per block, amortizing to one per input row only for a full $128$-row block. The source's cache-residency and bottleneck remarks are engineering intent, not measurements verified here.

## 2. Build A in one pass, with the operations in the correct order

After row batching, [rs_eq_ind_from_factors](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/ring_switch.rs#L196) writes

$$
A[y]=L_\gamma\left(
q_{\mathrm{lo}}[y_{\mathrm{lo}}]q_{\mathrm{hi}}[y_{\mathrm{hi}}]
\right)
$$

directly into its output buffer. It creates one bytewise binary-linear transformation and reuses it. The factors are multiplied in $K$ **before** bit projection; in general

$$
L_\gamma(ab)\ne L_\gamma(a)L_\gamma(b).
$$

For example $c_0(\alpha^2)=1$ while $c_0(\alpha)c_0(\alpha)=0$ in $\mathbb F_4$. The fused code avoids storing then rereading a full intermediate $q$ table, but still materializes $A$ for the next prover stage.

The transformation-factory API was inspected at this call site; its internal kernel and architecture implementations remain later engineering work.

## 3. Private endpoint: ask the oracle protocol

[The private prover caller](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/prove.rs#L260) removes the segment-selector coordinate, invokes ring-switching on the packed non-public witness, then queues

$$
\sum_y\pi_{\mathrm{private}}[y]A[y]=s'
$$

through prove_oracle_relation.

[The verifier caller](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/verify.rs#L265) supplies verify_oracle_relation with a closure evaluating $\widetilde A(p)$ via eval_rs_eq using the saved suffix $z$ and row weights $\gamma$.

This closes the ring-switch interface, but not the commitment proof. Concrete linear-oracle batching, codeword/Merkle authentication and BaseFold terminal checks are the next stage. Deferred native wiring evaluation also remains a separate obligation; proving a witness query does not itself check the public wiring function.

## 4. Public endpoint: a smaller sumcheck and direct evaluations

The current implementation handles the public segment separately from the private oracle. It does not prove “the public prefix of the hidden oracle” in this helper.

For $n_{\mathrm{pub}}\ge1$ public words, define

$$
m_{\mathrm{pub}}=\max(\lceil\log_2n_{\mathrm{pub}}\rceil-1,0).
$$

The helper pads to $2^{m_{\mathrm{pub}}}$ packed cells, or $2^{m_{\mathrm{pub}}+1}$ words. Its bit-query point is

$$
r_J\;\Vert\;r_Y[0:m_{\mathrm{pub}}+1].
$$

[prove_public_eval](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/ring_switch.rs#L358) sends the public bit-MLE claim, ring-switches it, then runs ordinary degree-two product sumcheck for

$$
s'_{\mathrm{pub}}
=\sum_{y\in\{0,1\}^{m_{\mathrm{pub}}}}
\widetilde\pi_{\mathrm{pub}}(y)\widetilde A_{\mathrm{pub}}(y).
$$

[verify_public_eval](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/reduction.rs#L323) reverses the sumcheck challenges into coordinate order, packs its known public words, evaluates their padded MLE, evaluates the tensor indicator, and checks their product against the terminal sumcheck claim. It does not accept either endpoint factor as an unchecked prover value.

With one or two public words, $m_{\mathrm{pub}}=0$: product sumcheck has no rounds, while ring-switching still samples seven row coordinates. Directly evaluating the public packed MLE can take work linear in the public table; a small proof does not imply logarithmic verifier time.

To return the claim in the shift proof's larger word-index space, the verifier multiplies by

$$
\prod_{j=m_{\mathrm{pub}}+1}^{L-1}(1-r_{Y,j}),
$$

because every word outside the public segment's padded span is zero.

Under ideal fresh uniform challenges and fixed public data, a false local public bit claim can survive this combined reduction with probability at most

$$
\frac{7+2m_{\mathrm{pub}}}{|K|}.
$$

This adds row-batch error and ordinary degree-two sumcheck error. It excludes earlier reductions, transcript instantiation, validation and other protocol events.

## 5. What is and is not established

[verify_lesson038.py](../../work/verify_lesson038.py) checks eight GHASH-sized subset-table/fused-indicator fixtures, 3,686 complete honest toy public sumcheck transcripts and six actual-width public packing/endpoint fixtures. The toy model uses one-bit words and $\mathbb F_4$, preserving the two-words-per-cell indexing structure. The actual-width checks include odd word counts and zero-round public sumchecks.

These are independent algebra/layout checks, not executed Rust tests, cache measurements, proof serialization tests, a complete protocol audit or a zero-knowledge argument. The plain $v$ vector contains witness-dependent information; hiding it requires the separate ZK composition.

**Takeaway:** public data can be authenticated by direct verifier computation; private data require the committed-oracle machinery.

**Recall:** why does the public helper still need a ring-switch consistency check when the verifier already knows the public words?
