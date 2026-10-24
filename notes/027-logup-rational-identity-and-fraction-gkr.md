# Lesson 027 — Prove the buckets with fractions

Introduced: 2026-10-23. Understanding unchecked. Code revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf.

**One idea:** each address gets its own rational-function pole. Equality of the rational functions forces the weighted bucket contents to match.

Prerequisites: [weighted pushforward](026-weighted-pushforward-for-indexed-lookups.md), [GKR layers](021-one-gkr-layer-and-line-reduction.md). This note separates the algebraic identity, the random test and the circuit proof.

## 1. Embed addresses without confusing them with integers

Let $\beta_0,\ldots,\beta_{m-1}$ be linearly independent binary basis elements in $K$. Embed address $v$ by

$$
\iota(v)=\sum_{j=0}^{m-1}\beta_j v_j.
$$

This is an injective map of bit vectors into field elements, preserving XOR. It does not in general preserve ordinary integer multiplication or define an embedding of a field of size $2^m$.

The table-address polynomial is public and linear:

$$
J(X)=\sum_{j=0}^{m-1}\beta_jX_j.
$$

Index-vector field entries use the same embedding, so $\overline I[x]=\iota(I[x])$.

## 2. The rational identity

For one looker with weights $w_x=\operatorname{eq}(r,x)$ and honest pushforward $Y$,

$$
\sum_x\frac{w_x}{C-\iota(I[x])}
=\sum_{v=0}^{M-1}\frac{Y[v]}{C-\iota(v)}.
\tag{1}
$$

This is an identity in the rational-function field $K(C)$: regroup the left terms by address. For multiple lookers, replace the left weights by $\gamma^j w_{j,x}$ and sum over $j$.

Now let $\widehat Y$ be the claimed buckets and $\delta_v=Y[v]-\widehat Y[v]$. Their discrepancy is

$$
R(C)=\sum_v\frac{\delta_v}{C-\iota(v)}.
$$

Multiply by the square-free address polynomial

$$
D(C)=\prod_v(C-\iota(v)).
$$

The numerator becomes

$$
N(C)=\sum_v\delta_v\prod_{u\ne v}(C-\iota(u)),
\qquad \deg N\leq M-1.
$$

At one address,

$$
N(\iota(v))
=\delta_v\prod_{u\ne v}(\iota(v)-\iota(u)).
$$

The product is nonzero because addresses are distinct. Therefore any nonzero bucket error makes $N$ nonzero. This argument works in characteristic two and does not rely on ordinary integer multiplicities surviving field reduction.

## 3. A fresh challenge tests that identity

The prover commits its buckets before the verifier samples $c\in K$. Conditional on the index map being fixed and in range, a nonzero discrepancy has at most $M-1$ non-pole roots. There are $M$ possible poles.

A conservative local union bound, treating all poles as exceptional, is

$$
\Pr[\text{pole or false identity passes}]
\leq\min\left(1,\frac{2M-1}{|K|}\right).
\tag{2}
$$

If $c$ were explicitly sampled outside the address set, the conditional root bound would instead use $(M-1)/(|K|-M)$. The inspected routine calls the ordinary channel sampler; no such exclusion is visible here.

For the IntMul table $M=65{,}536$, this contribution scales with the number of addresses, not merely its 16 address variables. Equation (2) is a local conservative bound, not the complete protocol error or a claimed concrete security level. Claim batching, GKR, openings, commitments, out-of-range indices and Fiat–Shamir analysis have separate obligations.

The module documentation writes a bound involving $n+m$ while defining $n,m$ as variable counts, including for denominator poles. That shorthand is not justified by the inspected identities. For example, an eight-entry table in $\mathbb F_{16}$ already gives eight possible table poles, regardless of a one-variable looker's length. This is a source-comment precision issue to carry into the full security accounting, not an established implementation attack.

If index field entries could lie outside the table's address image, the denominator support expands to those distinct values as well. The argument must then use that enlarged support or establish the index range through the later bit binding. Do not silently assume valid index rows in a full soundness proof.

## 4. Avoid division: build a fractional-addition circuit

Represent a fraction as its numerator/denominator pair $(a,b)$. Addition is

$$
(a_0,b_0)\oplus_{\mathrm{frac}}(a_1,b_1)
=(a_0b_1+a_1b_0,\ b_0b_1).
$$

No inverse is required to construct the circuit. The honest fraction equals the intended field ratio on the event that all denominator factors are nonzero.

For each looker the leaf pair is

$$
\left(\gamma^j\operatorname{eq}(r_j,x),\ c-\overline I_j[x]\right).
$$

The table-side pair is

$$
\left(\widehat Y[v],\ \iota(v)-c\right).
$$

Putting the negation in the table denominator subtracts the table rational sum. In a binary field subtraction equals addition, but the source writes the signs generically.

Each circuit combines its leaves using high-half pairing. A top circuit adds the instance root fractions. Its numerator should be zero; the prover sends only its denominator, and the verifier starts from the claim

$$
(N_{\mathrm{root}},D_{\mathrm{root}})=(0,d_{\mathrm{sent}}).
$$

The zero numerator is supplied by the verifier, not accepted as an unconstrained prover message.

## 5. One GKR layer for a pair of columns

For a parent numerator/denominator claim at point $\rho$,

$$
\begin{aligned}
N_{\mathrm{par}}(\rho)
&=\sum_x\operatorname{eq}(\rho,x)
\bigl(N_0(x)D_1(x)+N_1(x)D_0(x)\bigr),\\
D_{\mathrm{par}}(\rho)
&=\sum_x\operatorname{eq}(\rho,x)D_0(x)D_1(x).
\end{aligned}
$$

The verifier batches these two factored MLE-check claims using a fresh scalar. Their prime compositions have individual degree two. At the terminal point it receives four values, checks the batched numerator/denominator composition and only then samples one fresh line coordinate $\eta$.

It folds both numerator halves using the same $\eta$, folds both denominator halves with that $\eta$, appends $\eta$ to the newly randomized content point, and proceeds to the next layer. The final claim still requires authenticating the original leaf columns.

For IntMul there are 12 looker circuits and one table circuit. The top selector dimension is $\lceil\log_2 13\rceil=4$; the whole verifier call has $4+\max(\ell,16)$ GKR layers. A layer is not one sumcheck round: each layer runs a sumcheck over its current content variables.

## 6. Padding and final leaf checks

An absent instance holds the neutral fraction $(0,1)$, not $(0,0)$. Unequal-depth circuits also pad with neutral fractions.

With low-coordinate padding weight $\lambda=\operatorname{eq}(0^\nu,X_{\mathrm{pad}})$, the interpolated padded pair is

$$
(N,D)\longmapsto
\left(\lambda N,\ 1+\lambda(D-1)\right).
$$

Thus the denominator is one-padded; multiplying it by $\lambda$ would be wrong. Shorter lookers land at suffixes of the common node point, unlike the high-coordinate zero padding of ordinary unequal-size sumcheck from Lesson 017.

At the GKR leaves, the verifier:

- Evaluates looker numerators from public old points and $\gamma$ powers.
- Receives index evaluations to reconstruct $c-\widetilde{\overline I}$.
- Receives the table pushforward evaluation.
- Evaluates the table denominator $J-c$ itself.
- Applies each instance's padding map and checks selector interpolation of both leaf columns.

The inspected verifier does not separately reject a zero root denominator. The pole event therefore remains an explicit limitation of the fraction-as-rational interpretation and of a conservative error analysis; GKR itself checks polynomial relations even on that event.

## 7. Why the transparent table still needs an opening

The IOP wrapper binds one $Y$ oracle before $c$. After GKR it queues

$$
\langle Y,\operatorname{eq}_z\rangle=\widetilde Y(z),
\qquad
\langle Y,T\rangle=\sum_j\gamma^j e_j.
$$

Both relations use the same commitment. The second is the essential link to the lookup values; the bare transparent IP reduction does not inspect $T$ and leaves that relation unchecked for its caller.

For a succinct public table the verifier supplies the formula for $\widetilde T$. It can skip the generic table-side product sumcheck and open the two linear relations through the oracle channel instead. Deferred channel openings and returned index claims are still proof obligations.

With several tables, the actual body reuses a scalar's power series within each table but samples an independent pole challenge $c_t$ per table. Do not import documentation wording suggesting separate batching scalars where the function signature and body pass one shared $\gamma$.

## Sources and checks

- [IP lookup verifier](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip/src/logup_star/verify.rs): verify_reduction_transparent body, root claim, padding and leaf reconstruction.
- [Fraction verifier](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip/src/fracaddcheck.rs): degree-two batched MLE-check, four received values, shared line fold and pad_leaf_fraction.
- [Fraction circuit construction](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip-prover/src/fracaddcheck/circuit.rs): build and pop_layer bodies; high-half construction and retained buffers.
- [Lookup prover](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/ip-prover/src/logup_star/prove.rs): circuit construction, zero-numerator assertion, sent root denominator and index-claim extraction.
- [Committed wrapper](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/iop/src/logup_star.rs): two queued relations against the same oracle.
- [Logup* primary paper](https://eprint.iacr.org/2025/946): metadata/abstract only; PDF could not be retrieved. The proof above is independently derived, not a claim to have verified the paper's lemma.

Independent checks cover 768 two-looker random-point/challenge cases over $\mathbb F_{16}$, honest rational equality and combined high-half fraction roots. All 255 nonzero residue vectors with coordinates in raw values zero through three obey the four-address non-pole root bound; a case reaches three roots. An eight-address example verifies eight pole challenges in a16-element field. These are scalar identities, not a Rust transcript, commitment proof or complete soundness calculation.

**Remember:** distinct poles identify the buckets; GKR proves the fraction circuit; committed openings and index authentication finish the lookup.
