# Lesson 043 — Authenticate a whole folding coset with one Merkle leaf

Introduced: 2026-10-23. Understanding unchecked. Binius64 revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** BaseFold opens groups of symbols needed by a fold. Merkle authentication binds each group to an earlier root, while a shared internal layer reduces repeated path costs.

Prerequisites: [fold consistency](041-basefold-butterflies-and-the-terminal-check.md) and [virtual lifted queries](042-unequal-oracles-lift-repeat-and-first-fold-routing.md).

## 1. Distinguish a symbol, a leaf and a root

A codeword with $2^L$ field symbols and leaf width $B=2^a$ has $2^{L-a}$ leaves. Leaf $j$ contains the contiguous symbols

$$
c[jB],\ldots,c[jB+B-1].
$$

Initial-oracle leaves hold their interleaved batch cosets. Later FRI leaves hold the coset needed by the next grouped fold. The tree depth is $d=L-a$, not $L$.

The proof transmits a root digest. The verifier attaches the expected depth and leaf width from protocol parameters; it does not accept an arbitrary tree shape merely because the root has the right byte length. [recv_merkle_commitment](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/merkle_channel.rs#L208) constructs that handle.

To open the leaf, the prover supplies its $B$ field values and sibling digests. The verifier hashes the values, uses index bits to order left/right children, and compares the resulting ancestor with an authenticated digest.

## 2. The standard hash suite uses different leaf and parent functions

[StdHashSuite](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/hash/src/lib.rs#L28) selects SHA-256 leaves and Sha256Compression parents.

The leaf function is ordinary SHA-256 over the concatenated serialized field values:

$$
\operatorname{Leaf}(v_0,\ldots,v_{B-1})
=\operatorname{SHA256}
(\operatorname{Ser}(v_0)\Vert\cdots\Vert\operatorname{Ser}(v_{B-1})).
$$

[hash_serialize](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/hash/src/serialization.rs#L73) streams serialization into the hasher. The inspected AArch64 GHASH underlier serializes each field value as16little-endian bytes.

For parents, [Sha256Compression](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/hash/src/sha256.rs#L21) concatenates two32-byte digests and applies **one raw SHA-256 block compression**, with no padding block or length suffix. Its initial state comes from the digest of the tag “BINIUS SHA-256 COMPRESS”, reinterpreted as native32-bit words. The resulting state is emitted in native word order.

Thus the parent function differs from ordinary SHA-256 of the two children. Domain separation distinguishes leaf hashing from internal compression. The native-word convention is material for exact reproduction; our model matches the little-endian pinned known-answer vector. Cross-endian interoperability was not tested.

## 3. Share a verified layer across many openings

Let $q$ queries address a tree of depth $d$. Choose an internal layer at depth $h$ below the root, containing $2^h$ digests.

First send that entire layer and fold it to the root. Then each leaf needs only $d-h$ siblings to reach its entry in the layer. The digest count is

$$
2^h+q(d-h).
\tag{1}
$$

Increasing $h$ by one changes this count by $2^h-q$. Therefore a minimizing choice in this format is

$$
h=\min(d,\lceil\log_2q\rceil).
$$

For depth three and three queries, $h=2$: send four layer digests and one sibling per query, totaling seven digests.

[recv_openings](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/merkle_channel.rs#L220) reads and verifies the layer once, then each leaf and path. Duplicate query indices remain duplicate entries in this format; it does not deduplicate all shared branch nodes. Equation (1) describes this implementation's format, including its layer payload.

## 4. Trace one path

[verify_opening](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/merkle_tree/scheme.rs#L227) starts from the leaf digest. At every level:

$$
D_{\mathrm{next}}=
\begin{cases}
C(D,S),&\text{current index even},\\
C(S,D),&\text{current index odd}.
\end{cases}
$$

Then shift the index right by one bit. After $d-h$ steps, the remaining index selects the claimed layer entry. An incorrect value, sibling, ordering or destination must fail unless the hash assumptions are broken.

Shape/index preconditions are checked by assertions; invalid authentication data return a verification error. These are different failure mechanisms. A malformed-proof robustness review remains separate from deriving valid proof behavior.

## 5. Why terminal vectors have no authentication paths

The terminal codeword is small enough to reveal in full. [recv_committed_vector](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/merkle_channel.rs#L255) reads exactly the length implied by its handle and calls verify_vector. That method rehashes every leaf and folds the whole tree to the earlier root.

No extra sibling advice is needed: the full vector supplies every leaf. Its root was fixed before the final fresh folding/query challenges, so its later revelation does not make it freely adjustable.

The Merkle layer proves membership under a root. It does not establish the original codeword's low degree, the truth of a linear claim, or privacy of opened symbols. Those properties come from the folding/proximity protocol, earlier reductions and the separate ZK composition.

## 6. Representative prover organization

[BinaryMerkleTree](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/hash-prover/src/binary_merkle_tree.rs#L164) stores all leaf digests and internal layers in one allocator-backed buffer, widest layer first and root last. The node count is $2^{d+1}-1$.

It hashes leaves, then recursively builds disjoint subtrees with Rayon. Small subtrees use batched compression without further task splitting; a parent waits for its two children. This explains allocation and dependency structure. No performance benchmark or architecture-kernel validation was run here.

## Validation and recall

[Independent report](../../work/lesson043-validation.json): a standalone raw compression model agrees with normal SHA-256 on five messages and with the pinned custom compression vector. It checks160honest Merkle batches,720rejected mutations and160cap-depth minimizations, including depth-zero trees and repeated indices. Rust and parallel hash kernels were not executed.

**Takeaway:** authenticate the exact coset and tree shape used by the algebraic fold.

**Recall:** what does a valid Merkle opening establish, and what does it leave for FRI to establish?
