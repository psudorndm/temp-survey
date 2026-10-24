# Lesson 014 — Computing the skip message efficiently

2026-10-23 · Honest prover kernel · Introduced; understanding unchecked.

## The target calculation

For each of the 64 output-domain points z in D' minus D, compute the weighted row sum H(z) from Lesson 013. A direct implementation could interpolate A, B, C separately for every row and do all weighting in GHASH. The inspected implementation reorganizes the same calculation using linear maps, AES arithmetic, and precomputed tables.

This lesson explains the algebra and source flow. Additive-NTT polynomial bases, butterfly derivations, hardware instruction selection, and runtime performance measurements remain separate work.

## Step 1: extrapolate a word through byte lookups

For a word w, define LDE(w) as its 64 interpolated evaluations on D' minus D, when the original 64 bits are evaluations on D. This is a low-degree extension, not the multiplication of a 64-bit field element. Its output is a vector of 64 AES-field values.

Interpolation is linear over F₂. Splitting w into eight bytes gives

\[
\operatorname{LDE}(w)=\bigoplus_{b=0}^{7}T_b[\operatorname{byte}_b(w)].
\]

Each T_b contains 256 possible 64-byte output vectors. A full table layout would use 8·256·64 = 128 KiB. Per operand word, lookup evaluation then takes eight vector lookups and XOR combination, avoiding a fresh interpolation transform in the inner loop.

The source method is named `NTTLookup::ntt`, but it computes this LDE. To build its tables, `LowDegreeExtension::transform` inverse-transforms the 64 input evaluations, zero-pads the recovered polynomial coefficients, then forward-transforms on the 128-point superdomain and selects the upper 64 outputs. The coefficients are in the transform's chosen polynomial basis; do not silently identify them with monomial coefficients before studying that basis.

## The current table is compressed to 32 KiB

`NTTLookup` actually stores two tables, one for even byte positions and one for odd byte positions: 2·256·64 = 32 KiB of field-data payload. Six other byte positions are reconstructed by permuting output lanes.

Why a permutation is possible: shifting the input index by a subspace element translates the output-domain point by the same element. The input domain is an additive subspace and the output is its coset, so translation permutes both. Therefore corresponding LDE matrix columns differ by fixed output permutations.

In the inspected `ntt` loop, byte position b selects table b mod 2 and permutes the four 128-bit storage lanes by `lane[i] = lane[i XOR floor(b/2)]`. Each lane contains 16 AES evaluations. The parity tables handle the within-lane offset; the remaining byte positions need only this lane permutation.

`NTTLookup::new` computes 16 unit-bit LDEs (two byte positions times eight bits), then builds each 256-entry byte table by subset sums. It is called during message preparation; no global permanently cached table was established by this source trace. The 32-KiB number describes stored field payload, not measured total process memory or a promised speedup.

## Step 2: do the eight-row inner calculation in AES

For each row x, the kernel obtains three packed vectors:

\[
A_z=\operatorname{LDE}(A[x]),\quad
B_z=\operatorname{LDE}(B[x]),\quad
C_z=\operatorname{LDE}(A[x]\mathbin{\&}B[x]).
\]

The C input word is derived in registers. Its LDE is computed separately; it is not obtained by multiplying the other two vectors.

Group rows by the first three row bits u. With the AES equality weights w_u from Lesson 012, compute a 64-entry AES vector

\[
S_v[z]=\sum_{u\in\{0,1\}^3}w_u
\bigl(A_z(u,v)B_z(u,v)+C_z(u,v)\bigr).
\]

All factors in this inner expression lie in AES. `PackedRijndael64x8b` processes the 64 output-domain evaluations as independent AES lanes. The code accumulates wide multiplication outputs across the eight weighted residuals and calls `reduce` once for the group; concrete wide arithmetic and CPU instructions are not yet traced.

Here the fixed weights enable small-field aggregation, while their basis property separately establishes the no-cancellation argument on actual bit-domain constraints. Off-domain residuals need not be bits; the kernel uses ordinary field arithmetic there, not a new assertion of bit independence.

## Step 3: include the large-field weights by another lookup

The code divides remaining row coordinates into four fresh large-field coordinates and any extra coordinates. The constant `N_FIXED_LARGE_CHALLENGES=4` fixes the *number* used per window; these four values are still sampled, unlike the three deterministic AES coordinates.

Four coordinates yield 16 equality weights λ_v. For each, `B8ToExtMulMap` precomputes

\[
M_v[t]=\lambda_v\,\iota(t),\qquad t\in\mathbb F_{2^8}.
\]

It constructs the eight embedded AES-basis products with λ_v and expands their subset sums into 256 entries. Calls are then direct byte-indexed lookups combining embedding with multiplication by a reused large-field constant. For GHASH elements, 16 such tables have 16·256·16 = 64 KiB of field-data payload. This is separate from the 32-KiB LDE table.

The source groups rows into 128-word windows: 8 friendly-weighted rows times 16 large-weight subgroups. Each window accumulates Σ_v M_v[S_v[z]] for every output point z, then multiplies the 64 accumulated values by the equality weight for the remaining, higher window-index coordinates. Parallel workers' windows are added to obtain the final message.

Thus the calculation does use GHASH: table construction, large-field accumulation, and the outer window weighting remain there. The optimization reduces repeated large-field operations and conversion work; it does not move the entire proof into an 8-bit field.

## Short columns: pad within the logical axis, repeat beyond it

Rows beyond the actual column length but inside its next-power-of-two axis are zero. For a logical axis shorter than an internal 128-word window, `padded_chunks` repeats that padded axis to fill the window. This is important when artificial coordinates inherit nonzero fixed small-field weights: summing duplicated values against both endpoint weights gives (1−ρ)+ρ=1. Merely zero-filling those extra coordinates would incorrectly scale by 1−ρ.

Missing large-field window coordinates are handled with zero-padded equality-weight arrays, so absent subgroups carry zero weight. Full windows are borrowed directly; only a partial window needs a padded copy. This is an implementation-shape adjustment, not additional witness constraints.

## Source evidence and boundaries

At pinned revision `4428e759dd27956bd5b9ae5025d67eb8ccc1accf`, `sumcheck_round_messages.rs::univariate_round_message_extension_domain` implements the sequence above. Its inner loop calls `ntt_lookup.ntt` three times, derives c_i=a_i&b_i, accumulates the AES group, then calls the outer-weight maps. A later loop applies the window equality weight and reduces across parallel chunks.

One comment in ntt_lookup.rs says `ntt` is used only in tests and the production kernel inlines table access. The actual inspected production body calls `ntt` directly. Follow that body rather than using the stale comment as a call-graph description.

The Blueprint's straightforward eight-position lookup costs 128 KiB. The current two-parity-table code costs 32 KiB of payload; this is a source-version/implementation refinement. No measured throughput, cache-miss count, constant-time property, or complete backend validation is claimed.

Recall: which part remains in the large field? The outer weighting/aggregation and its precomputed maps. Which part uses AES? Per-word LDE values and the eight-row weighted residual groups.

Next: additive NTT mathematics. Explain why characteristic-two domains use additive subspaces rather than a classical power-of-two roots-of-unity transform, then derive subspace polynomials and the recursive interpolation basis.

Sources: [Blueprint §4.3.3](https://github.com/binius-zk/binius.xyz/blob/05061c6f9eca316b6844396883afb9ae37ff2fe9/whitepaper/main.tex); [lookup construction and compression](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/bitand/ntt_lookup.rs); [actual message kernel and weight maps](https://github.com/binius-zk/binius64/blob/4428e759dd27956bd5b9ae5025d67eb8ccc1accf/crates/prover/src/protocols/bitand/sumcheck_round_messages.rs).

Validation: built an independent textbook Lagrange matrix over an explicit AES subspace and checked the two-table permutation scheme on all 64 unit words and representative full words. Modeled AES grouping, actual retained GHASH embedding, higher-coordinate weighting, and short-axis repetition against a direct weighted sum at lengths 1,3,8,15,128,129,384,401. These are algebra/layout checks; Rust, the additive NTT engine, SIMD backends, and end-to-end proofs were not executed.
