# Lesson 023 — Frobenius twists of multilinear evaluation claims

Introduced: 2026-10-23. Understanding unchecked. Pinned code: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf. Pinned Blueprint: 05061c6f9eca316b6844396883afb9ae37ff2fe9.

**One idea:** if one table is obtained by applying Frobenius to every entry of another, undoing that transformation on an evaluation requires transforming both its value and its point.

Prerequisites: [MLE interpolation](005-multilinear-extensions.md), [equality weights](006-equality-weights-and-folding.md), [variable-base leaves](020-bit-selected-powers-and-product-trees.md), and [Frobenius](022-frobenius-automorphism-and-fixed-fields.md).

## 1. The exact table identity

Work in $K=\mathbb{F}_{2^{m}}$, with $\varphi (a)=a^{2}$. Let T be a Boolean-indexed table, and form a second table by applying $\varphi ^i$ entry by entry:

$$
Q_i[x]=\varphi^i(T[x])=T[x]^{2^i},\qquad x\in\{0,1\}^n.
$$

Write $\widetilde{T}$ and $\widetilde{Q}_i$ for their MLEs. Set $\psi =\varphi ^{-i}$. Then

$$
\psi\bigl(\widetilde Q_i(r)\bigr)=\widetilde T\bigl(\psi(r)\bigr).\tag{1}
$$

where $\psi (r)$ means applying $\psi$ to every coordinate of r. Consequently the claim $\widetilde{Q}_i(r)=s$ is equivalent to

$$
\widetilde T\bigl(\psi(r)\bigr)=\psi(s).
$$

This is a deterministic, invertible change of the claim. It samples no fresh challenge, sends no additional prover value and adds no probabilistic soundness loss of its own. Its prerequisite is the actual pointwise relation between the two tables.

## 2. Why the point has to move

Start from the interpolation formula:

$$
\widetilde Q_i(r)=\sum_{x\in\{0,1\}^n}\operatorname{eq}(r,x)\varphi^i(T[x]).
$$

Apply $\psi$ to both sides. Because $\psi$ preserves sums and products,

$$
\psi\bigl(\widetilde Q_i(r)\bigr)=\sum_x\psi\bigl(\operatorname{eq}(r,x)\bigr)T[x].
$$

The Boolean entries x_j and the coefficients0,1 of eq are fixed by Frobenius, so

$$
\psi\bigl(\operatorname{eq}(r,x)\bigr)=\operatorname{eq}\bigl(\psi(r),x\bigr).
$$

This follows factor by factor from

$$
\operatorname{eq}(r,x)=\prod_j\bigl((1-r_j)(1-x_j)+r_jx_j\bigr).
$$

Substituting gives equation(1). The point moves because the interpolation weights move. $\mathbb{F}_2$-linearity suffices together with preservation of products; K-linearity, which Frobenius generally lacks, is not being assumed.

The table equality $Q_i[x]=T[x]^{2^{i}}$ is stated on Boolean indices. It does not generally mean the formal polynomial identity $\widetilde{Q}_i(X)=\widetilde{T}(X)^{2^{i}}$ with unchanged variables. The latter raises variable degrees too, while $\widetilde{Q}_i$ is multilinear.

## 3. A complete $\mathbb{F}_{4}$ example

Take $\alpha ^{2}=\alpha +1$ and the tables

$$
T=[1,\alpha],\qquad Q=[1,\alpha+1].
$$

so Q is T squared entry by entry. Their MLEs are

$$
\widetilde T(X)=1+(\alpha+1)X,\qquad\widetilde Q(X)=1+\alpha X.
$$

At $r=\alpha$, the original evaluation is $\widetilde{Q}(\alpha )=\alpha$. In $\mathbb{F}_{4}$, inverse Frobenius is squaring. Therefore

$$
\text{new point}=\alpha^2=\alpha+1,\qquad\text{new value}=\alpha^2=\alpha+1.
$$

The transformed claim is $\widetilde{T}(\alpha +1)=\alpha +1$, which holds because

$$
1+(\alpha+1)^2=1+\alpha=\alpha+1.
$$

If only the value were transformed, the purported claim would be $\widetilde{T}(\alpha )=\alpha +1$. But $\widetilde{T}(\alpha )=0$. Moving both is essential.

## 4. The actual IntMul leaf has its own selector table

In the variable-base branch, denote the Boolean-row base by $A[x]=g^{a[x]}$ (with integer $a[x]$) and the exponent's ith bit by b_i[x]. The leaf table is

$$
Q_i[x]=1+b_i[x]\bigl(A[x]^{2^i}-1\bigr).
$$

Define the **bit-specific** unsquared table

$$
T_i[x]=1+b_i[x]\bigl(A[x]-1\bigr).
$$

Since each b_i[x] is0 or1 and therefore fixed by Frobenius,

$$
\begin{aligned}\varphi^i(T_i[x])&=1+b_i[x]\bigl(\varphi^i(A[x])-1\bigr)\\&=Q_i[x].\end{aligned}
$$

The original leaf claim $\widetilde{Q}_i(r)=s_i$ thus becomes

$$
\widetilde T_i(r_i)=u_i,\qquad r_i=\varphi^{-i}(r),\quad u_i=\varphi^{-i}(s_i).\tag{2}
$$

Different bits generally give different T_i. They share the base table A, not the complete selector table. For example, with A=$[1,\alpha ]$ and exponent words[0,2], bit0 is[0,0] while bit1 is[0,1]. Then

$$
T_0=[1,1],\qquad T_1=[1,\alpha].
$$

These are plainly different tables. The original frobenius_twist helper comment describes all twisted claims as being on one shared polynomialP. That is an oversimplification for this call site unless P is read as a family of bit-indexed sections. The helper body only transforms points and scalars; it does not assert that the underlying tables are identical. The actual next-phase verifier uses the separate b_i evaluations and the shared A evaluation, which matches equation(2).

This resolves the comment question from Lesson022. It is a documentation precision issue in the inspected path, not evidence of a protocol failure.

## 5. What has become low-degree?

Equation(2) is now a claim on the MLE of a selector table whose Boolean entries are bilinear in A and b_i. Expand it as

$$
u_i=\sum_{x\in\{0,1\}^n}\operatorname{eq}(r_i,x)\bigl[1+\widetilde b_i(x)(\widetilde A(x)-1)\bigr].\tag{3}
$$

There is no $A^{2^{i}}$ in this composition. The bracket has individual row-variable degree at most two; its known equality indicator adds one for ordinary sumcheck, giving degree at most three. This is the degree bound used by the actual combined phase3.

The high powers have not magically become low-degree formal polynomials at the same input variables. The automorphism and corresponding point transformation instead produce an equivalent evaluation claim with a different interpolation representation.

Do not shortcut equation(3) into

$$
u_i=1+\widetilde b_i(r_i)(\widetilde A(r_i)-1).
$$

The MLE of a pointwise bilinear table generally differs from the product of its input MLEs off the cube. In the $\mathbb{F}_{4}$ example A=$[1,\alpha ]$, $b_i=[0,1]$ and $r_i=\alpha +1$, the true $\widetilde{T}_i(r_i)=\alpha +1$, while that shortcut gives0. The next sumcheck is needed to prove the weighted Boolean sum.

## 6. Points, messages and correlations

The64 original leaf-section claims are received after the product-tree reduction at one shared row point r. The inverse helper creates64 points r_i, one per bit, with no new randomness in this step. These points are correlated deterministic functions of r.

If r is uniform, each r_i individually is uniform because inverse Frobenius is bijective. This does not make the64 points mutually independent. Later fresh batching randomness has a separate role and cannot be replaced by pretending these points are independent.

Applying the same inverse automorphism to both sides of a field equality preserves both equality and inequality. A false scalar claim about a fixed $\widetilde{Q}_i(r)$ stays false about the corresponding fixed $\widetilde{T}_i(r_i)$. This exact transformation is separate from probabilistic checks in GKR, batching and the later selector sumcheck.

## 7. Actual code boundary before the next lesson

- common::frobenius_twist applies inv_frobenius to the ith leaf value and constructs the corresponding coordinatewise inverse point. It uses the coordinate-sequence reuse inspected in Lesson022. Both prover and verifier call this shared helper.
- The prover's phase3 receives one Claim{point,value} per bit, one large-field A table, and the original b words. SelectorMlecheckProver::new creates separate equality trackers for the per-bit points, keeps the common selected field table, and passes the words to BinarySwitchover. Its composition is selected·selector+(1−selector), equal to1+b_i(A−1).
- The constructor enforces64 claims, equal point dimensions and one weight per claim. Those weights come from a newly sampled six-coordinate $\gamma$ tensor. The detailed round kernel, switchover and batching analysis are next lesson; they have not been validated merely by reading this constructor.
- The verifier's verify_phase_3 closes each term using its own eq(r_i,r_new) factor and the received ith b evaluation with the one shared A evaluation. It also batches the C_lo·C_hi root-product claim. This confirms that the caller does not implement a single common selector table shortcut.

The six-coordinate $\gamma$ batching, its outer scalar batch and the recombination of b outputs are the next implementation step. This lesson establishes their correct mathematical inputs.

## 8. Sources and independent verification

Primary sources at the pinned revisions:

- [Blueprint source](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex), §4.5 Variable-Base Exponentiation: automorphism applied to equality weights and bit selectors; low-degree weighted sum.
- [IntMul common.rs](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/intmul/common.rs): frobenius_twist body and its abbreviated shared-P comment.
- [IntMul verifier](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/intmul/verify.rs): verify_phase_3, especially separate expected_selected_terms and one common gpow_a_eval.
- [IntMul prover](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/intmul/prove.rs): frobenius_twist call and phase3 selector claim construction.
- [selector_mle.rs](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/intmul/selector_mle.rs): constructor fields/preconditions and composition contract. The rest of the retained437-line source still needs its detailed round/fold walkthrough.

Independent checks cover8,320 exhaustive $\mathbb{F}_{4}$ table/point/twist cases across one and two variables,16,384 $\mathbb{F}_{16}$ cases,1,152 bit-selector cases, the exact worked claim and counterexamples to value-only/shared-table/off-cube shortcuts. For actual GHASH parameters, all64 bit leaves across four rows agree with the pointwise-Frobenius and weighted-selector identities. These are scalar mathematical checks, not execution of Rust, the selector kernel, transcript or full IntMul proof.

**Remember:** inverse-twist the value and every point coordinate; keep each bit's selector identity distinct.

Next: how the selector sumcheck combines64 correlated-point claims with six fresh batching coordinates and produces shared-point base/exponent claims.
