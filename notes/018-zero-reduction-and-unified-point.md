# Lesson 018 — Zero constraints and the unified point

Introduced: 2026-10-23. Understanding unchecked. Pinned implementation revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf; pinned Blueprint revision: 05061c6f9eca316b6844396883afb9ae37ff2fe9.

**One idea:** a Zero array is already a linear residual. Its interpolation is identically zero when valid, so the reduction can immediately claim zero at a fresh point and pass that claim to witness authentication.

Prerequisites: [native constraints](009-binius64-constraint-language.md), [oblong interpolation](011-oblong-multilinearization.md), and [re-randomization](017-batching-and-rerandomization.md).

## 1. What does a Zero constraint assert?

A native Zero constraint has one operand d[x], an XOR of references to words with fixed shifts. It asserts $d[x]=0$ as a 64-bit word. For example,

$$
d[x]=u[x]\oplus v[x]\oplus t[x]=0.
$$

expresses $t[x]=u[x]$ XOR v[x]. Every output bit is an $\mathbb{F}_2$-linear function of witness bits. Fixed logical, rotational or arithmetic shifts preserve this linearity: they copy bits or insert zero bits.

For n_zero rows, let $\ell _{\mathrm{zero}}=\lceil\log_2(n_{\mathrm{zero}})\rceil$, using the implementation's empty-set convention discussed below. Zero-pad the rows to a power of two. Write d[i,x] for bit i of row x. The residual is the oblong polynomial

$$
\widehat d(Z,X)=\sum_{i=0}^{63}L_i(Z)\widetilde d_i(X).
$$

with Z on the 64-point additive bit domain D, and X containing $\ell _{\mathrm{zero}}$ row coordinates. Each $\widetilde{d}_i$ is the row MLE of the bit table. Thus $\deg_Z(\widehat{d})\leq 63$ and each row variable has $\deg\leq 1$.

If every residual bit is zero, all interpolation coefficients vanish and **$\widehat{d}$ is the zero polynomial**. Conversely, if some residual bit equals one, $\widehat{d}$ takes value one at its corresponding domain point, so it is not the zero polynomial.

Compare the earlier valid AND example: a product of operand MLEs can yield $X^{2}+X$, which vanishes on Boolean inputs yet is nonzero off the cube. For Zero, the residual is directly the unique interpolant of its zero table; no product raises the degree and changes that conclusion.

## 2. The entire reduction

After BitAnd's skip and batched row sumcheck, use their fresh bit-axis challenge z and row point $r^\star$. The Zero reduction outputs

$$
\widehat d\bigl(z,r^\star|_{\ell_{\mathrm{zero}}}\bigr)=0.
$$

There is no Zero sumcheck, no Zero round polynomial, and no scalar value sent by the prover. The claimed value is the known constant zero. The later shift reduction checks that claim against the actual witness-derived operand.

“No message” is a statement about this reduction, not about the cost of proving Zero constraints end to end. The extra array still contributes to the wiring/shift computation and to later authentication.

## 3. A tiny invalid table

For a two-bit word and two rows, take residual words $[10_{2},00_{2}]$. Let the bit domain be {0,1}, the bit variable Z and the row variable X. The only nonzero residual is bit1 of row0, so

$$
\widehat d(Z,X)=Z(1+X).
$$

Over $\mathbb{F}_{4}$, with $\alpha ^{2}=\alpha +1$, evaluating at $z=x=\alpha$ gives

$$
\widehat d(\alpha,\alpha)=\alpha(1+\alpha)=1.
$$

The demanded value zero is false. Authenticating it against this fixed table would expose the violation.

An invalid table can nevertheless evaluate to zero at some points: here $z=0$ or $x=1$ makes the evaluation zero. There are 7 such pairs among the 16 $\mathbb{F}_{4}^{2}$ pairs. That is why a random evaluation is a probabilistic test and the real challenge field must be large.

XORing all residual words would also be insufficient. For $[10_{2},10_{2}]$, the raw XOR is zero, but its interpolant is $\widehat{d}(Z,X)=Z$, which is not identically zero.

## 4. Why reusing the point is sound

Assume the witness is fixed by a binding oracle commitment before these challenges. A violated Zero constraint then determines a fixed nonzero $\widehat{d}$ of total degree at most $63+\ell_{\mathrm{zero}}$. For independent uniform challenges in K,

$$
\Pr\!\left[\widehat d\bigl(z,r^\star|_{\ell_{\mathrm{zero}}}\bigr)=0\right]\leq\min\!\left(1,\frac{63+\ell_{\mathrm{zero}}}{|K|}\right).
$$

This is the local Schwartz–Zippel bound, not a complete proof-system soundness error. The later batching, shift reduction and commitment opening have their own conditions and error terms.

The row point here consists of **fresh sumcheck round challenges**. It is not the initial BitAnd equality-weighting point whose first three coordinates are deterministic AES constants from Lesson012. Reusing those constants would require a different argument.

Other reductions also use this point, so their checks are correlated. The local Zero bound needs the point's uniform marginal distribution and a fixed residual; it does not need independent random points for different constraint families. Analyze its bad event without conditioning on other reductions having passed. Full acceptance with a violated Zero constraint requires either this bad event or a failure of later authentication. An adversary aborting on unfavorable challenges does not enlarge the unconditional acceptance probability.

In an interactive ideal-channel analysis, each challenge is fresh even though the preceding prover message may depend on earlier challenges. Fiat–Shamir/channel security remains a separate study topic.

## 5. What if Zero has more rows?

Let $\ell _{\mathrm{sc}}$ be the maximum row-variable count among BitAnd, IntMul and BinMul. Their shared sumcheck provides $\ell _{\mathrm{sc}}$ row coordinates. Set

$$
\ell_{\mathrm{cons}}=\max(\ell_{\mathrm{sc}},\ell_{\mathrm{zero}}),\qquad r_x=(r_{\mathrm{sc}},\text{fresh high coordinates}).
$$

where exactly $\operatorname{max}(\ell _{\mathrm{zero}}-\ell _{\mathrm{sc}},0)$ new coordinates are drawn. These are appended at the high end of the low-to-high coordinate list.

| Zero dimension | Unified point | Coordinates read by the unpadded Zero array |
|---|---|---|
| $\ell _{\mathrm{zero}}<\ell _{\mathrm{sc}}$ | Keep r_sc | Its first $\ell _{\mathrm{zero}}$ coordinates |
| $\ell _{\mathrm{zero}}=\ell _{\mathrm{sc}}$ | Keep r_sc | All coordinates |
| $\ell _{\mathrm{zero}}>\ell _{\mathrm{sc}}$ | Append the shortfall | All coordinates of the extended point |

These draws add no sumcheck rounds or prover field elements. They do advance the challenge stream, so prover and verifier must sample them in the same place.

The code supports batched instances too: low instance coordinates r_rho come before the per-instance constraint coordinates. reduce_constraints splits them off before extending the constraint point. This lesson's numerical explanation is single-instance; full batched-instance wiring and proving are not traced here.

## 6. A second alignment step: prefix values to padded matrices

Lesson017 ended with unpadded arrays evaluated at prefixes. The actual shift interface takes all operation matrices padded to one full row dimension. These views are equivalent, but their evaluation values differ by a known factor.

For a row-multilinear M on $\ell$ coordinates, its zero-padded matrix has extension

$$
M^+(X)=M(X_0,\ldots,X_{\ell-1})\prod_{j=\ell}^{\ell_{\mathrm{cons}}-1}(1-X_j).
$$

Consequently, if $M(r_x|_\ell )=a$, its full padded claim is

$$
M^+(r_x)=a\,\operatorname{eq}(0,r_x[\ell..]).
$$

The verifier's padding_scales computes these factors by suffix products once and selects the factor for each operation's row count. The prover applies the same factors in OperandClaims::from_rerand. Zero's value remains zero under any scale, so its scale is not needed when assembling its claimed scalar.

There are two distinct uses of high-variable padding in the pipeline:

1. Inside Lesson017's combined sumcheck, it aligns summands' variable counts, and the closing equation includes those weights.
2. At the shift interface, the new output claims are lifted to the actual padded operation matrices at the full constraint point, including any freshly appended Zero coordinates.

The final shift claim does not accidentally include the old equality factor $\operatorname{eq}(\rho ,r)$: that factor belonged to the previous sumcheck's closing identity. Only the new array value and its matrix-padding scale are passed on.

A padding scale can be zero, for example when a high coordinate equals one. Implementations multiply by scales, without dividing. For $\nu$ padding coordinates the chance of this event is at most $\nu/|K|$; a full reduction analysis must account for such degeneracies instead of assuming all scales are always invertible.

## 7. Four families, fourteen columns, four fresh batching coordinates

The assembled shift claim vector has this fixed order:

| Slots | Family | Arity |
|---|---|---|
| 0 | Zero | 1 |
| 1–3 | BitAnd | 3 |
| 4–7 | IntMul | 4 |
| 8–13 | BinMul | 6 |

Absent multiplication families contribute zero scalars in their slots. The empty Zero set is represented vacuously by one zero row and zero row variables: log_zero_constraints returns None, read as0 by the callers. A single real Zero row also has zero row variables, but a nonzero bit-axis polynomial can still expose an invalid word. Do not interpret zero variables as zero entries.

After these claims are fixed, the actual shift verifier samples **four fresh coordinates**, creates the16 equality-indicator weights for a four-variable operand axis, and uses the first14 to combine the claims. The two unnamed slots are zero padding. The prover's prepare samples and expands the same four-coordinate tensor.

These weights are structured, not fourteen independent field elements and not powers of the previous $\theta$. For any fixed nonzero vector of padded claim errors, their weighted sum is the evaluation of a nonzero four-variate multilinear. Its total degree is at most4, giving the local bound4/|K| under the usual fresh-challenge assumptions. This explains the concrete implementation of the Blueprint's generically described fresh downstream batching coefficients. It still requires the rest of the shift/witness proof to authenticate the batched value.

## 8. Source anchors and scope

All code links are at the pinned implementation revision:

- [zero.rs](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/zero.rs): reduction_point copies the available prefix and calls a supplied sampler for missing coordinates. Both prover and verifier use this helper. Its tests were read, not executed.
- [reduction.rs](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/reduction.rs): reduce_constraints runs IntMul/BinMul before BitAnd, splits the sumcheck point, extends for Zero, computes padding_scales, and inserts the constant-zero operand claim. Source includes other public-input routines that were not analyzed in this lesson.
- [Prover claim assembly](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/shift/claims.rs): from_rerand mirrors extension/scales; prepare samples the operand axis and builds its equality weights.
- [Shift verifier](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/shift/verify.rs): only the initial claim batching and call boundary were traced; the full shift sumcheck and closing/wiring checks remain future work.
- [Constraint system](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/core/src/constraint_system/system.rs): log_zero_constraints empty/ceil-log conventions. Retaining this large source does not imply all its validation/serialization methods were reviewed.
- [Prover orchestration](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/prove.rs) and [verifier orchestration](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/verify.rs): send_oracle/recv_oracle occurs before the constraint-reduction calls in the channel abstraction. Binding of the concrete PCS and transcript implementation is not established merely by this call order.
- [Blueprint source](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex), §4.4 Zero reduction and §4.7.1 output unification: interpolation equivalence, total-degree bound and marginal-distribution argument.

Independent scalar verification covered all15 invalid binary two-bit/two-row residuals at all16 $\mathbb{F}_{4}^{2}$ points; the7/16 worked example;1398 direct padded-MLE versus prefix-times-scale comparisons; fresh-extension prefix marginals; and three actual64-point AES Lagrange basis residuals at all256 AES values, each with exactly63 roots. These are mathematical reference checks. No Binius Rust, concrete channel, full shift proof, PCS or machine kernel was executed.

**Remember:** Zero supplies a known-zero claim without a new proof message; the witness authentication is where that claim is enforced.

Next: why IntMul is encoded as an identity in the multiplicative group of the binary field, and what information exponent wraparound can lose.
