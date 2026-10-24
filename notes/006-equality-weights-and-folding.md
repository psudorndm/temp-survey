# Lesson 006 — Evaluate by folding the table

2026-10-23 · MLEs, part 2 · Introduced; understanding unchecked.

## Recall

The table [0,1] extends to f̃(X) = X. More generally, endpoints a,b give the affine expression a + r(b − a) at r. This same expression is the operation used in every fold.

## Fixing one variable halves a table

Keep the previous table [1,0,1,1], indexed by i = x₀ + 2x₁, with f̃(X,Y) = 1 + X + XY. To bind the highest-index variable Y to s, pair corresponding entries across the two halves:

| Remaining X index | Y=0 entry | Y=1 entry | Folded value at Y=s |
|---|---|---|---|
| 0 | 1 | 1 | (1−s)·1 + s·1 = 1 |
| 1 | 0 | 1 | (1−s)·0 + s·1 = s |

The four-entry table becomes [1,s]. It describes f̃(X,s), not a newly chosen unrelated polynomial. Now fix X = r: the result is (1−r)·1 + r·s = 1 + r + rs in characteristic two.

In our toy F₄, choose r = s = α. The sequence is

\[
[1,0,1,1]\ \longrightarrow\ [1,\alpha]
\ \longrightarrow\ 1+\alpha+\alpha^2=0.
\]

The first pair is entries 0 and 2, not adjacent entries 0 and 1. Little-endian indexing makes the highest variable select halves. Binding the lowest variable first would instead pair adjacent entries; either order gives the same full evaluation when coordinates are matched correctly.

## Equality weights express the same calculation

For one coordinate define

\[
\operatorname{eq}(x,y)=(1-x)(1-y)+xy.
\]

For Boolean x,y, it is 1 when they agree and 0 otherwise. In characteristic two, it simplifies to 1 + x + y. For vectors, multiply the coordinate factors:

\[
\operatorname{eq}_n(r,u)=\prod_{j=0}^{n-1}(1+r_j+u_j).
\]

When u is Boolean, this is exactly the MLE basis weight: the j-th factor is 1−rⱼ if uⱼ=0 and rⱼ if uⱼ=1. Hence

\[
\widetilde f(r)=\sum_{u\in\{0,1\}^n}f(u)\operatorname{eq}_n(r,u).
\]

For two variables, the weight vector in little-endian order is

\[
[(1-r)(1-s),\ r(1-s),\ (1-r)s,\ rs].
\]

At (α,α), this becomes [α,1,1,α+1]. Its inner product with [1,0,1,1] is 0, agreeing with folding. Weights sum to 1 because each coordinate's two factors sum to 1. They are field elements, not probability weights.

Outside the Boolean cube, eq is an algebraic extension, not a general-purpose Boolean equality test. For example, eq₂((0,0),(α,α+1)) = 1 even though the vectors differ. Its selector interpretation is guaranteed on Boolean pairs.

## How the implementation uses these identities

At pinned Binius64 revision `4428e759dd27956bd5b9ae5025d67eb8ccc1accf`:

- `fold_highest_var_inplace` in `multilinear/fold.rs` broadcasts one challenge across packed lanes, splits the buffer in half, calls `extrapolate_line` on corresponding packed entries, and truncates the logical length by one variable. Parallel iteration distributes the pair updates.
- `extrapolate_line` in `line.rs` implements a + (b − a)r. Thus each pair uses one field multiplication in the generic scalar operation model. SIMD and special-value optimizations affect actual machine cost.
- `eq.rs` delegates Boolean-cube operations to `OneCube` in `hypercube.rs`. Its characteristic-two `eq_one_var` returns x + y + 1.
- Scalar weight expansion starts with [1]. For each coordinate r, it replaces each old value v by the low contribution v − vr and appends the high contribution vr. This creates the little-endian weight vector with one multiplication per old value.

For N = 2ⁿ entries, generic full folding performs N/2 + N/4 + … + 1 = N−1 pair multiplications. It is linear-time evaluation, not a logarithmic-time verifier. Computing just one eqₙ(r,u) takes O(n) operations, but materializing all N weights takes O(N) work and storage. Keep those two costs distinct.

Buffer splitting, allocation, SIMD field multiplication, and parallel scheduling rely on lower-level implementations not validated by this inspection. No Rust code was executed here.

## What this enables next

Folding is deterministic partial evaluation. By itself it proves nothing and does not authenticate the result. Sumcheck will use verifier challenges to turn a large sum claim into a smaller claim round by round; polynomial commitments later authenticate the final evaluations.

Recall: when binding Y first in our table, why are the pairs (0,2) and (1,3)? Because Y is the high index bit and selects which half holds an entry.

Next: sumcheck, with a full small transcript and an explicit final evaluation check.

Sources: [Blueprint MLE and equality definitions](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex); [fold routine](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/math/src/multilinear/fold.rs#L38); [line interpolation](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/math/src/line.rs#L20); [equality implementation](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/math/src/multilinear/hypercube.rs#L88).

Validation: checked weight expansion, Boolean selection, and sum-of-weights identities through three variables. A three-variable table's repeated folds agree with the weighted sum at all 64 points of F₄³. The worked four-entry example and the off-cube equality counterexample were also checked independently.
