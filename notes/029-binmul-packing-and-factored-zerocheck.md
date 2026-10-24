# Lesson 029 — BinMul is multiplication in the proof field

Introduced: 2026-10-23. Understanding unchecked. Code revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf. Blueprint revision: 05061c6f9eca316b6844396883afb9ae37ff2fe9.

**One idea:** BinMul operands already represent GHASH field elements. Pack their bits linearly, then check one quadratic field relation per row.

Prerequisites: [binary field arithmetic](001-binary-field-arithmetic.md), [zerocheck/MLE-check](008-zerocheck-and-mle-check.md), and [native constraints](009-binius64-constraint-language.md).

## 1. Different multiplication, different reduction

IntMul asserts ordinary integer multiplication, including carries. It needs the exponent-encoding machinery from Lessons 019–028.

BinMul asserts multiplication modulo the GHASH polynomial

$$
Z^{128}+Z^7+Z^2+Z+1.
$$

Its inputs are binary polynomial coefficients. Addition is XOR, multiplication is carryless polynomial multiplication followed by reduction. Even on the same bit patterns, this differs from ordinary integer multiplication and bitwise AND.

For a small-field reminder, in $\mathbb F_4$ with $\alpha^2=\alpha+1$, raw pattern $3$ represents $\alpha+1$ and its square is raw pattern $2$. Integer $3\cdot3=9$, while $3\mathbin{\&}3=3$.

## 2. Two words carry one field element

At the default $K=\mathbb F_{2^{128}}$, let $\beta_i=Z^i$ in the GHASH polynomial basis. A pair of64-bit words gives

$$
\operatorname{pack}(z_{\mathrm{lo}},z_{\mathrm{hi}})
=\sum_{i=0}^{63}\beta_i z_{\mathrm{lo},i}
+\sum_{i=0}^{63}\beta_{64+i}z_{\mathrm{hi},i}.
$$

The high half uses basis positions64–127; it is not separately added with the low basis reset. This differs from a16-bit lookup address, whose basis resets for each independent limb.

Three such pairs give row tables $A,B,C$ with the desired relation $A[x]B[x]=C[x]$. Packing commutes with MLE interpolation because it is linear:

$$
\widetilde A(r)
=\sum_{i=0}^{63}\beta_i\widetilde a_{\mathrm{lo},i}(r)
+\sum_{i=0}^{63}\beta_{64+i}\widetilde a_{\mathrm{hi},i}(r).
\tag{1}
$$

The same holds for $B,C$. The Blueprint discusses compatible larger-field embeddings; the inspected constraint-reduction caller uses the default128-bit field. Do not instantiate the generic code with an unrelated field representation and assume it still proves GHASH multiplication.

## 3. Randomly test the residual table

Define the composition

$$
G(X)=\widetilde A(X)\widetilde B(X)-\widetilde C(X).
$$

It has individual row-variable degree at most two. Let $E$ be the MLE of the Boolean residual table $E[x]=G(x)$. The field relation holds everywhere exactly when $E$ is identically zero.

The verifier samples $\tau\in K^\ell$ and checks

$$
0=E(\tau)
=\sum_{x\in\{0,1\}^\ell}
\operatorname{eq}_\ell(\tau,x)G(x).
$$

For a fixed nonzero residual table, $E$ is a nonzero polynomial of total degree at most $\ell$, so this initial random test has error at most $\ell/|K|$.

$E$ and $G$ agree on Boolean points but generally differ off the cube. Their distinction matters at the endpoint of the proof.

## 4. Factored MLE-check closes on the composition

The actual code calls degree-two factored MLE-check, rather than ordinary degree-three sumcheck on the entire equality-weighted summand. Its internal round handling removes and tracks equality factors.

After $\ell$ rounds, with challenges reversed into coordinate order, it supplies a terminal prime target $t$ at a new point $r$. The verifier checks

$$
t=\widetilde A(r)\widetilde B(r)-\widetilde C(r).
$$

There is no additional explicit multiplier $\operatorname{eq}(\tau,r)$ in this closing check: the factored reduction already accounts for it. Adding one would mix the ordinary-sumcheck and factored-MLE-check conventions.

Under later terminal-column authentication and fresh uniform challenges, the local factored reduction contributes at most $2\ell/|K|$. Together with the initial residual test, $3\ell/|K|$ is a conservative isolated reduction bound. It is not a full proof-system or Fiat–Shamir security claim.

## 5. Valid rows can have a nonzero terminal target

Over $\mathbb F_4$, take

$$
A=[1,\alpha],\qquad B=[\alpha,1],\qquad C=[\alpha,\alpha].
$$

Both rows multiply correctly. Their MLEs give

$$
G(X)=\alpha(X+X^2).
$$

Thus $G(0)=G(1)=0$, while $G(\alpha)=\alpha$. The residual-table MLE $E$ is zero everywhere, but the composition at the terminal point need not be zero.

An honest one-variable prime round is exactly $\alpha T+\alpha T^2$, and the verifier should accept terminal target $\alpha$ if the new point is $\alpha$. Requiring the composition itself to vanish there would reject valid constraints.

## 6. Six raw word arrays authenticate the packed endpoint

The prover sends64 scalar bit-column evaluations for each word, in order

$$
[a_{\mathrm{lo}},a_{\mathrm{hi}},
b_{\mathrm{lo}},b_{\mathrm{hi}},
c_{\mathrm{lo}},c_{\mathrm{hi}}].
$$

The verifier recombines them using equation (1) to obtain $\alpha_A,\alpha_B,\alpha_C$, then asserts

$$
\alpha_A\alpha_B-\alpha_C-t=0.
$$

The reduction outputs the same six arrays at the common point. Their operand_claims order matches the wire order. Each scalar is a field-valued MLE evaluation, not generally a Boolean bit at the random point. Later rerandomization and shift/witness authentication must bind these claims to the actual operands.

This local subprotocol introduces no auxiliary committed oracle; that does not remove its reliance on the surrounding witness proof.

## 7. Inspected prover engineering

build_table combines each low/high word pair into a128-bit underlier. It allocates at the padded hypercube size, packs complete SIMD lanes from real rows, fills a partial tail with zeros, then zero-fills the remaining packed buffer.

The three packed tables use zero padding, so absent rows satisfy $0\cdot0=0$. This differs from the all-one exponentiation padding of IntMul.

The quadratic prover evaluates the finite selector composition as $AB-C$, and its infinity evaluator as $AB$, dropping the lower-degree $C$ term. After MLE-check, one WordAxisFolder is built for the common row point and used across the six word columns to compute raw bit evaluations. The selected source bodies show parallel orchestration; machine kernels and performance were not executed or benchmarked.

## Sources and validation

- [Blueprint source, §4.6](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex): packing, zerocheck and prover overview.
- [BinMul verifier](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/binmul.rs): full non-test body, basis recombination, terminal assertion and output interface.
- [BinMul prover](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/binmul.rs): build_table and prove bodies.
- [Native BinMul relation](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/core/src/constraint_system/constraint.rs): direct GHASH relation on paired word operands.

Independent checks cover4,096 honest one-variable cases over $\mathbb F_4$, including omitted-constant recovery at old coordinates zero and one;3,840 nonzero residual-vector fixtures obey the initial affine root bound. There are1,152 honest cases with nonzero off-cube composition. In24 full GHASH row-point fixtures, all six word arrays recombine to direct packed-table MLE evaluations, including a padded zero row.

These are scalar mathematical checks, not execution of the Rust primitive, packed code, oracle channel or full proof.

**Remember:** pack linearly, prove one quadratic field relation, then authenticate the six word columns.

Next: the shift reduction that connects all native-operation outputs to the shared witness.
