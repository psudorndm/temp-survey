# Lesson 041 — One challenge folds both the claim and its encoding

Introduced: 2026-10-23. Understanding unchecked. Implementation: Binius64 at 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** BaseFold keeps an algebraic evaluation claim and an encoded-data folding chain synchronized with the same fresh challenges. The final values must agree.

Prerequisites: [degree-one MLE-check](008-zerocheck-and-mle-check.md), [queued relations](039-linear-oracle-queues-and-relation-batching.md) and [bit-reversed coding](040-reed-solomon-encoding-and-bit-reversal.md).

This lesson derives the ordinary unbatched fold. The complete mixed-oracle lift/interleave geometry and Merkle authentication details remain subsequent lessons.

## 1. Fold the message's highest coordinate

Split the message into halves $\pi_0,\pi_1$. Fixing its highest multilinear coordinate to $\chi$ gives

$$
\pi_\chi[u]=(1-\chi)\pi_0[u]+\chi\pi_1[u].
\tag{1}
$$

The encoded vector must fold to $\operatorname{Enc}_{n-1}(\pi_\chi)$, rather than an arbitrary smaller vector.

By the novel-basis identity of Lesson040 and bit reversal,

$$
f_\pi(Z)=a(T(Z))+Zb(T(Z)),
\qquad T(Z)=Z^2+Z,
$$

where $a=f_{\pi_0}$ and $b=f_{\pi_1}$. The adjacent domain points $x,x+1$ have the same image $y=T(x)$. Their symbols are

$$
c_0=a(y)+xb(y),\qquad
c_1=a(y)+(x+1)b(y).
$$

Recover the two smaller-polynomial values by an inverse butterfly:

$$
b(y)=c_0+c_1,\qquad
a(y)=c_0+x(c_0+c_1).
$$

Then output

$$
c_\chi(y)=(1-\chi)a(y)+\chi b(y).
\tag{2}
$$

Equations (1)–(2) establish the exact commutation identity

$$
\operatorname{Fold}_\chi(\operatorname{Enc}_n(\pi))
=\operatorname{Enc}_{n-1}(\pi_\chi).
\tag{3}
$$

The field addition here is XOR; multiplication by $x$ and $\chi$ is extension-field multiplication.

## 2. A four-symbol example

Use $\mathbb F_4$, $\alpha^2=\alpha+1$, message $\pi=[1,\alpha]$, and inverse rate two. Its univariate coding polynomial is $1+\alpha Z$ and its codeword on $0,1,\alpha,\alpha+1$ is

$$
c=[1,\alpha+1,\alpha,0].
$$

For the first pair, $x=0$, so the recovered values are $(a,b)=(1,\alpha)$. For the second, $x=\alpha$, giving the same $(a,b)$. At $\chi=\alpha$ both folds equal zero:

$$
(1+\alpha)\cdot1+\alpha^2=0.
$$

The smaller message is the one-element table $[0]$, whose encoding is $[0,0]$.

Direct interpolation of the first encoded pair would instead give

$$
(1+\alpha)\cdot1+\alpha(\alpha+1)=\alpha,
$$

which is wrong. The inverse butterfly cannot be omitted.

[fold_pair/fold_chunk](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/fri/fold.rs#L16) perform exactly this inverse butterfly followed by extrapolate_line. The twiddle is the representative $x$ for the pair, obtained at absolute layer log_len minus one.

## 3. Move the claimed evaluation with the same challenge

Suppose the preceding relation sumcheck leaves a claim $\widetilde\pi(r)=s$. At a degree-one MLE-check round, let $\alpha$ be the highest remaining coordinate of the old point. The honest round polynomial is the message MLE with its lower coordinates fixed to the old point and its higher coordinates already fixed to new challenges:

$$
R(U)=a_0+a_1U,\qquad s=R(\alpha).
$$

The prover sends only $a_1$. The verifier recovers

$$
a_0=s-\alpha a_1,
\qquad
s_{\mathrm{next}}=s+(\chi-\alpha)a_1.
\tag{4}
$$

This recovery has no division, including at $\alpha=1$. After all rounds, the running scalar should equal the evaluation of the folded message, which (3) links to the folded codeword.

The fresh point is formed by reversing the high-to-low challenge sequence. It is not the old point $r$.

## 4. Concrete opening order

[verify_mlecheck_basefold](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/basefold/opening.rs#L43) handles any preceding mask/outer folds, then for each ordinary coordinate:

1. Receives the one MLE-check coefficient.
2. Processes any codeword commitment scheduled at this boundary.
3. Recovers the affine round and samples a fresh $\chi$.
4. Updates the scalar and adds $\chi$ to the codeword-fold challenge sequence.

[The prover counterpart](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop-prover/src/basefold/opening.rs#L48) sends coefficients before executing a scheduled fold/commit, then samples the shared challenge and advances both sides.

A root at a boundary binds a codeword produced from preceding challenges, before the next challenge. Multiple challenges may be grouped before a new root; there is not necessarily one Merkle commitment for every variable.

## 5. Query consistency and the terminal vector

After folding-phase commitments are fixed, [FRIQueryVerifier::verify](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/fri/verify.rs#L183) samples query indices, obtains initial virtual folded symbols, then follows each committed folding layer. At each step it checks that the propagated claim equals the selected opened symbol before applying the next coset fold.

The final committed vector can still have $2^t$ message degrees of freedom and length $R2^t$. It is sent in full and checked against its earlier root. Remaining $t$ fresh folding challenges are applied to each coset. The verifier checks:

- Every propagated query agrees with its selected terminal-vector entry.
- All $R$ fully folded coset values agree, giving one final field value.
- The final MLE-check scalar equals that field value.

The final equality is asserted inside verify_mlecheck_basefold. It is not left to an optional caller.

Merkle membership authenticates the symbols used in these equations. The root alone does not prove low degree; the randomized proximity/fold tests supply that part. A full FRI soundness theorem also needs proximity-gap and adversarial adaptivity analysis, not just the honest identity (3).

## 6. The honest-prover interpolation caveat reappears

Equation (4) is the verifier formula. The optimized [MultilinearEvalEvaluator](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip-prover/src/sumcheck/multilinear_eval.rs#L65) instead computes $R(1)$ and asks [RoundEvals::interpolate_eq](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip-prover/src/sumcheck/round_evals.rs#L119) to infer $R(0)$:

$$
R(0)=(s-\alpha R(1))(1-\alpha)^{-1}.
$$

The code uses invert_or_zero. At $\alpha=1$ this loses information. For the one-variable message $[1,0]$ and true old claim zero at one, the shortcut produces the zero polynomial instead of $1-U$. The verifier then obtains zero while the actual folded message is $1-\chi$, disagreeing unless $\chi=1$.

This is an algebraic reproduction of the inspected formula, extending the exceptional-coordinate issue in Lesson024 to this BaseFold evaluator. It is a possible honest-completeness failure at exceptional coordinates, not a demonstrated soundness attack. Rust reproduction and any sampler-level exclusion/retry policy remain unverified; the helper tests explicitly exclude $\alpha=1$.

## Validation and recall

[Independent model](../../work/verify_lessons039_041.py) checks1,160small-field fold/encode identities,15GHASH identities and36full degree-one point-moving/codeword chains. It also checks the numerical example and reproduces the exceptional shortcut case. No Merkle paths, Rust or serialized proofs were executed.

**Takeaway:** BaseFold needs both scalar reduction and authenticated codeword consistency, joined by the same challenges and a mandatory final equality.

**Recall:** why does interpolating two adjacent encoded symbols directly fail to implement a message MLE fold?
