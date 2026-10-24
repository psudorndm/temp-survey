# Lesson 032 — Group repeated wiring before doing field arithmetic

Introduced: 2026-10-23. Understanding unchecked. Pinned Binius64 revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** group all references to the same word under the same two-shift sequence, then reuse that group's one field coefficient across the word's bits.

Prerequisites: [five-factor shift relation](031-five-factor-shift-sumcheck-and-witness-closure.md). Next: [implicit shift-weight table](033-implicit-shift-weights-and-sparse-sumcheck-rounds.md).

## 1. A key is a word/shift group, not a cryptographic key

An operand term names a word $y$, inner shift $q_1$ and outer shift $q_2$. Write $q=(q_1,q_2)$. Several constraints and operand columns can refer to the same pair $(y,q)$.

A shift key identifies this group. It holds the shift sequence's dense index and a range of references into a flattened list. The word is implicit: its key range is reached through that word's index.

Let $\eta_m=\operatorname{eq}(r_M,m)$ be an operand weight and $\xi_x=\operatorname{eq}(r_X,x)$ a constraint-row weight. A key's field coefficient is

$$
\kappa_{y,q}
=\sum_{\text{references }(m,x)\text{ in key }(y,q)}\eta_m\xi_x.
$$

The same key spans operations; the operand-column number distinguishes Zero, BitAnd, IntMul and BinMul references. The operation is not a separate part of key identity.

## 2. The first witness-dependent table

For each active sequence $q$, accumulate

$$
g_q[j]=\sum_y\kappa_{y,q}\,w[y]_j.
\tag{1}
$$

This is the Boolean table of the witness-and-wiring factor from Lesson031, after summing the word axis but before binding bit position or shift slots.

Because each $w[y]_j$ is a bit, a key's coefficient is added only at set-bit positions. No general field multiplication is needed merely to multiply it by a zero/one witness bit.

A four-bit example over $\mathbb F_4$ has operand weights $(\alpha,\alpha+1)$ and row weights $(\alpha+1,1)$. If one key references $(m,x)=(0,0),(0,1),(1,0)$, then

$$
\kappa=\alpha((\alpha+1)+1)+(\alpha+1)^2=1.
$$

For word $\mathtt{1010}$, its contribution to the key's low-to-high bit row is $[0,1,0,1]$.

## 3. Why stable counting sort appears in the builder

The source walks operand columns first, constraints second and operand terms third. A builder then:

1. Counts references per word and records which sequence codes each segment uses.
2. Prefix-sums the counts into word ranges.
3. Scatters references into those ranges while preserving their walk order.
4. Groups each word's references by sequence, preserving first-appearance key order and reference order within each key.
5. Flattens keys and corrects chunk-relative key ranges.

This avoids a global comparison sort over all references. It does not mean there is only one pass: counting, scattering, grouping and small sequence-encoding work are separate operations.

The preserved column order lets Key::accumulate_wide add the row weights for one operand column before multiplying by that column's weight. It accumulates the resulting products in a wide representation and reduces after accumulation. The algebra is still the coefficient above.

## 4. Sparse identities and dense IDs are different coordinates

One actual shift slot has $8\cdot64=512$ spellings. A two-slot sequence has a canonical cube address

$$
\operatorname{code}(q)
=(\operatorname{index}(q_2)\ll9)
\mathbin{\vert}\operatorname{index}(q_1).
$$

The outer slot occupies the high bits, matching the sumcheck's binding order.

Each segment also builds a dense list of only its active sequences. DenseShiftEncoding sorts by the sequence's inner/outer ordering and uses binary search to find its dense ID. That dense order need not be increasing canonical cube-address order.

Before folding, the row list must decode dense IDs back into canonical sequence addresses. Treating a dense ID as a hypercube vertex would prove a different polynomial.

The final encoding is small and uses binary search. The builder nevertheless uses temporary direct code-indexed tables over the full $2^{18}$ code space for fast grouping. In particular its u32 ID map has 1 MiB logical payload per instance of that map; this is not a measured total-memory or persistent-cache figure.

## 5. Parallelism and packed masks

The builder partitions word ranges using targets of 4,096 words and 65,536 references. A heavily referenced word remains intact even if it exceeds the reference target. Disjoint chunks fill disjoint slices; keys are assembled independently and their offsets reconciled afterwards.

KeySegment::build_g builds one row per segment-local active sequence. It precomputes masks for packed lanes, broadcasts a key coefficient and selects its contribution according to word bits. Its current specialization requires packed width at most eight.

The byte loop stops after the highest nonzero byte. Internal zero bytes below that position are still visited; the implementation does not skip every zero byte.

Tasks own their accumulators and merge them additively. The source-level strategy is clear; no CPU execution, performance measurement or safety audit of the unsafe indexing was performed.

## 6. Public and hidden segments remain separate

Each segment owns its dense encoding and uses segment-relative word indices. The public and hidden $g$ rows are concatenated as tagged sparse rows. If both segments use the same canonical sequence, that address appears twice.

This is correct because the mathematical table at that address is the sum of its stored rows. Duplicates need not be merged immediately. The later sparse folds preserve this interpretation.

Concrete builder bounds include u16 dense sequence IDs and u32 key/reference ranges. These are data-representation limits, not cryptographic assumptions. Serialization validation and every caller's structural checks are a separate review scope.

## Sources and checks

- [Builder](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/shift/key_collection/builder.rs): reference walk, packed reference, counting/scatter/grouping and collection construction bodies.
- [Key](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/shift/key_collection/key.rs): accumulate_wide/accumulate.
- [KeySegment](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/shift/key_collection/key_segment.rs): word_keys and build_g.
- [Dense encoding](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/shift/key_collection/dense_shift_encoding.rs): construction, dense lookup and canonical shift_indices.
- [Blueprint §4.7.2](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex): witness/shift factorization and sparse index motivation. The multi-pass builder above is the actual pinned implementation.

An independent scalar model checks128 fixtures of stable word grouping and active-row accumulation against a direct term-by-term scan. The worked coefficient is checked over $\mathbb F_4$. These validate algebra and logical layout, not Rust serialization, SIMD masks or the real allocator.

**Remember:** collect references into a reusable coefficient; add that coefficient at the word's set bits.
