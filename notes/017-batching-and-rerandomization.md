# Lesson 017 — batching claims and moving them to a shared point

Introduced: 2026-10-23. Understanding unchecked. Code revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf; Blueprint source revision: 05061c6f9eca316b6844396883afb9ae37ff2fe9.

**One idea:** equality weights turn an evaluation claim at its old point into a sum. A shared sumcheck then moves several claims to the same new point.

Prerequisites: [equality weights](006-equality-weights-and-folding.md), [sumcheck](007-sumcheck-a-complete-transcript.md), and [BitAnd skip message](013-univariate-skip-message.md). This lesson returns from NTT preprocessing to the main protocol.

## 1. What needs to be unified?

Fix the shared bit-axis point z from BitAnd's univariate skip. An operand column becomes a row-multilinear $M(X)=\widehat{z}(z,X)$. IntMul and BinMul leave claims at their own row points $\rho$; BitAnd's row sumcheck will produce a different point $r^\star$. The downstream shift reduction benefits from having all operand claims at that common point, or low-coordinate prefixes for shorter arrays.

The Blueprint says a shift reduction can algebraically handle distinct row points, but each distinct point adds a tensor expansion and a verifier pass over constraint lists. Point unification is a cost optimization. The shared bit-axis point plays a separate role in the shift reduction's common factors.

There are two operations here:

1. **Batch:** turn many claimed identities into one random linear combination.
2. **Move the point:** use sumcheck so the remaining evaluations are at its fresh challenge point.

Adding claimed values alone performs neither the point translation nor its proof.

## 2. Move one multilinear claim

Suppose the existing claim is $M(\rho )=a$ for an n-variate multilinear M. Multilinear interpolation gives

$$
a=\sum_{x\in\{0,1\}^n}\operatorname{eq}(\rho,x)M(x).
$$

Here $\operatorname{eq}(\rho ,X)=\prod _{j}((1-\rho _{j})(1-X_{j})+\rho _{j}X_{j})$. In characteristic two each factor simplifies to $1+\rho _{j}+X_{j}$.

Run sumcheck on $P(X)=\operatorname{eq}(\rho ,X)M(X)$, of individual degree at most two. Its final target is

$$
P(r^\star)=\operatorname{eq}(\rho,r^\star)M(r^\star).
$$

The verifier computes $\operatorname{eq}(\rho ,r^\star )$ from public challenge points. The prover supplies a new claimed value $M(r^\star )$, which still needs authentication against the committed witness later. This is a reduction of the old evaluation claim to a new one.

There is no need to solve $M(r^\star )$ from a known scalar by dividing by eq. The real verifier receives the claimed value and checks a multiplication identity, including when the weight is zero.

## 3. Move several claims together

For two same-dimension claims $f(\rho )=a$ and $g(\sigma )=b$, sample $\theta$ after the claims are fixed and define

$$
H(X)=\operatorname{eq}(\rho,X)f(X)+\theta\operatorname{eq}(\sigma,X)g(X),\qquad S=a+\theta b.
$$

Then $S=\sum _{x}H(x)$. One sumcheck leaves

$$
H(r^\star)=\operatorname{eq}(\rho,r^\star)f(r^\star)+\theta\operatorname{eq}(\sigma,r^\star)g(r^\star).
$$

Both polynomials now appear at $r^\star$. Their old points survive in their separate equality factors. In general there is no common equality indicator to factor out, since $\rho$ and $\sigma$ differ. This is why the implementation converts each factored MLE-check to ordinary sumcheck and uses batch_verify, rather than the common-point batch_verify_mle interface.

For m claims, use coefficients $1,\theta ,\ldots ,\theta ^{m-1}$. If the original fixed claim errors are $e_{0},\ldots ,e_{m-1}$, a false batch can accidentally cancel only when

$$
E(\theta)=e_0+\theta e_1+\cdots+\theta^{m-1}e_{m-1}=0.
$$

For a nonzero fixed E, this happens with probability at most (m−1)/|K|, capped at one. A deterministic equal-weight sum has no such guarantee: two equal errors cancel in characteristic two. If errors can be chosen after $\theta$, choosing $(e_{0},e_{1})=(\theta ,1)$ cancels for every $\theta$. Message ordering is essential.

## 4. Complete one-round toy transcript

Use $\mathbb{F}_{4}$ with $\alpha ^{2}=\alpha +1$. Let $f(X)=1+X$ and $g(X)=X$, with old claims $f(0)=1$ and $g(1)=1$. Choose the illustrative batching challenge $\theta =\alpha$.

Since $\operatorname{eq}(0,X)=1+X$ and $\operatorname{eq}(1,X)=X$,

$$
\begin{aligned}H(X)&=(1+X)(1+X)+\alpha X^2\\&=1+(\alpha+1)X^2.\end{aligned}
$$

The initial claim is $S=1+\alpha =\alpha +1$. Indeed $H(0)=1$, $H(1)=\alpha$, so $H(0)+H(1)=S$.

There is one row variable, so the prover's only round polynomial is H(T). To illustrate the actual degree-three wire format, write its coefficients as $[1,0,\alpha +1,0]$. The first three are sent; the omitted cubic coefficient is recovered as $S-a_{1}-a_{2}=0$. The constant does not contribute to R(0)+R(1) in characteristic two.

Now sample the illustrative row challenge $r^\star =\alpha$. The new claims are

$$
f(\alpha)=\alpha+1,\qquad g(\alpha)=\alpha.
$$

The closing identity checks

$$
\begin{aligned}\operatorname{eq}(0,\alpha)(\alpha+1)+\alpha\operatorname{eq}(1,\alpha)\alpha&=(\alpha+1)^2+\alpha^3\\&=\alpha+1\\&=H(\alpha).\end{aligned}
$$

Both operands moved to $\alpha$ from different old points. The new values differ from their old values; the protocol proves the connection through the weighted sum, rather than assuming values remain constant while points move.

Tiny-field challenges are used only to make arithmetic visible. They do not provide cryptographic security.

## 5. The actual BitAnd batch

Before BitAnd samples z, the multiplication reductions have supplied 64 per-bit values for each operand column. The verifier already has these values. It folds each column locally using the 64-point Lagrange weights:

$$
a_j=\sum_i L_i(z)\,\operatorname{claimed\_bit\_eval}_j[i].
$$

This needs no new proof message. For a fixed nonzero vector of per-bit errors, its Lagrange interpolant is a nonzero degree-at-most-63 polynomial in z, so a fresh random z supplies the local binding test. This is why per-bit values must precede z. This local statement is separate from a full protocol soundness bound.

In the full ten-column layout there are four IntMul operand columns and six BinMul operand columns. The actual helpers accept the supplied column groups, including empty groups. Put the BitAnd claim g(z) first, followed by the folded claims in operand order. With equal dimensions, the combined summand is

$$
\begin{aligned}F(X)={}&\operatorname{eq}(\tau,X)\bigl(A(z,X)B(z,X)-C(z,X)\bigr)\\&+\sum_j\theta^j\operatorname{eq}(\rho_j,X)M_j(X).\end{aligned}
$$

and its claimed sum is $g(z)+\sum _{j}\theta ^{j}a_{j}$, with operand exponents starting at one.

The BitAnd residual has individual degree two in row variables, and its equality factor adds one: degree three. Each operand term has individual degree two. The whole ordinary sumcheck therefore uses degree bound three and sends three field elements per round after highest-coefficient compression. A standalone degree-two factored BitAnd MLE-check would send two. Separate old equality points prevent keeping one shared factored indicator outside this batch.

## 6. Unequal row dimensions

If a term $P(X_{0},\ldots ,X_{\ell -1})$ has fewer variables than the longest term, pad it on the high coordinates:

$$
P^+(X)=P(X_0,\ldots,X_{\ell-1})\prod_{j=\ell}^{n-1}(1-X_j).
$$

The extra indicator is one only on the all-zero Boolean high block, so $\sum P^{+}=\sum P$. Extending P independently of the extra variables would repeat its sum $2^{n-\ell }$ times; over a binary field this is zero as soon as one dimension is added.

The reference protocol binds high variables first. PaddedSumcheckDecorator emits a padding round from the original claim and accumulated padding factor, without touching its inner prover. It then scales inner rounds by the accumulated factor. Final multilinear values belong to the inner polynomials at the **low $\ell$ coordinates** of $r^\star$; the verifier multiplies the padding factor back into the closing equation.

This is zero padding for sumcheck summands. It differs from Lesson 014's repetition across artificial fixed coordinates inside the small-field kernel, which has a different purpose.

## 7. Concrete source path and implementation differences

Read function bodies at the pinned code revision:

- [Verifier rerand.rs](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/rerand.rs): folds per-bit claims, calls batch_verify with `SUMCHECK_DEGREE = 3`, reverses the sampled challenge list into a low-to-high eval_point, receives [a,b,c] and operand_evals, and checks the combined closing identity. Its weight is eq_ind(old_point,low_point) times eq_ind_zero(high_padding).
- [Prover rerand.rs](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/rerand.rs): BitAxisFolder folds each word column; MultilinearEvalEvaluator supplies the identity composition; SharedMleCheckProver groups columns sharing an old point; MleToSumCheckDecorator restores each indicator; PaddedSumcheckDecorator aligns variable counts.
- [Verifier batch.rs](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip/src/sumcheck/batch.rs): samples $\theta$, evaluates the list of initial claims as coefficients of a univariate in $\theta$, and runs ordinary sumcheck.
- [Prover driver](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip-prover/src/sumcheck/drive.rs): samples $\theta$ once, flattens individual round polynomials in summand order, combines by Horner evaluation, sends that combined polynomial, samples one challenge, and folds every prover with it. Final scalar claims are sent after all rounds by batch_prove_and_write_evals.
- [Equality adaptor](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip-prover/src/sumcheck/mle_to_sumcheck.rs) and [mul_by_eq](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip/src/sumcheck/common.rs): execute multiplies the inner polynomial by the actual eq factor and prior equality prefix. Some adaptor comments call that factor $X-\alpha$; the body calls mul_by_eq, which correctly uses $(1+\alpha )+X$ in characteristic two. The comments are imprecise.
- [Padding implementation](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip-prover/src/sumcheck/padded.rs): high padding is bound first; inner evaluation outputs stay unscaled and are weighted by the verifier.

Blueprint §4.7.1 proposes combining same-point operand columns into two $\theta$-weighted tables, one per multiplication reduction. The pinned prover instead maintains one folded column/evaluator per operand, grouping their stores by old point but combining their round polynomials in the driver. Its own source explicitly flags the two-table optimization as deferred until profiling justifies it. Do not describe that optimization as already implemented here.

## 8. What has and has not been established

The closing check is one scalar relation among several sent evaluations. It does not independently certify every value. In the toy transcript, the two closing weights are both $\alpha +1$, so changing both new values by one leaves that relation unchanged. The downstream shift/witness checks are essential for authenticating the resulting claims. Their batching coefficients and transcript ordering need a separate end-to-end trace.

The local fixed-error batching bound is (m−1)/|K|, and a degree-three n-round sumcheck has the usual 3n/|K| reduction bound under the standard fixed-polynomial/terminal-authentication assumptions. Neither is a full proof of Binius64 security. The skip-message degree distinction recorded in Lesson 013 remains unchanged.

Independent $\mathbb{F}_{4}$ checks covered 65,536 combinations of two tables, old points, $\theta$ and new points; the worked compressed round transcript; fixed-error root bounds through degree three; adaptive cancellation; the ambiguity of an isolated closing identity; and a mixed one-/two-variable padding example. No Binius Rust, channel implementation or machine kernels were executed.

**Remember:** retain each old point in its equality weight; let one sumcheck choose the shared new point; authenticate the resulting values later.

Next: the Zero reduction and how its constant-zero claim joins the unified point, including when its row axis is the longest.
