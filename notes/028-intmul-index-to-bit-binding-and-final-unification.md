# Lesson 028 — Tie the lookup addresses back to the bits

Introduced: 2026-10-23. Understanding unchecked. Code revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** a limb address is a fixed linear combination of operand bits. Linearity lets the last IntMul sumcheck check its index claims using the same bit-column outputs that go to witness authentication.

Prerequisites: [fixed-base limbs](025-fixed-base-limbs-and-transparent-power-table.md), [lookup reduction](027-logup-rational-identity-and-fraction-gkr.md), [IntMul range/parity argument](019-intmul-exponent-encoding-and-wraparound.md).

## 1. What the lookup returns

The transparent lookup returns 12 embedded-index evaluations at one common row point $r_I$:

$$
\nu_j=\widetilde{\overline I}_j(r_I),\qquad 0\leq j<12.
$$

Column $j=4t+l$ refers to limb $l$ of word family $t$, in order $a,c_{\mathrm{lo}},c_{\mathrm{hi}}$. Each address has 16 bits. Its field encoding is

$$
\overline I_{t,l}[x]
=\sum_{u=0}^{15}\beta_u\, e_{t,16l+u}[x],
\qquad \beta_u=\operatorname{basis}(u).
\tag{1}
$$

The field basis resets at zero for every limb: the coefficient is $\beta_u$, not $\beta_{16l+u}$. The limb position is already accounted for by the twisted exponentiation tables.

MLE interpolation is linear, so equation (1) holds at every row point:

$$
\widetilde{\overline I}_{t,l}(r)
=\sum_{u=0}^{15}\beta_u\widetilde e_{t,16l+u}(r).
\tag{2}
$$

No integer carries appear in this relation.

## 2. This is an address encoding, not a small-field embedding

The prover's embed_position places the address's bits in the field's binary coordinate basis. IntMul independently builds a table of all address embeddings using the same basis:

$$
\iota(0)=0,\qquad
\iota(k)=\iota\bigl(k\mathbin{\&}(k-1)\bigr)
+\beta_{\operatorname{ctz}(k)}.
$$

The bit operation clears the lowest set bit. The recurrence therefore adds exactly the basis elements corresponding to set address bits.

For the polynomial-basis GHASH representation, these 16-bit addresses span the low16 bit coordinates. That span is not a multiplicatively closed subfield: $X^{15}\cdot X=X^{16}$ leaves it. This is deliberately a bit-linear address embedding, different from the arithmetic-preserving AES embedding in Lesson 003.

## 3. Compress the twelve index claims

After receiving the 12 index evaluations from lookup, sample $\rho\in K^4$. Pad the vector to16 with zeros and use

$$
\lambda_j=\operatorname{eq}_4(\rho,j),\qquad
v=\sum_{j=0}^{11}\lambda_j\nu_j.
$$

Define the folded row table

$$
V[x]=\sum_{j=0}^{11}\lambda_j\overline I_j[x].
$$

Then the remaining index claim is $\widetilde V(r_I)=v$. For a fixed nonzero padded error vector, this tensor compression has local error at most $4/|K|$. The values must be fixed before $\rho$.

The prover builds $V$ row by row from integer index columns and an embedding table; it does not materialize all 12 embedded field columns first.

## 4. Three claims share the final sumcheck

From phase3, let $r_B$ be the row point and $r_I^b\in K^6$ the bit-index recombination point. Define

$$
B(X)=\sum_{i=0}^{63}\operatorname{eq}_6(r_I^b,i)\widetilde b_i(X),
\qquad B(r_B)=b^\star.
$$

The three sum claims are:

$$
\begin{aligned}
v&=\sum_x \operatorname{eq}(r_I,x)\widetilde V(x),\\
0&=\sum_x \operatorname{eq}(r_B,x)
\left(\widetilde a_0(x)\widetilde b_0(x)-\widetilde c_{\mathrm{lo},0}(x)\right),\\
b^\star&=\sum_x \operatorname{eq}(r_B,x)B(x).
\end{aligned}
$$

The middle statement is the low-bit parity relation needed to remove the sole endpoint alias of the exponent encoding. The old $r_B$ is the actual phase3 output point, despite older phase-number wording in nearby comments.

With fresh scalar $\theta$, the batch target is

$$
v+\theta\cdot0+\theta^2b^\star.
$$

The summands have individual degrees two, three and two respectively, so the ordinary batched degree bound is three. The parity residual is constrained on Boolean rows; it need not vanish at an arbitrary off-cube point.

## 5. Close from one set of raw bit-column evaluations

At the new row point $r_{\mathrm{out}}$, the prover sends four64-entry arrays. Wire order is

$$
a,\quad c_{\mathrm{lo}},\quad c_{\mathrm{hi}},\quad b.
$$

The verifier reconstructs all 12 index-column values using equation (2), pads to16 and evaluates them at $\rho$ to obtain $V^\star$. It reconstructs

$$
B^\star=\sum_i\operatorname{eq}_6(r_I^b,i)b_i^\star.
$$

Its expected terminal value is

$$
\begin{aligned}
&\operatorname{eq}(r_I,r_{\mathrm{out}})V^\star\\
&\quad+\theta\operatorname{eq}(r_B,r_{\mathrm{out}})
\left(a_0^\star b_0^\star-c_{\mathrm{lo},0}^\star\right)\\
&\quad+\theta^2\operatorname{eq}(r_B,r_{\mathrm{out}})B^\star.
\end{aligned}
$$

This is compared with the ordinary sumcheck's terminal target. In particular the verifier does not separately require the off-cube parity residual to be zero.

The returned IntMulOutput names the arrays and its operand_claims method supplies downstream order

$$
[a,b,c_{\mathrm{lo}},c_{\mathrm{hi}}],
$$

all at $r_{\mathrm{out}}$. These are still claims to be authenticated by later rerandomization, shift/wiring and witness commitments. An honest prover computing the arrays directly from its words is not itself a malicious-prover security argument.

## 6. Why the chain now connects the right objects

The chain is:

1. Integer product becomes exponentiation equality, plus a parity obligation.
2. Variable-base GKR and Frobenius selectors produce base/output exponentiation claims and a bit-recombined $b$ claim.
3. Fixed-base GKR reduces base/output claims to limb-table gathers.
4. Lookup ties the gathers to embedded-index claims.
5. Basis linearity ties those index claims to the same operand bit columns used for parity and the recombined $b$ claim.
6. Later witness authentication must tie those columns to the actual circuit operands.

The subprotocol therefore returns a common interface of raw-bit claims. It has not independently completed the full witness proof.

## Sources and checks

- [IntMul verify_phase_5](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/intmul/verify.rs): full body through output construction, basis recombination, batch ordering and terminal assertion.
- [IntMul prover phase5](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/intmul/prove.rs): index gathering, embedding table, folded column, final three-prover batch and raw outputs.
- [Common output interface](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/intmul/common.rs): operand_claims order.
- [Lookup embedding](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip-prover/src/logup_star/witness.rs): embed_position and index-denominator construction.

Independent checks use an eight-bit-word miniature with four two-bit limbs, four valid integer-multiplication rows, 12 index columns and16-column padding. Across16 challenge fixtures and all256 output row points over $\mathbb F_{16}$, reconstructed index evaluations match direct folded-column evaluations and all4,096 terminal compositions match an explicitly expanded polynomial of individual degree at most three. The initial Boolean sum matches the three-claim target. The valid parity residual is nonzero at3,584 off-cube sample points, confirming why a separate zero-at-terminal assertion would be wrong. A GHASH polynomial-basis example checks the address-span distinction.

No Rust proof, oracle opening, packed kernel or end-to-end circuit execution was performed.

**Remember:** the address is linear in bits; the final sumcheck checks addresses, exponent recombination and parity using one common set of bit-column outputs.

Next: BinMul, then the shift/witness layer that authenticates the native operation outputs.
