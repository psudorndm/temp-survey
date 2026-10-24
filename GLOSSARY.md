# Glossary

Definitions are deliberately short; later notes will supply the formal versions.

| Term | Meaning |
|---|---|
| Witness | Values the prover supplies to satisfy the public statement's constraints, including intermediate computation values. |
| Constraint system | A precise language for saying which assignments count as correct computations. |
| Binary field | A finite field of characteristic two, with 2^k elements. Field addition is XOR in a fixed binary basis. |
| Characteristic two | Adding the multiplicative identity to itself gives zero: 1 + 1 = 0. Consequently a + a = 0 for every element a. |
| Polynomial basis | Represents a field element by coefficients of 1, α, …, α^(k−1), with α satisfying a chosen irreducible degree-k polynomial. |
| Irreducible polynomial | A nonconstant polynomial that cannot be factored into two positive-degree polynomials over its coefficient field. |
| Polynomial reduction | Replacing a polynomial by its remainder on division by the defining polynomial. |
| GHASH field | Binius64's field F₂[X]/(X¹²⁸ + X⁷ + X² + X + 1), with 128-bit polynomial-basis representations. |
| Formal polynomial identity | Equality of polynomial coefficients, stronger than equality of their values on a restricted finite set. |
| Root bound | A nonzero univariate polynomial of degree d has at most d distinct roots over a field. |
| Challenge field | The field from which random verification challenges are drawn; it may be larger than the field containing the original witness values. |
| Soundness error | A bound on the probability that a dishonest prover can cause acceptance of a false claim, under the specified protocol and assumptions. |
| Binary tower | A nested sequence of binary extension fields. Central to original Binius; keep representations and current GHASH usage distinct. |
| Extension degree | The dimension of a larger field as a vector space over a chosen subfield. A degree-d extension represents each element by d subfield coordinates. |
| Basis packing | Combining several subfield coordinates into one extension-field element using a chosen basis. Preserves linear structure, not coordinatewise multiplication. |
| SIMD field packing | Storing multiple scalar field elements as independent lanes, whose arithmetic acts lane by lane. Distinct from basis packing. |
| Field embedding | An injective map preserving addition, multiplication, and 1; its image is a subfield of the target field. |
| Subfield | A subset of a field that is itself a field using the containing field's addition and multiplication. |
| Multilinear extension (MLE) | The unique polynomial of degree at most one in each variable matching a table on Boolean inputs. |
| Individual degree | The largest exponent of one chosen variable across the polynomial's monomials; differs from total degree. |
| Boolean hypercube | The set {0,1}ⁿ of n-bit index vectors, which labels a table of 2ⁿ entries. |
| Lagrange basis on the cube | Multilinear selector polynomials that equal one at one Boolean index and zero at all others. Table values are coefficients in this basis. |
| Folding a multilinear | Fixing one variable to a field value and storing the remaining polynomial's half-sized evaluation table. |
| Equality indicator | The multilinear polynomial eqₙ(x,y) = ∏ⱼ((1−xⱼ)(1−yⱼ)+xⱼyⱼ), which selects equal Boolean vectors and supplies MLE interpolation weights. |
| Sumcheck | A protocol reducing a claimed sum of polynomial evaluations over a Boolean cube to a random-point evaluation claim. |
| Sumcheck round polynomial | A univariate partial sum with prior challenge coordinates fixed and the other unbound coordinates summed over bits. |
| Terminal evaluation check | Authentication of the final polynomial value produced by a reduction; without it, sumcheck's internal consistency alone is insufficient. |
| Zerocheck | A reduction checking that a polynomial vanishes at every Boolean index, commonly using a random equality-weighted sum followed by MLE-check. |
| Deterministic zerocheck coordinates | Fixed weighting coordinates chosen so their equality weights form a basis over the errors' small coefficient field; permits injective grouping before randomizing over the remaining coordinates. |
| Friendly challenges | Binius code's fixed AES-field zerocheck coordinates, chosen for an injective bit grouping and efficient small-field calculations; distinct from fresh random sumcheck challenges. |
| MLE-check | A specialized sumcheck for Σᵤeq(τ,u)g(u), reducing the MLE evaluation of g's Boolean values to an evaluation of g at fresh challenge coordinates. |
| Constraint residual | The value measuring whether a constraint holds at one index; zero means the relation is satisfied there. |
| Reduction | A protocol step replacing one claim with another, with a stated soundness guarantee. |
| Polynomial commitment | A way to bind polynomial data and later verify claims about it; binding and hiding are separate properties. |
| Ring-switching | A reduction linking evaluation claims over small-field data to claims over a packed extension-field representation. |
| Fiat–Shamir | Replaces interactive verifier randomness with challenges derived from a transcript hash, under an appropriate security model. |
| Zero knowledge | The proof reveals no additional information about the witness beyond what the statement itself implies, in a formal simulation sense. |
| Word | A 64-bit unit in Binius64. A word is not automatically a field element with ordinary integer arithmetic. |
| Operand | In the Binius64 constraint language, an XOR combination of shifted value references. |
| Shifted value index | A reference to a word together with up to two fixed shift operations, evaluated inner then outer. |
| BitAnd constraint | A word relation A AND B=C, equivalent to 64 bitwise multiplication relations over F₂. |
| Constraint array | A column of operand values, one per constraint row, obtained by evaluating that operand's XOR/shift expression against prover data. |
| Bit axis | The coordinate identifying a bit position within a word; six Boolean coordinates for 64 positions in the baseline representation, one 64-point univariate coordinate in Binius64's oblong representation. |
| Oblong-multilinearization | Interpolation that is univariate of degree below the long-domain size on one axis and multilinear on the remaining Boolean axes. |
| Long axis | The larger evaluation domain replacing several Boolean coordinates; for Binius64's word bits, a 64-point additive subspace of the AES field. |
| Univariate-skip message | First-round polynomial data handling the long bit axis before row-variable rounds; the BitAnd encoding sends 64 off-domain values and prescribes 64 zeros. |
| Vanishing polynomial of a domain | W_D(Z)=∏_{d∈D}(Z−d); every polynomial zero on all distinct points of D is divisible by W_D. |
| Low-degree extension (LDE) | Re-evaluating the same bounded-degree polynomial on a larger or different domain after interpolation from the original domain. |
| Linearized polynomial | In characteristic two, a polynomial with only power-of-two exponents and no constant term; its evaluation map is F₂-linear. |
| Normalized subspace polynomial | s_j(Z)=W_j(Z)/W_j(β_j), zero on the j-dimensional prefix subspace and one at the next basis element. |
| Novel polynomial basis | N_i(Z)=∏_{j:bit_j(i)=1}s_j(Z), a basis with degree N_i=i used by the additive NTT. |
| Additive NTT butterfly | The invertible local update (a,b)→(a+tb,a+(t+1)b) associated with two sibling additive cosets. |
| Inverse additive NTT | The reverse-layer transform recovering novel-basis coefficients from domain evaluations, with inverse pair update b=u+v, a=u+tb. |
| Generic context image domain | The subspace returned by GenericOnTheFly::subspace(i), generated from normalized subspace-polynomial images; generally differs from the original basis's i-element prefix span. |
| Re-randomization of evaluation points | Rewriting old multilinear evaluations as equality-weighted sums, then using shared sumcheck challenges to reduce them to new evaluations at one point or its low-coordinate prefixes. |
| Powers batching | Combining ordered claims or summands with coefficients1,θ,…,θ^(m−1) for a fresh θ sampled after the original claims are fixed. |
| High-variable sumcheck padding | Multiplying a shorter summand by ∏(1−Xⱼ) on added high variables, preserving its Boolean sum and aligning different variable counts. |
| Byte LDE lookup | A precomputed linear transform image for each byte value and selected byte position; XOR combines images into a word's LDE. |
| Coset of an additive subspace | A translated set t+D={t+d:d∈D}; the skip message's output domain is the other coset of D inside D'. |
| Univariate Lagrange weight | Lᵢ(z)=∏ⱼ≠ᵢ(z−dⱼ)/(dᵢ−dⱼ), selecting table entry i at domain point dᵢ. |
| IntMul constraint | Unsigned 64-bit operand multiplication whose full 128-bit product is represented by constrained low/high words. |
| BinMul constraint | GHASH field multiplication, with each 128-bit operand represented by two word-valued operands. |
| Zero reduction | The reduction outputting d̂(z,r)=0 for the linear Zero-operand array, reusing fresh shared challenges and requiring no separate prover message or sumcheck. |
| Unified constraint point | The common low-to-high row point from the BitAnd/operand sumcheck, extended with fresh high coordinates when the Zero array is longer. |
| Matrix-padding scale | eq(0,r[ℓ..]), multiplying an ℓ-variable prefix evaluation to obtain the corresponding zero-padded matrix evaluation at the full point. |
| Operand-axis batching | The shift entry's random multilinear combination of14 operand claims using equality weights from4 fresh coordinates on a16-slot axis. |
| Multiplicative generator | A field element whose powers cycle through all nonzero field elements; in F₂^λ its order is2^λ−1. |
| Integer exponent encoding | Mapping an integern tog^n, preserving addition as multiplication and retainingn only modulo the order ofg. |
| IntMul wraparound alias | The false candidate2^128−1 for a zero integer product, indistinguishable by generator powers at128-bit field size and rejected by low-bit parity. |
| Fixed-base exponent limb | One16-bit chunk of a64-bit exponent in the inspected IntMul code; four limb lookups multiply to the fixed-base exponentiation result. |
| Selector polynomial | sel(s,v)=1+(v−1)s, choosing1 orv for Boolean selectors0 or1; off-cube it is a field polynomial value. |
| Variable-base exponentiation leaf | The Boolean-row value sel(bᵢ[x],f[x]^(2ⁱ)); multiplying64 such leaves givesf[x]^(integer b[x]). |
| Product-tree layer | A table whose entries multiply pairs in the child table; the inspected constructor pairs corresponding entries of low/high halves. |
| Multiplicative padding | Filling absent product factors with1 so they preserve the product; padded exponent0 gives all-one exponentiation leaves. |
| GKR product-tree layer reduction | Rewriting a parent MLE evaluation as an equality-weighted sum of child products, then reducing it to child evaluations by MLE-check. |
| Child line reduction | Combining two child-layer claims at(t,0) and(t,1) intoC(t,η)=u₀+(u₁−u₀)η after sampling a freshη. |
| Reduction soundness error | The probability that a false input claim is accepted or transported to true output claims by a reduction; later reductions must authenticate its remaining output claims. |
| Frobenius automorphism | φ(x)=x² in a binary field; an invertible map preserving XOR, multiplication and bits, with φ^m=id on F₂^m. |
| Inverse Frobenius | The square-root map φ⁻¹=φ^(m−1) on F₂^m, extended to repeated inverse twists by reducing iteration counts modulo m. |
| Fixed field | Elements fixed by a field automorphism; Fix(φ^d) in F₂^m has 2^gcd(d,m) elements. |
| F₂-linear map | A transformation preserving XOR and multiplication by bit scalars; represented by a binary matrix after choosing field bit coordinates. |
| MLE Frobenius twist | For Q_i[x]=φ^i(T[x]), the equivalent claim T̃(φ⁻ⁱ(r))=φ⁻ⁱ(Q̃_i(r)); transform every point coordinate and the value. |
| Bit-specific unsquared selector table | T_i[x]=1+b_i[x] (A[x]−1), whose pointwise Frobenius image is the ith variable-base exponentiation leaf. |
| Tensor claim batching | Weight a $2^k$-entry claim table by $\operatorname{eq}_k(\gamma,i)$ at a fresh point $\gamma\in K^k$; a fixed nonzero error vector is missed with probability at most $k/|K|$. |
| Selector switchover | Retain packed bits while calculating partial field evaluations from challenge-weighted lookups, then materialize a smaller field buffer after up to six bound row coordinates. |
| Infinity round sample | The leading coefficient of a fixed-degree round polynomial; used as an interpolation datum without choosing another finite evaluation node. |
| Exponent limb | A block of integer exponent bits; here $e=\sum_{j=0}^3 2^{16j}e_j$ with four16-bit blocks. |
| Fixed-base limb column | A row table $L_j[x]=(g^{2^{16j}})^{e_j[x]}$ obtained by gathering the relevant limb's power table. |
| Succinct public-table evaluator | A compact formula for the table MLE, such as $\widetilde T_0(Y)=\prod_j[1+(g^{2^j}-1)Y_j]$, allowing verifier evaluation without enumerating table rows. |
| Lookup index binding | Establishing that a looked-up value uses the address derived from the intended operand, rather than an arbitrary address whose table value is valid. |
| Weighted pushforward | $Y[v]=\sum_{x:I[x]=v}\operatorname{eq}(r,x)$; collect interpolation weights by address. |
| Lookup residue discrepancy | $R(C)=\sum_v\delta_v/(C-\iota(v))$; distinct address poles distinguish weighted buckets. |
| Fraction-addition gate | $(a,b),(c,d)\mapsto(ad+cb,bd)$, a polynomial representation of adding fractions. |
| Neutral fraction | $(0,1)$, used for missing instances and fraction-tree padding. |
| Address basis encoding | $\iota(v)=\sum_u\beta_uv_u$, an injective bit-linear map; it need not be a field embedding. |
| Factored BinMul endpoint | The terminal target equals $\widetilde A(r)\widetilde B(r)-\widetilde C(r)$ with no additional equality multiplier. |
| Shift routing indicator | Boolean predicate $S_v(i,j,s)$ saying output bit i copies input bit j at amount s. |
| Shift transpose weights | $\psi_j^\prime=\sum_i\psi_iS(i,j)$; push weighted output queries back toward witness bits. |
| Five-factor shift polynomial | Oblong bit weights times outer routing, inner routing, sparse wiring and bit witness; every coordinate occurs in at most two factors. |
| Deferred wiring claim | A claimed public wiring evaluation exported for later discharge; carrying it does not perform its check. |
| Shift key | A word/two-shift group carrying a flattened list of operand/constraint references; not a cryptographic key. |
| Additive sparse row list | Tagged row contributions that may share labels; their sum defines the logical table throughout folding. |
| Implicit shift-weight table | $h$ is evaluated through successive applications of a small transpose-shift operator, without storing its full $2^{24}$ entries. |
| Segmented product round | A quadratic selector round over public/hidden array pairs, computed without concatenating their buffers. |
| Exceptional extraction event | A zero divisor in an honest prover's quotient recovery; invert_or_zero does not recover the lost information. |
| Binary coordinate projection | $c_b:K\to\mathbb F_2$ extracts one basis coordinate; binary-linear, generally not $K$-linear. |
| Ring-switch auxiliary vector | $v_i=\sum_yw(i,y)\operatorname{eq}(z,y)$, exposing one field partial evaluation per packed bit position. |
| Binary basis transpose | $u_b=\sum_i\beta_i c_b(v_i)$; exchange the two axes of the binary coordinate matrix. |
| Row-batching functional | $L_\gamma(a)=\sum_b\gamma_b c_b(a)$, where $\gamma$ is an equality expansion of fresh row coordinates. |
| Ring-switch coefficient table | $A[y]=L_\gamma(\operatorname{eq}(z,y))$; gives a linear query on the packed oracle. |
| Vertical/horizontal field copies | $a\otimes1$ and $1\otimes b$ in $K\otimes_{\mathbb F_2}K$; distinct scalar actions agreeing on binary scalars. |
| Ring-switch tensor functional | $\Phi_\gamma(a\otimes b)=L_\gamma(a)b$; yields the correct off-cube MLE of the coefficient table. |
| Subset-sum row table | All256weighted sums for eight binary row-selection bits; one byte lookup replaces their conditional additions. |
| Deferred oracle relation | A queued target $\langle\pi,t\rangle=s$ awaiting the channel's final authenticated opening. |
| Per-oracle powers batch | Use one fresh $\lambda$ to combine several coefficient tables and targets for the same committed oracle. |
| Product-relation padding | Multiply the whole smaller product by $\prod_j(1-X_j)$; do not accidentally square this factor off the cube. |
| Coding polynomial | $f_\pi(Z)=\sum_j\pi[\operatorname{rev}(j)]N_j(Z)$, distinct from the message MLE. |
| Gao–Mateer basis | A binary basis with $b_0=1$ and $b_{j+1}^2+b_{j+1}=b_j$, fixing the additive code domain. |
| Interleaved codeword | Several independently encoded blocks stored one domain position at a time, with batch-index bit reversal accounted for. |
| BaseFold pair fold | Invert an additive-NTT butterfly, then interpolate its two recovered values at a fresh multilinear challenge. |
| Terminal consistency | Authenticate the final vector, match propagated query claims, require its fully folded cosets to agree and compare with the MLE-check scalar. |
| Codeword lift | Virtual adjacent duplication of encoded symbols, corresponding to high-coordinate zero-padding of a message. |
| High-coordinate repetition | Repeat a message block so its MLE is independent of added high coordinates; contributes no eq-zero factor. |
| First-fold routing | Early mask, outer oracle and later ordinary challenge groups; each oracle uses the appropriate suffix windows. |
| Coset leaf | A Merkle leaf holding all field symbols consumed together by an interleave or grouped FRI fold. |
| Shared Merkle layer | $2^h$ authenticated ancestor digests sent once, shortening each query's path by $h$ levels. |
| Query-test error factor | The configured $(1+\rho)/2$ factor used to choose the query count; separate from total protocol soundness. |
| FRI component size | Input and round roots, opened cosets/paths/layers and full terminal vector; excludes other BaseFold/Binius messages. |
