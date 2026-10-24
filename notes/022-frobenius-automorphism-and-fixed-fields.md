# Lesson 022 — Frobenius: squaring as an invertible bit-linear map

Introduced: 2026-10-23. Understanding unchecked. Pinned code revision: 4428e759dd27956bd5b9ae5025d67eb8ccc1accf; Blueprint revision: 05061c6f9eca316b6844396883afb9ae37ff2fe9.

**One idea:** in a binary field, squaring preserves XOR and multiplication and permutes the field elements. Repeated squaring can therefore be reversed without losing information.

Prerequisites: [binary fields](001-binary-field-arithmetic.md), [embeddings](003-subfields-and-embeddings.md) and [IntMul bit powers](020-bit-selected-powers-and-product-trees.md). The fixed-field appendix below finishes the subfield existence/uniqueness proof left open in Lesson003. The main chat lesson focuses on squaring and its inverse.

## 1. Define the map and identify its linearity

Let $K=\mathbb{F}_{2^{m}}$ and define the Frobenius map

$$
\varphi(x)=x^2.
$$

In characteristic two,

$$
(x+y)^2=x^2+2xy+y^2=x^2+y^2.
$$

Also $(xy)^{2}=x^{2}y^{2}$ and $1^{2}=1$. For a scalar c in $\mathbb{F}_{2}$, $c^{2}=c$, so

$$
\varphi(x+y)=\varphi(x)+\varphi(y),\qquad\varphi(cx)=c\varphi(x).
$$

This is **linearity over $\mathbb{F}_{2}$**, the bit field. Relative to any chosen m-bit basis, squaring is multiplication by an m×m binary matrix: it can be expressed using XORs of input bits. That is a mathematical representation, not a claim that the current hardware backend uses a dense matrix.

It is generally not linear over the whole field K. For $c\in K$, $\varphi (cx)=c^{2}\varphi (x)$, and $c^{2}$ need not equal c. For example, in $\mathbb{F}_{4}$, $\varphi (\alpha \cdot 1)=\alpha +1$ while $\alpha \varphi (1)=\alpha$. Calling squaring a degree-two polynomial and an $\mathbb{F}_2$-linear map is consistent: these refer to different scalar fields/coordinate descriptions.

Because $\varphi$ also preserves products and 1, an invertible $\varphi$ is a field automorphism: a structure-preserving permutation ofK.

## 2. A two-bit example you can retain

In $\mathbb F_4$, $\alpha ^{2}=\alpha +1$. For bits $a_0,a_1$,

$$
(a_0+a_1\alpha)^2=(a_0+a_1)+a_1\alpha.
$$

Thus the coordinate map is

$$
(a_0,a_1)\longmapsto(a_0\oplus a_1,a_1).
$$

Its binary matrix is

$$
\begin{pmatrix}1&1\\0&1\end{pmatrix}.
$$

Applying it twice gives the identity:

| x | $x^{2}$ | $(x^{2})^{2}$ |
|---|---|---|
| 0 | 0 | 0 |
| 1 | 1 | 1 |
| $\alpha$ | $\alpha +1$ | $\alpha$ |
| $\alpha +1$ | $\alpha$ | $\alpha +1$ |

The bits0 and 1 are fixed. The other two elements are exchanged.

## 3. Why squaring is invertible

If $x^{2}=y^{2}$, then $(x+y)^2=0$. A field has no nonzero element whose square is zero, so $x+y=0$ and $x=y$. Squaring is injective; on a finite set it is consequently bijective.

There is also an explicit inverse. The nonzero multiplicative group has order $2^m-1$, giving $x^{2^m}=x$ for every nonzerox; zero satisfies the same equation. Hence

$$
\varphi^m=\operatorname{id},\qquad\varphi^{-1}=\varphi^{m-1},\qquad\varphi^{-i}=\varphi^{m-i}\quad(0\leq i\leq m).
$$

Here $\varphi ^i$ meansi iterations of the function, $so\varphi ^i(x)=x^{2^{i}}$. For arbitraryi, reduce the iteration count modulom. The inverse operation is another sequence of squarings, not exponentiation to a negative integer.

The global order $of\varphi$ is exactlym. If0<j<m and $\varphi ^j$ were the identity on allK, the nonzero polynomial $T^{2^j}-T$, of degree $2^{j}<2^m$, would vanish at all $2^m$ elements, contradicting the root bound. Individual elements may return sooner because they lie in smaller subfields.

In particular, every element has a unique square root:

$$
\sqrt{x}=x^{2^{m-1}}.
$$

For $\mathbb F_{16}=\mathbb F_2[u]/(u^4+u+1)$, the Frobenius orbit ofu is

$$
u\longmapsto u^2\longmapsto u+1\longmapsto u^2+1\longmapsto u.
$$

Thus $\operatorname{sqrt}(u)=u^{2}+1$. Multiplicative inversion is a different map: $1/u=u^3+1$. In the smaller $\mathbb F_4$ example those two maps happen to coincide on nonzero elements, which could otherwise hide this distinction.

## 4. Why exponent bits are special

For a bit $b\in\{0,1\}$, $\varphi ^i(b)=b$ for everyi. Every binary coefficient and Boolean index is fixed by Frobenius. More generally,

$$
\varphi^i(x+y)=\varphi^i(x)+\varphi^i(y),\qquad\varphi^i(xy)=\varphi^i(x)\varphi^i(y).
$$

The IntMul leaf values from Lesson020 contain powers $f[x]^{2^i}=\varphi^i(f[x])$ together with Boolean exponent bits. Inverse Frobenius can undo these large powers while preserving sums/products and leaving those bits unchanged.

The subtlety is interpolation: applying inverse Frobenius to an MLE evaluation changes its coefficients and equality weights. The actual helper transforms **both** the claimed value and every evaluation-point coordinate. The exact identity and its bit-specific leaf semantics are next lesson; simply transforming the value while keeping the point is not the general rule.

## 5. What the actual helper does

In verifier IntMul common.rs:

- inv_frobenius(x,i) computes an iterate at positionm−i of the forward squaring sequence, where `m = F::Scalar::N_BITS`. Its used indices must be within0…m; the helper does not implement arbitrary modular reduction ofi.
- inv_frobenius_sequence(x,n) creates$[x,\varphi (x),\ldots ,\varphi ^m(x)]$, reverses it and keeps then requested terms. Since $\varphi^m(x)=x$, the result is$[x,\varphi ^{-1}(x),\varphi ^{-2}(x),\ldots ]$. It asserts $n\leq m+1$.
- frobenius_twist requests $n=2^k$ terms for each point coordinate, assembling one transformed point per leaf. It applies the corresponding inverse iterate to each leaf evaluation too. For64-bit IntMul, $k=6$ and leaf indices are0…63. The verifier's field-width requirement $m\geq128$ puts these indices inside the helper's range.
- twist_limb_claim uses the same inverse-map idea for fixed-base lookup columns, whose table bases differ by Frobenius powers.

The sequence helper reuses one coordinate's forward orbit across its many inverse twists. The value helper handles each separate leaf value. These are source-level algorithm facts; exact iterator, compiler, packed-kernel and wall-clock costs were not measured. The code uses its field square operations, not the independent Gaussian-elimination model described below.

The high-level comment on frobenius_twist describes inverse twists as claims on one shared polynomialP. The preceding variable-base leaves depend on the specific exponent bit, so that comment needs to be interpreted/reconciled with the exact selector tables in the next lesson. This lesson verifies the automorphism and helper outputs, without treating that abbreviated description as a full leaf-discharge argument.

## 6. Fixed points form subfields

This section completes an earlier mathematical question; it is additional reference material.

For any positive integerd, consider the fixed set

$$
S_d=\{x\in\mathbb F_{2^m}:\varphi^d(x)=x\}.
$$

It contains0 and 1. Additivity and multiplicativity imply it is closed under addition and multiplication. For nonzero $x\in S_d$, applying $\varphi^d$ to $xx^{-1}=1$ gives $\varphi^d(x^{-1})=x^{-1}$. ThusS_d is a subfield.

Count it using a generatorg ofK*. Nonzero $x=g^t$ is fixed exactly when

$$
x^{2^d-1}=1,\qquad(2^m-1)\mid(2^d-1)t.
$$

In a cyclic group of orderM, the congruence $at\equiv0$ modM hasgcd(a,M) solutions. Therefore

$$
\begin{aligned}|S_d|&=1+\gcd(2^d-1,2^m-1)\\&=2^{\gcd(d,m)}.\end{aligned}
$$

The last identity follows from the Euclidean algorithm: reducing $2^m-1$ modulo $2^d-1$ reduces the exponentm modulod, since $2^d\equiv1$ in that modulus. Iterate until the exponents' gcd is reached.

Ifd dividesm, this constructs a subfield with exactly $2^d$ elements. It is unique: every element of any $2^d$-element subfield satisfies $x^{2^d}=x$, so that subfield is contained inS_d and has the same cardinality. Conversely, a subfield of degree d forcesd|m by the vector-space degree formula from Lesson003.

Consequently,

$$
\mathbb F_{2^d}\hookrightarrow\mathbb F_{2^m}\quad\Longleftrightarrow\quad d\mid m.
$$

and its image is the unique fixed field $of\varphi ^d$. An abstract small-field representation can still have several embeddings into that image; unique image subfield does not mean every chosen representation has only one embedding map.

Examples:

- In $\mathbb F_{16}$, $\varphi^2$ fixes $\{0,1,u^2+u,u^2+u+1\}$, raw coordinates[0,1,6,7], the embedded $\mathbb F_4$ of Lesson003.
- In the AES field $\mathbb F_{2^8}$, $\varphi ^{6}$ fixes only $2^{\gcd(6,8)}=4$ elements. It does not produce a64-element subfield. This is another explanation of why BitAnd's64-point additive domain is a vector subspace, not $\mathbb F_{64}$.
- Since8 divides128, the AES field has an embedding into GHASH128, consistent with the conversion table checked in Lesson003.

## 7. Source anchors and independent verification

- [Blueprint source](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex), §2.1 Binary Fields: characteristic-two squaring, cyclic multiplicative group, subfields and Frobenius. §4.5 variable-base exponentiation motivates inverse twists. The fixed-field counting proof above is derived here from those standard algebraic properties.
- [IntMul common.rs](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/intmul/common.rs): inv_frobenius, inv_frobenius_sequence, frobenius_twist and twist_limb_claim. Read actual helper bodies, not just their comments.
- [IntMul verify.rs](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/verifier/src/protocols/intmul/verify.rs): minimum field width and call from product-tree leaves into frobenius_twist. Full subsequent selector sumcheck remains pending.
- [GHASH field declaration](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/field/src/fields/ghash.rs): the polynomial field model used for the independent128-bit checks. No machine square implementation was validated here.

Exhaustively checked additivity, multiplicativity, permutation/inverse cycles and fixed-field counts over $\mathbb{F}_{4},\mathbb{F}_{16}$ and AES. Verified the displayed matrices/orbits/subfield examples. Independently modeled GHASH squaring as a128×128 bit matrix, inverted it by Gaussian elimination and checked rank128. For five field values and 64 inverse twists each, the source-shaped repeated-squaring/reverse method agrees with iteration of that independently computed inverse matrix. The declared full-order generator has a128-element Frobenius orbit in the model.

These checks execute scalar polynomial/bit-matrix mathematics, not Binius Rust, its packed square backend, transcript or full leaf-discharge proof. Coverage does not imply user mastery.

**Remember:** squaring permutes field elements, preserves XOR, and fixes bits. Its inverse is repeated squaring around the same finite cycle.

Next: applying inverse Frobenius to a multilinear evaluation—why the value and the evaluation point must move together—and the exact selector table behind each IntMul leaf.
