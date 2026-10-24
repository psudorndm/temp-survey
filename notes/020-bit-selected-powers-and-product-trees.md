# Lesson 020 — exponentiation as bit-selected powers and a product tree

Introduced: 2026-10-23. Understanding unchecked. Pinned code: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf. Pinned Blueprint: 05061c6f9eca316b6844396883afb9ae37ff2fe9.

**One idea:** expand an integer exponent into bits; each bit chooses a power or1. Their product can be arranged into a tree of quadratic relations.

Prerequisite: [Lesson019](019-intmul-exponent-encoding-and-wraparound.md). This lesson concerns the **variable-base** branch of IntMul. The actual fixed-base branch uses the limb/lookup path identified there.

## 1. Expand the exponent before doing field arithmetic

For a64-bit integer b,

$$
b=\sum_{i=0}^{63}2^i b_i,\qquad b_i\in\{0,1\}.
$$

For a field base $f\neq 0$, exponent rules give

$$
f^b=\prod_{i=0}^{63}\bigl(f^{2^i}\bigr)^{b_i}.
$$

The exponent sum is ordinary integer arithmetic; the product is field multiplication. Every factor is either1 or $f^{2^{i}}$. Define the polynomial selector

$$
\operatorname{sel}(s,v)=1+(v-1)s.
$$

Then $\operatorname{sel}(0,v)=1$ and $\operatorname{sel}(1,v)=v$, so a leaf is

$$
L_i=\operatorname{sel}(b_i,f^{2^i}).
$$

The powers $f,f^{2},f^{4},\ldots ,f^{2^{63}}$ come from repeated squaring. The mathematical list needs63 transitions. Each source leaf-building loop writes a square after every iteration, including the final iteration; whether a compiler removes the final unused square has not been checked.

For IntMul, the base varies by constraint row: $f[x]=g^{a[x]}$ (with integer $a[x]$). The exponent is the word b[x]. This constructs the left side $(g^a)^b$ of the previous lesson's encoding.

## 2. Four-bit worked example

Take $b=13=1101_{2}$. In least-significant-bit order its bits are [1,0,1,1], giving

$$
[L_0,L_1,L_2,L_3]=[f,1,f^4,f^8].
$$

Their product is $f^{1+4+8}=f^{13}$. In the reference tree's actual pairing order, first combine corresponding entries of the low and high halves:

| Stage | Values |
|---|---|
| Leaves | $[f,1,f^{4},f^{8}]$ |
| Pair(0,2), pair(1,3) | $[f^{5},f^{8}]$ |
| Root | $[f^{13}]$ |

For a numerical check, choose $f=u$ in $\mathbb{F}_{16}$ with $u^{4}=u+1$. In raw polynomial coordinates the buffers are

$$
[2,1,3,5]\longrightarrow[6,5]\longrightarrow[13].
$$

These raw numbers represent field polynomials, not integer products. For instance raw2·raw3 gives raw6 in this example because $u(u+1)=u^{2}+u$; all products use the $\mathbb{F}_{16}$ modulus.

The exponent bit zero chooses1 because1 is the multiplicative identity. Choosing0 instead would force the whole product to zero whenever any exponent bit was zero.

## 3. Sixty-four leaves, six levels

For64 exponent bits, the layer widths are

$$
64\longrightarrow32\longrightarrow16\longrightarrow8\longrightarrow4\longrightarrow2\longrightarrow1.
$$

There are six multiplication levels and 63 multiplication nodes per row. Depth is logarithmic; total tree-building work is linear in the number of leaves.

With $N=2^\ell$ padded rows, each tree level is itself a table over rows and the remaining node index. The initial table has64N entries, and the root table hasN entries. Each internal relation is simply

$$
\operatorname{parent}[x,\operatorname{node}]=\operatorname{child}_0[x,\operatorname{node}]\operatorname{child}_1[x,\operatorname{node}].
$$

on Boolean row/node indices. GKR turns a claim about a parent layer's MLE into claims about its children. Thus the large exponent is replaced by a succession of low-degree polynomial checks. The full layer reduction, including equality weights and line challenges, is next lesson.

“Six levels” does not mean six sumcheck rounds. The inspected single-tree verifier calls six factored degree-two MLE-checks on $\ell ,\ell +1,\ldots ,\ell +5$ variables, for $6\ell+15$ sumcheck rounds, plus one child-combining line challenge per level. This count is for that product-tree reduction, not the entire IntMul proof.

## 4. Exact layout and pairing in the implementation

compute_b_leaves stores all rows of bit0, followed by all rows of bit1, and so on:

$$
\operatorname{offset}(i,x)=iN+x.
$$

Therefore row coordinates are the low index bits; the six leaf-index coordinates are high. The physical packed representation groups field elements into lanes, but the scalar indexing contract remains this one.

During this product check, i is represented by six Boolean leaf-index coordinates. Later IntMul outputs64 per-bit claims, which are folded against the shared univariate bit-axis point in Lesson017. Keep these representations of the bit index distinct.

ProdcheckProver::new splits the whole previous layer into equal low/high halves and multiplies corresponding entries. At the bottom level, this pairs bit-index i with i+32 for each row. It removes the highest remaining leaf-index coordinate. The next level pairs the remaining index across a distance16, then8,4,2,1. Adjacent-pair trees have the same final product by commutativity, but different intermediate tables; they are not the exact buffers this implementation builds.

For two rows with bases $[u,u^{2}]$ and four-bit exponents [13,6], the flattened buffers are

$$
\begin{aligned}\text{leaves: }&[2,1,\;1,3,\;3,5,\;5,1],\\\text{next: }&[6,5,\;5,3],\\\text{roots: }&[13,15].\end{aligned}
$$

At any depth d, a remaining index j labels the product of original leaves whose low6−d index bits equal j. This gives an independent mathematical description of every node and of the layout.

## 5. Scalar and packed leaf construction

The scalar fallback loops over rows and then exponent bits. It keeps the current squared base, extracts one exponent bit, constructs a mask and writes

$$
1+\operatorname{select\_mask}(\operatorname{base}-1,\operatorname{bit}).
$$

For a Boolean mask this matches the selector formula: choose base−1 or0, then add1. The packed path performs the same selection per lane and squares packed bases while filling a strided output view. Its dispatch condition is `P::LOG_WIDTH <= n_vars`; otherwise the scalar fallback is used. These source-level branches are not benchmarks of when parallel execution is beneficial on a particular machine.

The mask helper's machine implementation and memory safety invariants have not been independently traced in this lesson. Mathematical agreement of selectors is not validation of the packed backend.

## 6. Padding and retained buffers

Rows past the real exponent-column length have exponent word0. Every leaf on those rows is1 and their root is1. In the full IntMul construction, a padded a word is also0, so its exponentiation base is $g^0=1$. Padding the original integer words with0 is therefore compatible with padding exponentiation tables by1.

This neutral product padding differs from the zero-filled residual arrays in Zero/BitAnd. Padding choices follow the operation being represented.

The witness builder retains the original b_leaves and gives a copied buffer to ProdcheckProver. Source comments explain that prodcheck folds its working leaf layer in place, while IntMul phase1 later needs the original leaves to evaluate their64 per-bit sections. The constructor stores layers from the leaves down to the two-child layer, returning the root separately.

For a64-leaf tree, the stored layers have126N logical scalar entries (64+32+16+8+4+2 timesN), and the returned root hasN. These are logical counts, not a physical memory measurement: allocation, packed-lane minimums, retained copies and other trees add their own storage.

## 7. Boolean selector identity versus off-cube MLE

The statement $L_{i}(x)=\operatorname{sel}(b_{i}[x],f[x]^{2^{i}})$ defines the **Boolean row table**. After interpolation, the leaf MLE generally does not equal the same selector expression applied to independently interpolated inputs.

Counterexample in $\mathbb{F}_{4}$, with $\alpha ^{2}=\alpha +1$: take two rows $(f,b_0)=(1,0)$ and $(\alpha,1)$. The leaf values are$[1,\alpha ]$, so

$$
\widetilde L_0(X)=1+(\alpha+1)X,\qquad\widetilde L_0(\alpha)=0.
$$

But $\widetilde{f}(\alpha )=0$ and $\widetilde{b}_{0}(\alpha )=\alpha$, giving

$$
\operatorname{sel}\bigl(\widetilde b_0(\alpha),\widetilde f(\alpha)\bigr)=1+(0-1)\alpha=\alpha+1.
$$

The two values differ. The product/selectors must be handled through the proper sumcheck and leaf-discharge arguments; they cannot be replaced by that off-cube identity. At a non-Boolean s, sel(s,v) is a field polynomial value, not a Boolean branch choice. For example $\operatorname{sel}(\alpha ,\alpha )=0$ in $\mathbb{F}_{4}$.

The variable-base bit leaves still require proof that they came from the claimed base and exponent bits. Product-tree consistency alone authenticates neither those leaves nor the original witness. That link is the purpose of subsequent Frobenius/selector reductions and downstream witness checks.

## 8. Proof boundary and code sources

Primary sources at the pinned revisions:

- [Blueprint source](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex), §4.5.2 Exponentiation Subprotocols: bit selection, product-tree relations and GKR outline.
- [IntMul witness.rs](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/intmul/witness.rs): compute_b_leaves and compute_b_leaves_parallel; Witness::new uses a_root as the base and calls ProdcheckProver::new with `Word::LOG_BITS = 6`.
- [Prover prodcheck/mod.rs](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip-prover/src/prodcheck/mod.rs): new builds half-wise products and retains layers; layer_prover splits a child layer; prove proceeds from roots toward leaves. Its much larger batch/unequal-depth routines are retained but not reviewed here.
- [Verifier prodcheck.rs](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip/src/prodcheck.rs): each level runs a degree-two MLE-check, receives two child evaluations, checks their product, samples a line challenge and appends its coordinate. The detailed soundness/line reduction is deferred to Lesson021.
- [IntMul verify_phase_1](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/intmul/verify.rs): calls prodcheck::verify(6,…), then receives64 leaf evaluations at the final row point and checks their six-variable multilinear interpolation against the folded leaf claim. These are64 scalars, not the full64N table; later reductions bind their meaning.

Independent $\mathbb{F}_{16}$ checks covered all240 nonzero-base/four-bit-exponent pairs against direct repeated multiplication, every buffer in the worked examples, and four64-leaf padded row shapes. Every intermediate node was compared to a separately defined product over its original high-index leaves; checked the logical layer counts, neutral padding and off-cube counterexamples. No Binius Rust, packed kernels, allocator or proof execution was performed.

**Remember:** a zero-valued bit selects1; a one-valued bit selects its squared power. Multiply the leaves, then prove the tree relations.

Next: one GKR product-tree layer, from a parent evaluation claim through MLE-check to two child evaluations, then their random line combination.
