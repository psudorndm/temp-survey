# Lesson 021 — one GKR product-tree layer and its line reduction

Introduced: 2026-10-23. Understanding unchecked. Pinned code revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf; Blueprint revision: 05061c6f9eca316b6844396883afb9ae37ff2fe9.

**One idea:** move a parent evaluation claim to two child evaluations using MLE-check, then use one fresh selector challenge to turn those two claims back into one. Repeating this prevents the number of claims from doubling at every tree level.

Prerequisites: [MLE-check](008-zerocheck-and-mle-check.md), [point switching](017-batching-and-rerandomization.md), and [product-tree layout](020-bit-selected-powers-and-product-trees.md).

## 1. The objects in one layer

Let C(X,J) be the child layer's multilinear extension. The vector X contains the current row and node coordinates; J is its highest-index child-selector coordinate. The two child sections are

$$
A(X)=C(X,0),\qquad B(X)=C(X,1).
$$

The parent table is $P[x]=A(x)B(x)$ at Boolean indices x. Write $\widetilde{P}$ for its MLE. The current proof obligation is

$$
\widetilde P(\rho)=s.
$$

$\widetilde{P}(X)$ generally differs from A(X)B(X) off the Boolean cube: the former is multilinear, while the latter can have degree two in every coordinate. This distinction is the reason the next step is a sum, rather than a direct product check at $\rho$.

## 2. Turn the parent claim into a weighted sum

By multilinear interpolation,

$$
s=\sum_{x\in\{0,1\}^n}\operatorname{eq}(\rho,x)A(x)B(x).\tag{1}
$$

A plain sumcheck on the entire summand would have individual degree three, because the equality indicator adds one. The actual prodcheck verifier instead calls the **factored degree-two MLE-check** studied in Lesson008. It keeps the known equality factors outside the prover's degree-two prime round polynomials.

For a one-variable layer, the round polynomial is simply $R(T)=A(T)B(T)$, and its weighted endpoint equation is

$$
s=(1-\rho)R(0)+\rho R(1).
$$

For more variables, each round sums over the remaining low Boolean indices with their old equality weights, while previously bound high coordinates use the fresh round challenges. The prime polynomial still has degree at most two in the current variable.

The protocol binds high coordinates first. After n rounds, let t be the new point in low-to-high coordinate order. The remaining target is

$$
A(t)B(t)=v.\tag{2}
$$

The old point $\rho$ is replaced by t. The output target is the degree-two composition evaluated at t, not necessarily the parent MLE evaluated at t and not necessarily zero.

## 3. Receive two child evaluations

The prover sends purported values

$$
u_0=A(t),\qquad u_1=B(t).
$$

The verifier checks $u_{0}u_{1}=v$. This is an algebraic consistency check. It does not, by itself, prove that either received value is an authentic evaluation of the child layer. Those two claims must be carried toward the actual leaves.

Naively recursing on both children would double the claim count at every level. The child-selector coordinate gives a cheaper route.

## 4. One random line combines the two claims

Only after $u_{0}$ and $u_{1}$ are in the transcript, sample a fresh $\eta\in K$. Since C is multilinear in its single selector coordinate J,

$$
C(t,\eta)=(1-\eta)C(t,0)+\eta C(t,1).
$$

The verifier computes

$$
w=u_0+(u_1-u_0)\eta.
$$

and continues with **one** claim

$$
C(t,\eta)=w.\tag{3}
$$

No extra line coefficients are sent: the two endpoint values determine the affine polynomial. The next point is $t||\eta$, not $\rho\mathbin\Vert\eta$. Existing content coordinates have already been re-randomized by MLE-check, and $\eta$ is appended as the new highest coordinate.

This works because the two child points agree in every coordinate except J. It is not a general license to interpolate linearly between two arbitrary MLE evaluation points that differ in many coordinates; such a restriction can have higher degree.

## 5. Complete $\mathbb{F}_{4}$ transcript

Work in $\mathbb{F}_{4}=\{0,1,\alpha ,\alpha +1\}$, $\alpha ^{2}=\alpha +1$. Use one content variable X and child tables

$$
\begin{aligned}A[0]&=1,&A[1]&=\alpha,\\B[0]&=\alpha,&B[1]&=\alpha+1.\end{aligned}
$$

All four Boolean child entries are nonzero. Their MLEs and the parent MLE are

$$
\begin{aligned}A(X)&=1+(\alpha+1)X,\\B(X)&=\alpha+X,\\\widetilde P(X)&=\alpha+(\alpha+1)X.\end{aligned}
$$

because the parent table is$[\alpha ,1]$. Start at old point $\rho=\alpha$, giving the honest claim $s=\widetilde{P}(\alpha )=\alpha +1$.

**MLE-check round.** The prime polynomial is

$$
R(T)=A(T)B(T)=\alpha+(\alpha+1)T^2.
$$

Its endpoints are $R(0)=\alpha$ and $R(1)=1$, and

$$
(1+\alpha)\alpha+\alpha\cdot1=\alpha+1=s.
$$

The prover sends its two nonconstant coefficients$[0,\alpha +1]$. The verifier reconstructs the constant as

$$
\begin{aligned}a_0&=s-\rho(a_1+a_2)\\&=(\alpha+1)+\alpha(\alpha+1)=\alpha.\end{aligned}
$$

Now sample $t=\alpha +1$. The new target is $R(t)=\alpha +1$. Note that $\widetilde{P}(t)=0$: directly checking the parent MLE at the fresh point against the child product would be wrong.

**Child values.** The prover sends $u_{0}=A(t)=\alpha +1$ and $u_{1}=B(t)=1$. Their product $is\alpha +1$, matching the target.

**Line challenge.** After receiving both, sample $\eta=\alpha$. Then

$$
\begin{aligned}w&=(\alpha+1)+(1-(\alpha+1))\alpha\\&=(\alpha+1)+\alpha^2=0.\end{aligned}
$$

The next claim is $C(\alpha +1,\alpha )=0$. Indeed the child table has MLE

$$
C(X,J)=1+(\alpha+1)X+(\alpha+1)J+\alpha XJ.
$$

which evaluates to zero there. A zero off-cube claim is compatible with the four nonzero Boolean entries.

| Stage | Remaining claim |
|---|---|
| Parent input | $\widetilde{P}(\alpha )=\alpha +1$ |
| After MLE-check | $A(\alpha +1)B(\alpha +1)=\alpha +1$ |
| Child values received | $A(\alpha +1)=\alpha +1$, $B(\alpha +1)=1$ |
| After line challenge | $C(\alpha +1,\alpha )=0$ |

## 6. Why a fresh line challenge binds the pair

At the fixed new content point t, let the received child values have errors $\delta_0$ and $\delta_1$ relative to the actual child MLE. The resulting line error is

$$
\Delta(\eta)=(1-\eta)\delta_0+\eta\delta_1.
$$

If at least one child claim is false, $\Delta$ is a nonzero polynomial of degree at most one. Since the errors are fixed before $\eta$ is sampled, it can vanish with probability at most1/|K|. If $\delta _{0}=\delta _{1}\neq 0$, it is a nonzero constant and never vanishes.

In the worked example, false child values $(\alpha,\alpha)$ preserve the required product, because $\alpha ^{2}=\alpha +1$. They nevertheless disagree with the actual pair $(\alpha+1,1)$. Their line combination coincides with the true line only $at\eta =\alpha +1$, one of the four challenges. This illustrates both the need for the line step and its probabilistic nature.

If errors could be chosen after $\eta$, the choice $(\delta_0,\delta_1)=(\eta,\eta+1)$ would make $\Delta(\eta)=0$ for every $\eta$. This demonstrates failure of the line-binding argument under that ordering; it is not a complete attack on the full product-check protocol. The actual verifier receives both child values and checks their product before sampling $\eta$.

## 7. Local reduction soundness and cost

For one layer with n content variables, the degree-two MLE-check has the usual at-most2n/|K| false-claim reduction error. The line step adds at most1/|K|. A conservative local bound is therefore

$$
\frac{2n+1}{|K|}.
$$

capped at one, under the standard fixed truth-table and fresh-challenge assumptions, and conditional on authenticating the final child-layer evaluation downstream. This is a statement about transporting a false claim; neither intermediate scalar consistency check is a commitment opening.

For k tree levels above $\ell$ row coordinates, the successive content dimensions are $\ell,\ell+1,\ldots,\ell+k-1$. Summing these local bounds gives

$$
\frac{2k\ell+k^2}{|K|}.
$$

At $k=6$ this $is(12\ell +36)/|K|$ for this isolated product-tree reduction, before leaf-discharge, outer batching, commitment and other protocol errors. It matches the Blueprint's asymptotic $O(k(\ell +k))/|K|$ form; it is not a concrete security claim for Binius64.

Each layer sends two field elements per MLE-check round, then two child evaluations. It samples one MLE-check challenge per content variable and one line challenge. Even at $n=0$, there are no MLE-check rounds, but the verifier still checks the child product and samples the selector challenge. IntMul phase1 additionally re-exposes64 leaf-section evaluations after all tree levels, as Lesson020 recorded.

## 8. Exact source correspondence

- [Verifier prodcheck.rs](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip/src/prodcheck.rs), verify: calls mlecheck::verify(point,2,eval), receives two child values, asserts their product, samples $\eta$, computes extrapolate_line, reverses MLE-check challenges and appends $\eta$, then recurses with k−1.
- [Prover prodcheck/mod.rs](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip-prover/src/prodcheck/mod.rs), layer_prover and prove: pops the next child layer, creates its split-half bivariate-product prover, runs factored MLE-check, sends both resulting evaluations and derives the same appended-point claim. Batch/unequal-depth paths remain unreviewed.
- [bivariate_product_mle.rs](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip-prover/src/sumcheck/bivariate_product_mle.rs), new_split_half: hands the whole child buffer to MleStore::push_split_half, registers the product composition in a QuadraticMleEvaluator and constructs SharedMleCheckProver at the old parent point. The low half fixes the highest selector to0 and the high half to1. Store/kernel internals and the detailed quadratic interpolation optimization were not traced here.
- [mlecheck.rs](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip/src/mlecheck.rs), verify and RoundProof::recover: loops over old coordinates in reverse order, receives two nonconstant coefficients before sampling each challenge, recovers the constant from the weighted endpoint equation and updates the prime evaluation target. The plain path, not its ZK variant, is analyzed here.
- [line.rs](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/math/src/line.rs), extrapolate_line: computes $x_{0}+(x_{1}-x_{0})z$.
- [Blueprint](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex), §4.5.2 product-tree argument: weighted parent/child relation, degree-two MLE-check and selector-line reduction.

Independent $\mathbb{F}_{4}$ checks covered16,384 exhaustive one-variable honest layer configurations,1,024 two-variable configurations, the complete numerical transcript, challenge ordering and the zero-variable edge case. Checked all nonzero child-error pairs for the degree-one root bound, plus adaptive cancellation. For the fixed worked child table and old point, exhaustively optimized false parent claims, compressed round polynomials and allowed child pairs, assuming an exact final child evaluation check; the maximum toy acceptance probability was5/8, below the conservative3/4 local bound. This is a finite reference experiment, not a Rust proof-system test or full security audit.

**Remember:** parent claim $\to$ weighted product check $\to$ two children $\to$ one fresh selector $\to$ one new child-layer claim.

Next: the Frobenius automorphism—why repeated squaring is $\mathbb{F}_2$-linear and invertible—and how it prepares the variable-base leaf-discharge argument.
