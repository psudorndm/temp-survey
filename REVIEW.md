# Recall cards

Orientation delivered: 2026-10-23. User understanding: not yet checked.

Try answering before reading the answer.

1. **What should a witness satisfy?** The constraints representing the public computation statement.
2. **Why is XOR natural in a binary field?** Coefficients add modulo two, with no carries.
3. **Does one AND of two packed words equal one extension-field multiplication?** No. Word AND is coordinatewise on bits; extension-field multiplication mixes coordinates.
4. **Does a sound proof automatically hide the witness?** No. Zero knowledge is an additional property.
5. **Which implementation is our working track?** Binius64; original Binius remains a separately labeled historical track.
6. **Has the whole codebase been reviewed?** No. The initial survey covered selected sources and orchestration.

## Lesson 001 — field arithmetic

Introduced: 2026-10-23. User understanding: not yet checked. Saved reference also explains irreducibility and inverses; those details have not been discussed interactively.

7. **What does `10` mean in the toy polynomial basis?** α, not the integer two under integer arithmetic.
8. **Why is addition XOR?** Each coefficient adds modulo two, with no carries between positions.
9. **With α² = α + 1, what is `10 × 10`?** `11`.
10. **What is `(α + 1)²`?** α, represented by `10`.
11. **What is α's multiplicative inverse?** α + 1, since α(α + 1) = 1.
12. **Why require an irreducible modulus?** It makes the polynomial quotient a field, ensuring every nonzero element has an inverse.
13. **Is F₄ integer arithmetic modulo 4?** No. They have different addition and multiplication; modulo 4, the nonzero element 2 has no inverse.

## Lesson 002 — random challenges

Introduced: 2026-10-23. User understanding remains unchecked.

14. **What was the answer to Lesson 001's final question?** `10 × 10 = 11`, because α² = α + 1.
15. **Can a nonzero polynomial vanish at both bits?** Yes: P(T) = T² + T.
16. **What is P(α) in our F₄?** 1.
17. **What is the basic root bound?** A nonzero degree-d univariate polynomial has at most d roots over a field.
18. **When is random identity testing useful?** When the discrepancy is fixed before a uniform challenge and its degree is small relative to the sampling field.
19. **Does a 128-bit field alone establish 128-bit protocol security?** No. Degrees, rounds, other reductions, and cryptographic terms must be accounted for.
20. **Does sumcheck alone authenticate its final evaluation?** No. That claim needs another check, such as a polynomial-opening protocol in the full construction.

## Lesson 003 — embeddings

Introduced: 2026-10-23. User understanding unchecked. The full toy mapping and algebraic verification are in the saved reference.

21. **Why fix the polynomial before the random challenge?** Otherwise it can be chosen as T − r and vanish at the known challenge.
22. **What must a field embedding preserve?** Addition, multiplication, and 1; it is injective and also preserves 0.
23. **Does copying small-field bits into a wider register always give an embedding?** No. It generally preserves XOR but may fail multiplication.
24. **When does F₂ᵐ embed into F₂ⁿ?** Exactly when m divides n.
25. **In our toy embedding, where does α go?** To β = u² + u, which satisfies β² = β + 1.
26. **What does the Binius conversion table do?** Converts AES polynomial-basis coordinates into compatible GHASH subfield coordinates.

## Lesson 004 — towers and packing

Introduced: 2026-10-23. Understanding unchecked; full multiplication derivation is in the saved note.

27. **What is an element of a quadratic extension of E?** A unique a₀ + a₁θ with two coefficients in E.
28. **If E has four elements, how many elements does its quadratic extension have?** Sixteen: four choices for each coefficient.
29. **Under θ² = 1 + αθ, what is the square of coordinate pair (0,1)?** (1,α).
30. **What if (0,1) represents independent SIMD lanes instead?** Its lane-wise square is (0,1).
31. **Is basis packing a field embedding of the coordinatewise product?** No. It is a linear coordinate map; extension multiplication mixes coefficients.
32. **How is one subfield value a embedded in this basis?** a → a + 0θ.
33. **Does packing remove information?** No. Two 2-bit coordinates still require four bits.

## Lesson 005 — multilinear extensions

Introduced: 2026-10-23. Understanding unchecked; saved reference includes uniqueness proof and implementation details.

34. **What is the MLE of the two-entry table [a,b]?** (1 − X)a + Xb.
35. **Is XY multilinear?** Yes; each variable has degree one, although the total degree is two.
36. **Is X² multilinear?** No; X has degree two.
37. **What indexes a table with 2ⁿ entries?** n Boolean coordinates; the MLE has n variables.
38. **Are table entries monomial coefficients?** Generally no. They are values on the Boolean cube (equivalently, Lagrange-basis coefficients).
39. **Can a bit-valued table's MLE evaluate to a non-bit?** Yes, at non-Boolean extension-field inputs.
40. **Does multiplying MLEs preserve multilinearity?** Not generally. X·X = X² is the simplest counterexample.
41. **Does one claimed MLE evaluation prove correctness by itself?** No; authentication and the surrounding protocol are still needed.

## Lesson 006 — weights and folding

Introduced: 2026-10-23. Understanding unchecked; detailed weight algebra is saved in the note.

42. **What was the previous recall answer for table [0,1]?** Its MLE is X.
43. **How is an endpoint pair (a,b) folded at r?** a + r(b − a), equivalently (1−r)a + rb.
44. **Which pairs bind Y first in [w₀,w₁,w₂,w₃], indexed x₀ + 2x₁?** (w₀,w₂) and (w₁,w₃).
45. **What does [1,0,1,1] become after Y=s?** [1,s].
46. **What is eq(x,y) in characteristic two?** 1 + x + y.
47. **Is eq a Boolean equality test at arbitrary extension-field vectors?** No; its Boolean selector meaning is guaranteed on Boolean inputs.
48. **How do folding and equality weights relate?** Both compute the same MLE evaluation, by repeated partial evaluation or one weighted sum.
49. **How much work does generic folding need for N entries?** O(N), with N−1 pair multiplications; the number of stages is log₂N.

## Lesson 007 — sumcheck

Introduced: 2026-10-23. Understanding unchecked; saved reference contains numerical transcript, soundness argument, and compressed message details.

50. **What does the first round polynomial sum over?** Every still-Boolean variable except the variable being bound that round.
51. **What connects consecutive rounds?** Rᵢ(0)+Rᵢ(1) equals the current claim; the next claim is Rᵢ at the newly sampled challenge.
52. **Does the prover choose its next polynomial before or after the previous challenge?** After the previous challenge, but before the next one.
53. **What remains after all sumcheck rounds?** A polynomial evaluation claim that must be authenticated.
54. **What is the basic error bound?** At most nd/|K| for n variables, individual degree at most d, under the stated independence and final-check assumptions.
55. **Why can the code omit one round coefficient?** The current endpoint-sum claim uniquely determines the missing highest coefficient.
56. **In characteristic two and degree bound one, what is the recovered slope?** The current sum claim.
57. **Is a field sum of three ones equal to three?** No; in characteristic two, 1+1+1=1.

## Lesson 008 — zerocheck and MLE-check

Introduced: 2026-10-23. Understanding unchecked; the factored protocol and recovery derivation are in the saved note.

58. **Does a zero sum imply every error is zero?** No. Errors 1 and 1 cancel in characteristic two.
59. **What random quantity checks the whole error table?** Its MLE E(τ)=Σᵤe(u)eq(τ,u), after binding the table before τ.
60. **What if some error is nonzero?** E is a nonzero multilinear; a random zero occurs with probability at most n/|K|.
61. **Must a polynomial vanishing on bits vanish at all field points?** No. X²+X vanishes at both bits but equals 1 at α in F₄.
62. **What is the factored MLE-check endpoint equation?** c=(1−τₖ)R(0)+τₖR(1).
63. **Which coefficient does MLE-check omit?** The constant coefficient, recovered as c−τₖΣⱼ≥₁aⱼ without division.
64. **Must zerocheck's terminal claimed value be zero?** No. It must equal the authenticated constraint polynomial at the fresh terminal point.

## Lesson 009 — native constraints

Introduced: 2026-10-23. Understanding unchecked; source details and examples are in the saved note.

65. **What is an operand?** An XOR of fixed shifted references to 64-bit words.
66. **Which shift runs first in a two-shift index?** The inner shift, followed by the outer shift.
67. **How does Zero express z=x XOR y?** Constrain x XOR y XOR z=0.
68. **How many bit relations does one word BitAnd express?** Sixty-four coordinatewise AND equations.
69. **What distinguishes IntMul and BinMul?** Full unsigned integer multiplication with carries versus GHASH extension-field multiplication.
70. **Does IntMul discard its high product word?** No; both low and high words are constrained.
71. **Are direct Constraint::verify methods the succinct external verifier?** No; they evaluate the relation against a full value vector.
72. **Does a shift inside an operand require a separate nonlinear constraint?** No, but its linear wiring still needs verification in the proof protocol.

## Lesson 010 — AND polynomial claims

Introduced: 2026-10-23. Understanding unchecked; source boundaries and optimized-path distinctions are in the note.

73. **What are the two axes of the conceptual AND table?** Constraint row and bit position.
74. **How many Boolean coordinates index 64 bit positions?** Six.
75. **What is the baseline constraint polynomial?** G=ãb̃+c̃, of individual degree at most two.
76. **Is G the same as the MLE of its Boolean residual table?** Generally no; only their Boolean values must agree.
77. **What remains after the terminal composition check?** Authentication of the claimed operand polynomial evaluations against witness expressions.
78. **Does deriving C=A AND B let a wrong circuit C operand pass?** No; the derived-C claim must still match the actual C operand through the later wiring check.
79. **What does actual Binius64 do to the six bit-axis variables?** Uses one univariate coordinate over a 64-point domain, while keeping row coordinates multilinear.

## Lesson 011 — the long axis

Introduced: 2026-10-23. Understanding unchecked; code and numerical details are saved in the note.

80. **What replaces the six bit-position variables?** One variable interpolated on a 64-point domain D.
81. **What is an operand polynomial's degree on that axis?** At most 63.
82. **What is the AND product's degree bound there?** At most 126.
83. **Is D the field F₂⁶?** No. It is a six-dimensional F₂-linear subspace inside F₂⁸; 6 does not divide 8.
84. **What do Lagrange weights do at domain point dⱼ?** Select exactly the j-th table value.
85. **After binding the long axis, what remains?** An ordinary multilinear in the row coordinates.
86. **Is fold(A AND B)=fold(A)·fold(B)?** Generally no; the code folds the derived AND bits separately.
87. **Why are on-domain challenges safe for the weight calculation?** Its prefix/suffix product formula never divides by the challenge minus a domain point.

## Lesson 012 — deterministic zerocheck

Introduced: 2026-10-23. Understanding unchecked; basis proof and implementation details are in the note.

88. **Why can e₀+αe₁ encode two bit errors without cancellation?** 1 and α are independent over F₂, so zero implies both bits are zero.
89. **How many bits can one AES-field value encode injectively?** Eight, using an F₂-basis.
90. **Which three fixed AES coordinates does the code use?** 0x02, 0x04, 0x10, representing β,β²,β⁴.
91. **Why are their eight equality weights independent?** They are an invertible tensor basis change from 1,β,…,β⁷.
92. **Which coordinates are fixed?** Up to three low constraint-row coordinates of the initial zerocheck weighting point.
93. **Does this remove three fresh sumcheck rounds?** No. Initial weighting coordinates and round challenges serve different roles.
94. **Can arbitrary field-valued errors be packed with the same no-cancellation guarantee?** No; the guarantee requires bit coefficients.
95. **What initial random-test error remains for ℓ≥3?** At most (ℓ−3)/|K|; this is only one term of full soundness accounting.

## Lesson 013 — skip message

Introduced: 2026-10-23. Understanding unchecked; exact degree reconciliation is in the saved note.

96. **What are the 64 transmitted values?** Evaluations of the claimed weighted residual polynomial on the upper half of a 128-point domain.
97. **Who supplies its lower-half values?** The verifier inserts 64 prescribed zeros on the bit domain.
98. **Does zero-prefix interpolation authenticate the witness relation?** No. A later random identity check and row sumcheck tie the message to actual operands.
99. **Which interpolation domain evaluates the message polynomial?** The 128-point superdomain D'. Operand bit folding uses D with 64 points.
100. **What are the honest and generic encoded degree bounds?** At most 126 for the true product polynomial, at most 127 for an arbitrary received message under the inspected encoding.
101. **Which additional relation forces encoded degree at most 126?** The 64 transmitted values sum to zero, on this additive-domain format.
102. **Why must z be sampled after receiving the message?** So the discrepancy polynomial is fixed before the random identity test.

## Lesson 014 — message kernel

Introduced: 2026-10-23. Understanding unchecked; kernel formulas and memory figures are in the note.

103. **What does NTTLookup::ntt actually return?** The 64 off-domain AES evaluations of the degree-below-64 polynomial specified by a word's bit evaluations.
104. **Why can a word's transform be assembled from byte lookups?** The LDE is F₂-linear.
105. **How much LDE-table field payload is stored?** 32 KiB: two byte-position parity tables, with the others recovered by lane permutation.
106. **Why are eight rows aggregated in AES?** Their fixed equality weights lie in AES, so the inner residual sum stays in that small field.
107. **What does B8ToExtMulMap precompute?** t → λ·ι(t), combining AES-to-large-field embedding with multiplication by a reused large-field weight.
108. **Are the four fixed-large-challenge values deterministic?** No; the count is fixed at four, but their values are fresh challenges.
109. **Why repeat a short logical axis in an internal window?** So artificial fixed coordinates sum to a factor of one instead of introducing an unwanted factor 1−ρ.
110. **Does all message computation stay in AES?** No; large-field maps, accumulation and outer window weights remain.

## Lesson015 — additive NTT foundations

Introduced: 2026-10-23. Understanding unchecked; the saved reference contains normalization/basis/butterfly derivations.

111. **Why can't F₂ᵐ contain a primitive64th root of unity?** Its nonzero multiplicative group has odd order2ᵐ−1.
112. **What does W_j vanish on?** The span of the first j independent domain-basis elements.
113. **What is its recurrence?** W_{j+1}(Z)=W_j(Z)(W_j(Z)+W_j(β_j)).
114. **Why is W_j constant on a coset a+V_j?** It is additive and vanishes on V_j, so W_j(a+v)=W_j(a).
115. **Why divide by W_j(β_j)?** So normalized sibling-coset values differ by one.
116. **What is the novel basis polynomial N_i?** The product of normalized subspace polynomials selected by the binary digits of i; it has degree i.
117. **What is the forward butterfly?** u=a+tb, v=u+b.
118. **What is its inverse?** b=u+v, a=u+tb; no division is needed.
119. **Does forward NTT accept monomial coefficients?** Its contract accepts novel-basis coefficients.

## Lesson 016 — four-point NTT and LDE

Introduced: 2026-10-23. Understanding unchecked.

120. **Which pairs does the four-point reference transform process first?** (0,2) and (1,3), followed by adjacent pairs (0,1) and (2,3).
121. **What do [1,α,α+1,1] mean in the worked input?** Coefficients of (1,Z,W,ZW), with W=Z²+Z; they are not monomial coefficients.
122. **What is the worked forward output?** [1,α+1,α+1,0], evaluated at [0,1,α,α+1].
123. **How does inverse undo a butterfly?** First b=u+v, then a=u+tb, using the recovered b; reverse the layer order.
124. **What gets zero-padded in the BitAnd LDE?** Novel coefficients recovered by the inverse transform, not the input evaluations.
125. **Why do the 64- and 128-point contexts preserve the same polynomial?** They are built from compatible prefix bases, so their first 64 novel polynomials agree.
126. **Can a shorter transform reuse a larger generic context without changing the domain?** Not generally; the effective domain can be a transformed image subspace.

## Lesson 017 — batching and re-randomization

Introduced: 2026-10-23. Understanding unchecked.

127. **How does an evaluation become a sumcheck claim?** M(ρ)=Σ_{x∈{0,1}ⁿ}eq(ρ,x)M(x), by multilinear interpolation.
128. **What allows different old points to share one sumcheck?** Give each term its own equality indicator, then batch the complete summands.
129. **Does point switching preserve the claimed value?** Generally no; it proves a relation leading to a new evaluation claim at the new point.
130. **Why use powers of a random θ?** Fixed errors form a nonzero polynomial in θ, whose accidental cancellation probability is bounded by its degree divided by the field size.
131. **When must the old claims be fixed?** Before sampling θ; per-bit claims must also be fixed before the bit-axis challenge z used to fold them.
132. **Why are the actual combined round polynomials cubic?** The quadratic BitAnd residual is multiplied by a linear equality indicator.
133. **How do shorter summands join the batch?** Multiply by the all-zero indicator on extra high variables; this preserves their Boolean sum and leaves output points on low-coordinate prefixes.
134. **Does the closing scalar equation certify every sent evaluation?** No; the resulting claims must be authenticated against the witness downstream.
135. **Does the pinned prover implement the spec's two-table optimization?** No; it keeps separate operand columns/evaluators and combines their round polynomials in the driver.
136. **What factor does MleToSumCheckDecorator actually restore?** eq(α,X), equal to1+α+X in characteristic two; comments describing X−α are imprecise.

## Lesson 018 — Zero reduction

Introduced: 2026-10-23. Understanding unchecked.

137. **Why can Zero skip a sumcheck?** Its linear residual is directly the interpolant of a zero table, so validity means that polynomial is identically zero.
138. **What value does the prover send for Zero?** None; the claimed evaluation is the known constant0, enforced by later witness authentication.
139. **Which row point does Zero reuse?** The fresh output challenges of the batched sumcheck, not the initial weighting point with deterministic coordinates.
140. **What if Zero has more row variables than the shared sumcheck?** Append fresh high coordinates for exactly the shortfall; no extra sumcheck rounds.
141. **What is the local random-evaluation bound?** At most(63+ℓ_zero)/|K| for a fixed nonzero oblong residual and independent uniform coordinates, capped at1.
142. **Why can several reductions reuse one random point?** Each local bound needs the point's uniform marginal distribution; their bad events need not be independent.
143. **How is a prefix evaluation lifted to the full padded matrix?** Multiply by eq(0,high_coordinates), the product of1−r_j over the added row coordinates.
144. **How many columns enter the shift claim interface?** Fourteen, ordered Zero1, BitAnd3, IntMul4, BinMul6; absent families occupy zero slots.
145. **How does the actual shift entry batch those columns?** Four fresh coordinates produce16 equality weights; the first14 weight the real slots, with two zero slots.

## Lesson 019 — IntMul exponent encoding

Introduced: 2026-10-23. Understanding unchecked.

146. **How does IntMul encode its integer relation?** (g^a)^b=(g^(2^64))^c_hi·g^c_lo for a public multiplicative generatorg.
147. **What does equality of generator powers imply?** Equality of integer exponents modulo the full group order2^λ−1.
148. **What is the sole false case atλ=128 with64-bit words?** Actual product0 versus candidate2^128−1, whose two output words are all ones.
149. **Why does one low-bit check remove that case?** The zero product is even, while the false maximal output is odd; requirea₀b₀=c_lo,₀.
150. **Why mustg be a generator rather than merely nonzero?** A smaller-order element introduces more exponent collisions; parity alone need not remove them.
151. **Is the high-half base the raw field representation of2^64?** No; it isg^(2^64), obtained by64 repeated field squarings.
152. **Does the verifier demand parity residual0 at the random off-cube output point?** No; it checks its equality-weighted contribution against the batched sumcheck target.
153. **What fixed-base optimization does the inspected code use?** Four16-bit lookup limbs per word, with limb product trees and logup*, instead of the paper's64 bit-selector leaves.
154. **Does exponentiating a row table then interpolating equal exponentiating its MLE value?** Generally no; the exponentiation table's MLE can even be0 off the cube while every Boolean-row value is nonzero.
155. **What is the actual final IntMul array wire order versus downstream order?** Wirea,c_lo,c_hi,b; operand claimsa,b,c_lo,c_hi.

## Lesson 020 — bit-selected powers and product trees

Introduced: 2026-10-23. Understanding unchecked.

156. **How does the exponent's binary expansion give a field product?** f^b=∏ᵢ(f^(2ⁱ))^bᵢ, because b=Σᵢ2ⁱbᵢ as an integer.
157. **What is the selector polynomial?** sel(s,v)=1+(v−1)s; a zero-valued bit chooses1 and a one-valued bit choosesv.
158. **What leaves represent exponent13?** [f,1,f⁴,f⁸], with bits ordered least-significant first.
159. **How does the actual four-leaf tree pair entries?** Across halves: (0,2),(1,3), yielding[f⁵,f⁸] for exponent13, then their productf¹³.
160. **Where is bit-indexi,rowx stored?** At scalar offsetiN+x; row coordinates are low and leaf-index coordinates high.
161. **Why are padded exponentiation rows filled with1?** The padded exponent is0; every bit chooses the multiplicative identity.
162. **Do six tree levels mean six sumcheck rounds?** No; the single-tree reduction uses MLE-check sizesℓ,…,ℓ+5, totaling6ℓ+15 rounds plus six line challenges.
163. **Does a consistent product tree prove that its leaves encode the witness exponent?** Not by itself; separate leaf-discharge and witness authentication are needed.
164. **Can the leaf MLE be replaced by a selector applied to input MLEs?** Generally no; multiplication and interpolation do not commute off the Boolean cube.

## Lesson 021 — GKR layer and line reduction

Introduced: 2026-10-23. Understanding unchecked.

165. **What is the parent evaluation identity?** P̃(ρ)=Σ_{x∈{0,1}ⁿ}eq(ρ,x)A(x)B(x), where the parent table contains pointwise child products.
166. **Why not checkP̃(ρ)=A(ρ)B(ρ) directly?** The multilinear interpolation of pointwise products generally differs from the off-cube product of child MLEs.
167. **What does the actual layer's MLE-check leave?** A new targetv=A(t)B(t) at fresh content pointt, using factored degree-two rounds.
168. **What follows the product check on received child values?** Sample freshη and formw=u₀+(u₁−u₀)η, producing the single child-layer claimC(t,η)=w.
169. **Why can those two child claims combine along an affine line?** Their points agree on all content coordinates and differ only in one multilinear selector coordinate.
170. **What is the next point?** The fresh MLE-check pointt followed byη; not the old parent pointρ followed byη.
171. **When must child values be fixed?** Beforeη; the code receives both and checks their product before sampling it.
172. **What is the line-combination error bound?** At most1/|K| for a fixed nonzero child-error pair, because its line error is nonzero of degree≤1.
173. **Does satisfying the child product equation authenticate each child value?** No; authenticity is transferred to the next child-layer claim and ultimately the leaves/witness.
174. **What is a conservative isolated layer bound?** (2n+1)/|K| for n content variables, including degree-two MLE-check and line reduction, assuming later authentication.

## Lesson 022 — Frobenius automorphism and fixed fields

Introduced: 2026-10-23. Understanding unchecked.

175. **Why does squaring preserve addition?** (x+y)²=x²+2xy+y²=x²+y² because 2=0.
176. **Over which scalars is Frobenius linear?** F₂; generally φ(cx)=c²φ(x), so it need not be linear over the whole extension field.
177. **What does squaring do to F₄ bit coordinates?** (a₀,a₁)→(a₀ XOR a₁,a₁) in basis (1,α), with α²=α+1.
178. **Why is squaring invertible?** Equal squares imply equal field elements; an injective map on a finite field is bijective.
179. **What is inverse Frobenius in F₂^m?** φ⁻¹=φ^(m−1); iteration counts reduce modulo m.
180. **Is inverse Frobenius multiplicative inversion?** Generally no; in F₁₆, sqrt(u)=u²+1 while 1/u=u³+1.
181. **How does inv_frobenius_sequence obtain inverse twists?** Construct the forward cycle through φ^m(x)=x, reverse it and keep the requested prefix.
182. **What is fixed by φ^d inside F₂^m?** A subfield of size 2^gcd(d,m); when d|m it is the unique 2^d-element subfield.
183. **Why is there no F₆₄ subfield inside AES?** Degree6 does not divide8; Fix(φ⁶) has only4 elements there.
184. **Does a unique subfield mean a unique abstract embedding map?** No; the image subfield is unique, but multiple embedding maps can have that image.

## Lesson 023 — Frobenius twists of MLE claims

Introduced: 2026-10-23. Understanding unchecked.

185. **What is the MLE twist identity for Q_i[x]=φ^i(T[x])?** φ⁻ⁱ(Q̃_i(r))=T̃(φ⁻ⁱ(r)), with the map applied coordinatewise to r.
186. **Why must the point move?** Inverse Frobenius transforms the interpolation weights: φ⁻ⁱ(eq(r,x))=eq(φ⁻ⁱ(r),x) for Boolean x.
187. **What does the worked claim Q̃(α)=α become?** T̃(α+1)=α+1 for T=[1,α], Q=[1,α+1] in F₄.
188. **Which table underlies bit i after inverse twisting?** T_i[x]=1+b_i[x] (A[x]−1), the unsquared bit-specific selector table.
189. **Why can the exponent bit stay unchanged under the twist?** Its Boolean-row values0 and1 are fixed by every Frobenius power.
190. **Do all64 twisted leaves refer to one identical selector table?** Generally no; they share A but have distinct bit selectors b_i. The helper's shared-P comment is abbreviated for this caller.
191. **Can T̃_i(r_i) be checked by plugging input MLE values into the selector?** Generally no; use the equality-weighted Boolean bilinear sum, which needs the next sumcheck.
192. **Does this deterministic twist add a soundness error?** No, provided the pointwise table relation holds; the bijective transformation preserves true and false claims exactly.
193. **Are the64 twisted points independent?** No; they are correlated functions of one old point, though each is individually uniform when that point is uniform.

## Lesson 024 — Selector batching and common-point outputs

Introduced: 2026-10-23. Understanding unchecked.

194. **How can six challenges weight 64 claims?** Treat their indices as six bits and use $w_i=\operatorname{eq}_6(\gamma,i)$.
195. **Are these 64 independent random coefficients?** No; they are tensor weights determined by six independent fresh coordinates and sum to one.
196. **What is the local initial batching error bound?** At most $6/|K|$ for a fixed nonzero 64-entry error vector, by the total degree of its MLE.
197. **Why is the shared sumcheck degree three?** Each term is an equality indicator times a bilinear selector or two-column product: individual degree at most $1+2=3$.
198. **What does the closing check authenticate by itself?** Consistency with its target, conditional on later authentication of the sent column values; it does not independently certify every bit/base/output value.
199. **Why receive bit values before sampling the recombination point?** It fixes their error vector before fresh six-coordinate weighting. An adaptive vector could arrange cancellation.
200. **Do packed selectors remain bits through six folds?** No; after the first fold their partial values generally lie in the large field. Packed bit storage can persist while lookups calculate those field values.
201. **What does the infinity sample mean?** The leading coefficient of the quadratic prime round, not evaluation at a field element named infinity.
202. **What assumption does the missing-constant interpolation need?** $\alpha\ne1$ to divide by $1-\alpha$; concrete sampler policy and exceptional-case handling remain to be traced.

## Lesson 025 — Fixed-base limbs and the public power table

Introduced: 2026-10-23. Understanding unchecked.

203. **What does fixed-base mean here?** The public base $g$ is shared; the integer exponent still varies by row.
204. **How does a64-bit exponent split?** $e=e_0+2^{16}e_1+2^{32}e_2+2^{48}e_3$, with each $e_j$ a16-bit integer.
205. **What are the four leaves?** $T_j[e_j]=(g^{2^{16j}})^{e_j}$; their product is $g^e$.
206. **Why are there12 limb columns but eight distinct tables?** Three word families have four limbs; a and c_lo share tables0–3, while c_hi uses4–7.
207. **Why does c_hi use a shifted base?** Its integer contribution is $2^{64}c_{\mathrm{hi}}$, so the base is $g^{2^{64}}$.
208. **How are zero exponent rows padded?** Each limb indexes row zero, whose power-table value is one; every padded tree root is one.
209. **How can eight power tables share one lookup table?** $T_s[k]=\varphi^{16s}(T_0[k])$; inverse-twist both each claimed value and its point coordinates.
210. **What is the shared table's MLE?** $\widetilde T_0(Y)=\prod_{j=0}^{15}[1+(g^{2^j}-1)Y_j]$; the verifier evaluates it in work linear in16 without expanding the table.
211. **Does a valid product tree establish correct lookup addresses?** No; lookup index claims must be tied to the original exponent limbs and ultimately authenticated witness bits.
212. **Does transparent table evaluation mean no lookup commitments?** No; the public table itself needs none, while lookup-related auxiliary oracles may be committed.

## Lesson 026 — weighted pushforward

Introduced:2026-10-23. Understanding unchecked.

213. **What is the weighted pushforward?** $Y[v]=\sum_{x:I[x]=v}\operatorname{eq}(r,x)$.
214. **What exact identity does it give?** $\widetilde{(T\circ I)}(r)=\langle T,Y\rangle$.
215. **Are buckets integer multiplicities?** No; they sum field-valued interpolation weights.
216. **How are12lookers batched here?** With powers $1,\gamma,\ldots,\gamma^{11}$ of one fresh scalar, not six-coordinate tensor weights.
217. **What must precede the pole challenge?** The commitment to the gamma-dependent pushforward.
218. **Does an honest scatter bounds check prove malicious-prover index range?** No; later index/bit authentication is still needed.

## Lesson 027 — rational lookup proof

Introduced:2026-10-23. Understanding unchecked.

219. **Why do distinct poles identify the buckets?** After clearing denominators, $N(\iota(v))=\delta_v\prod_{u\ne v}(\iota(v)-\iota(u))$.
220. **What size governs the conservative pole bound?** The number of distinct address values, at most $M=2^m$ for in-range indices, rather than just m variables.
221. **What are fraction-addition gates?** $(a,b)+(c,d)$ is represented by $(ad+cb,bd)$.
222. **Which fraction pads an absent instance?** $(0,1)$; interpolated padding maps $(N,D)$ to $(\lambda N,1+\lambda(D-1))$.
223. **What does the root verifier supply itself?** The zero numerator; it receives the denominator and checks circuit consistency.
224. **What two committed relations close a transparent table?** $\langle Y,\operatorname{eq}_z\rangle=\widetilde Y(z)$ and $\langle Y,T\rangle=E$, both against the same Y commitment.

## Lesson 028 — IntMul index binding

Introduced:2026-10-23. Understanding unchecked.

225. **How does an embedded limb index depend on word bits?** $\overline I_{t,l}=\sum_{u=0}^{15}\beta_u e_{t,16l+u}$; basis indices reset within each limb.
226. **Is address encoding a small-field arithmetic embedding?** Generally no; it preserves XOR and bit coordinates, without requiring multiplicative closure.
227. **Why sample four rho coordinates?** The12index claims are zero-padded to16 and weighted by $\operatorname{eq}_4(\rho,j)$.
228. **Which three claims enter the last IntMul batch?** Folded index claim, parity zero claim, recombined exponent-b claim, in that order.
229. **Must the parity composition vanish off the cube?** No; its weighted Boolean sum is zero, and its terminal composition matches the sumcheck target.
230. **Are final raw bit arrays already witness authenticated?** No; the output carries common-point claims into rerandomization and shift/witness checks.

## Lesson 029 — BinMul

Introduced:2026-10-23. Understanding unchecked.

231. **Why can BinMul avoid exponent encoding?** The native relation already is GHASH-field multiplication.
232. **How are high-word bits packed?** With basis positions64–127, unlike independently reset lookup-limb bases.
233. **What is the composition degree?** $\widetilde A\widetilde B-\widetilde C$ has individual degree at most two.
234. **Does the BinMul terminal check multiply by another equality indicator?** No; factored MLE-check has already handled that factor.
235. **Can valid rows produce a nonzero terminal composition?** Yes; for example $G(X)=\alpha(X+X^2)$ vanishes at bits but $G(\alpha)=\alpha$.
236. **How are absent rows padded?** Packed operands A,B,C are all zero, satisfying0 times0 equals0; six word claims remain for later authentication.

## Lesson 030 — shift routing

Introduced: 2026-10-23. Understanding unchecked.

237. **What does each shift-indicator row select?** At most one input bit, or no bit for zero fill.
238. **How do two shifts compose?** $\sum_{k,j}S_2(i,k)S_1(k,j)w_j$; k is a summation index, not a new witness word.
239. **How is the indicator evaluated without enumerating all triples?** A two-state carry recurrence checks bit-index addition, then derives left/right/arithmetic/rotate variants.
240. **How do32-bit variants keep the halves separate?** Use lower-five-bit routing and multiply by $\operatorname{eq}_1(I_5,J_5)$; the top amount bit is ignored.
241. **Which direction do output weights move?** By the transpose: $\psi_j^\prime=\sum_i\psi_iS(i,j)$.
242. **Are non-Boolean shift amounts machine shift instructions?** No; the verifier evaluates a multilinear extension of the Boolean routing predicate.

## Lesson 031 — five-factor wiring proof

Introduced: 2026-10-23. Understanding unchecked.

243. **Why is the product degree two rather than five?** Every coordinate appears in only two multilinear factors.
244. **How many shift sumcheck rounds are there?** $36+(L+1)$ for a common segment of $2^L$ words; the extra word coordinate selects public/private.
245. **Which shift slot is bound first?** Outer variant/amount, then inner variant/amount, then witness/intermediate/output bit axes and the word axis.
246. **How is the trace value reconstructed?** $w^\star=(1-\eta)p^\star+\eta h^\star$, using padded public and private evaluations.
247. **Does receiving a wiring scalar check it?** No; the returned WiringEvalClaim must be discharged against the correct public wiring function.
248. **Does a short proof force a logarithmic-time verifier?** No; direct sparse wiring evaluation reads the circuit terms and can cost circuit-size work.

## Lesson 032 — sparse keys

Introduced: 2026-10-23. Understanding unchecked.

249. **What identifies a shift key?** One word under one inner/outer sequence, spanning all operand columns that reference it.
250. **What coefficient does the key accumulate?** $\kappa_{y,q}=\sum_{(m,x)\text{ in key}}\eta_m\xi_x$.
251. **How does the key contribute to g?** Add $\kappa_{y,q}$ at every set bit of word y.
252. **Why preserve operand-column order?** It groups row weights before multiplying by each column weight.
253. **Is a dense sequence ID a cube address?** No; decode it to outer-major canonical sequence bits before folding.
254. **Must public/hidden rows at the same address be merged immediately?** No; duplicate stored rows add linearly.

## Lesson 033 — implicit weight tables

Introduced: 2026-10-23. Understanding unchecked.

255. **What full table is avoided?** $h(J,q_1,q_2)$ has $2^{24}$ field entries.
256. **What permits the small-table strategy?** Transpose shifting is linear: $T[(1-r)a+rb]=(1-r)T[a]+rT[b]$.
257. **Which round samples are computed?** $R(1)$ and the leading coefficient $R(\infty)$, with the claim supplying $R(0)$.
258. **Does sparse folding halve the row-list length?** No; it scales/relabels every row and halves the logical address space.
259. **When does the standard dense bit prover take over?** After all shift-slot coordinates bind and stored rows collapse to one64-entry row.
260. **How do intermediate bit stages avoid dividing by a carried scalar?** Retain and reevaluate the original unscaled weights at the new point.

## Lesson 034 — segment folding

Introduced: 2026-10-23. Understanding unchecked.

261. **What are the segment selector round values?** $R(1)=\langle H,M\rangle$, $R(\infty)=\langle P+H,Q+M\rangle$.
262. **How is the public-prefix correction used?** Replace the hidden prefix contribution inside the dense leading-coefficient pass by the correct cross composition.
263. **Which buffer is reused?** The hidden buffer becomes $\eta H+(1-\eta)P_{\mathrm{padded}}$ after selector binding.
264. **Does a division-free verifier imply a division-free honest prover?** No; current phase2 extracts private/wiring values by quotient formulas with invert_or_zero.
265. **What can happen at zero extraction factors?** The quotient loses information, potentially causing honest rejection by later witness or wiring authentication; no soundness attack is established.
266. **Is the segment coordinate included in the private ring-switch point?** No; the caller removes it, leaving $r_J\Vert r_Y$.

## Lesson 035 — packing and scalar linearity

Introduced: 2026-10-23. Understanding unchecked.

267. **Does one GHASH element lose information about its128bits?** No; coordinates in a fixed binary basis give a bijection.
268. **Why not extract bits after evaluating the packed MLE?** Coordinate extraction is binary-linear but generally not linear over the challenge field.
269. **Give a tiny failure of field-linearity.** In $\mathbb F_4$, $c_0(\alpha)=0$ but $\alpha c_0(1)=\alpha$.
270. **What occupies the first seven ring-switch coordinates?** Six within-word bit coordinates and the lowest word-index coordinate.
271. **How many64-bit words fit one packed field element?** Two, with a missing final word filled by zeros.
272. **Is the public/private selector part of the private ring-switch point?** No; it was removed before this call.

## Lesson 036 — transpose and fresh batching

Introduced: 2026-10-23. Understanding unchecked.

273. **What vector does the prover send?** $v_i=\sum_yw(i,y)\operatorname{eq}(z,y)$, one field value per packed bit position.
274. **What is transposed?** The binary coordinate matrix of that field-valued vector.
275. **What does the transpose establish?** $u_b=\sum_y\pi[y]c_b(\operatorname{eq}(z,y))$.
276. **What must precede the fresh row challenge?** Receiving the entire auxiliary vector, so it cannot adapt to that challenge.
277. **How many fresh row coordinates are sampled for GHASH?** Seven; they index128rows, independently of the packed-table dimension.
278. **What does the reduction output?** A packed linear relation $s'=\sum_y\pi[y]A[y]$, still requiring oracle authentication.

## Lesson 037 — tensor equality indicator

Introduced: 2026-10-23. Understanding unchecked.

279. **Why is $L_\gamma(\operatorname{eq}(z,p))$ generally wrong?** It moves field-valued interpolation weights through a map that is only binary-linear.
280. **What are the two field copies?** Vertical $a\otimes1$ and horizontal $1\otimes b$ inside $K\otimes_{\mathbb F_2}K$.
281. **How is horizontal scaling implemented?** Transpose, scale vertically, then transpose back.
282. **What equality product does eval_rs_eq compute?** $\prod_j(1+z_j\otimes1+1\otimes p_j)$.
283. **Which final functional yields the coefficient MLE?** $\Phi_\gamma(a\otimes b)=L_\gamma(a)b$.
284. **Is the tensor algebra a field?** No; in the toy example a nonzero $e$ satisfies $e(e+1)=0$.

## Lesson 038 — kernels and public closure

Introduced: 2026-10-23. Understanding unchecked.

285. **What replaces eight conditional row additions?** A byte indexes one of256precomputed subset sums of eight weights.
286. **How large is the logical low-factor table payload?** $16\cdot256\cdot16=65{,}536$ bytes.
287. **When does projection happen in fused A generation?** After multiplying the low and high equality factors in the field.
288. **Why can the public product endpoint be checked directly?** The verifier knows the public packed table and computes the transparent indicator.
289. **What happens for one or two public words?** The packed-index sumcheck has zero rounds; the seven-coordinate ring batch remains.
290. **What still authenticates the private relation?** The committed linear-oracle machinery, including its BaseFold/codeword/Merkle checks.

## Lesson 039 — relation queues and batching

Introduced: 2026-10-23. Understanding unchecked.

291. **Does verify_oracle_relation authenticate a BaseFold claim immediately?** No; it queues a closure and target, and finish performs the opening.
292. **Which lookup checks share one Y commitment?** Its point evaluation and its inner product against the transparent power table.
293. **What does the first batching scalar combine?** Relations for the same oracle: $T_i=\sum_j\lambda^jt_{ij}$ and $S_i=\sum_j\lambda^js_{ij}$.
294. **Why must claims precede that scalar?** Otherwise the prover can choose errors that cancel at the revealed value.
295. **How many padding factors appear in the implemented product relation?** One per extra coordinate, padding the entire relation.
296. **What remains after the shared product sumcheck?** Claimed oracle evaluations that still require commitment authentication.

## Lesson 040 — coding and representation

Introduced: 2026-10-23. Understanding unchecked.

297. **Are the message MLE and the coding polynomial the same polynomial?** No; message entries are Boolean values for one and bit-reversed novel-basis coefficients for the other.
298. **Does the codeword start with the original message?** Generally no; this encoding is not systematic.
299. **Why does a degree bound give code distance?** A nonzero degree-at-most $M-1$ difference has at most $M-1$ roots.
300. **Which basis defines the code domain?** The configured Gao–Mateer basis, distinct from the polynomial basis used to pack bits.
301. **Why repeat before the skipped-layer NTT?** It implements butterflies pairing live coefficients with zero padding.
302. **How are four original batch blocks ordered in encoded lanes?** $0,2,1,3$, from reversal of the two batch-index bits.

## Lesson 041 — synchronized folds

Introduced: 2026-10-23. Understanding unchecked.

303. **Why not average adjacent encoded symbols directly?** First invert their NTT butterfly to recover the two smaller-polynomial values.
304. **What is the fold/encoding identity?** Folding the encoding at $\chi$ equals encoding the message with its highest coordinate fixed to $\chi$.
305. **What does degree-one verifier recovery compute?** $a_0=s-\alpha a_1$, then the new scalar at the fresh challenge.
306. **Must a Merkle root arrive every variable?** No; fold arities can group several challenges between committed codewords.
307. **Which final values must agree?** All fully folded terminal cosets and the final MLE-check scalar.
308. **Does division-free verifier recovery remove the prover's alpha-one caveat?** No; the optimized prover infers an endpoint through invert_or_zero of $1-\alpha$.

## Lesson 042 — lifts and repeats

Introduced: 2026-10-23. Understanding unchecked.

309. **What factor does adding a zero high coordinate contribute?** $1-r_j$ at an evaluation point.
310. **What factor does repeating a table along a high coordinate contribute?** One; the polynomial ignores that coordinate.
311. **Which padding enters the Phase-B combined target?** Only log_lift coordinates, rather than every extra coordinate used in PhaseA.
312. **How is a lifted codeword queried virtually?** Shift its global leaf index right by log_lift.
313. **Which later first-fold challenges does a smaller unmasked oracle use?** A suffix of length equal to its own batch exponent.
314. **Why must outer oracle weights be sampled after the alpha messages?** To prevent false evaluation claims from adapting to their fresh combination.

## Lesson 043 — Merkle authentication

Introduced: 2026-10-23. Understanding unchecked.

315. **How deep is a tree over $2^L$ symbols with $2^a$ per leaf?** $L-a$.
316. **What hashes a standard leaf?** Ordinary SHA-256 of its serialized field values.
317. **What computes a standard parent?** One raw SHA-256 compression block from a domain-separated initial state, with native-word state conventions.
318. **Why send one internal layer for a batch of queries?** Its root is checked once, and each leaf needs a shorter path to that layer.
319. **What digest count does that format use?** $2^h+q(d-h)$, minimized at $h=\min(d,\lceil\log_2q\rceil)$.
320. **How is a full terminal vector authenticated?** Rehash all its leaves and rebuild its previously committed root.

## Lesson 044 — queries and costs

Introduced: 2026-10-23. Understanding unchecked.

321. **Which query error factor does the helper use?** $(1+\rho)/2$ for code rate $\rho$.
322. **Does the96-bit query target describe all protocol soundness losses?** No; folding, other reductions, hashes and Fiat–Shamir also need accounting.
323. **What happens to query counts as the rate gets very small?** They approach the requested number of security bits while the codeword continues growing.
324. **Why not choose a very wide grouped fold?** Opened coset values grow as $2^a$, trading against shorter paths and later codewords.
325. **Does the combined compiler use its supplied arity-strategy argument?** No; it invokes optimal_for_batch on this path.
326. **Does fri::proof_size count the entire Binius proof?** No; it counts the FRI component, including its initial roots and decommitments.

Next review: cards309–326. Research continuation remains active; user understanding is unchecked.
