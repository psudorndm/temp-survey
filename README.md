# Binius learning notebook

Start here. This project studies Binius from intuition through mathematics to implementation, in small lessons.

- [Project context](PROJECT_CONTEXT.md): your preferences and the current stopping point.
- [Learning roadmap](ROADMAP.md): sequence and coverage.
- [Completion tracker](COMPLETION.md): remaining course requirements and autonomous continuation.
- [Source register](SOURCES.md): official references, versions, and caveats.
- [Code map](CODE_MAP.md): where ideas meet Rust.
- [Glossary](GLOSSARY.md): short definitions.
- [Recall cards](REVIEW.md): questions with compact answers.
- [Open questions](QUESTIONS.md): unresolved points.
- [Lesson 000 — orientation](notes/000-orientation.md): first mental model.
- [Lesson 001 — binary-field arithmetic](notes/001-binary-field-arithmetic.md): a complete four-element field and its connection to GHASH.
- [Lesson 002 — large-field challenges](notes/002-why-large-field-challenges.md): why random checks need more than bits.
- [Lesson 003 — subfields and embeddings](notes/003-subfields-and-embeddings.md): preserving arithmetic across different field representations.
- [Lesson 004 — binary towers and packing](notes/004-binary-towers-and-packing.md): larger field elements as pairs; distinction from SIMD lanes.
- [Lesson 005 — multilinear extensions](notes/005-multilinear-extensions.md): a table's unique polynomial and evaluations beyond bit inputs.
- [Lesson 006 — equality weights and folding](notes/006-equality-weights-and-folding.md): halve a table by binding a variable; connect to weighted evaluation.
- [Lesson 007 — sumcheck transcript](notes/007-sumcheck-a-complete-transcript.md): a full two-round example, soundness reasoning, and coefficient compression.
- [Lesson 008 — zerocheck and MLE-check](notes/008-zerocheck-and-mle-check.md): prevent cancellation from hiding constraint errors; distinguish Boolean vanishing from formal identity.
- [Lesson 009 — Binius64 constraint language](notes/009-binius64-constraint-language.md): word operands, shifted references, and the four native relations.
- [Lesson 010 — AND constraints to polynomial claims](notes/010-and-constraints-to-polynomial-claims.md): bit/row tables, residual composition, and remaining operand authentication.
- [Lesson 011 — oblong-multilinearization](notes/011-oblong-multilinearization.md): a 64-point univariate bit axis, degree bounds, and actual Lagrange folding.
- [Lesson 012 — deterministic zerocheck](notes/012-deterministic-zerocheck.md): encode eight bit errors injectively using fixed AES equality weights.
- [Lesson 013 — univariate-skip message](notes/013-univariate-skip-message.md): prescribed zeros, message authentication, and honest versus encoded degree bounds.
- [Lesson 014 — small-field message kernel](notes/014-small-field-message-kernel.md): byte LDE lookups, AES grouping, and large-field window aggregation.
- [Lesson 015 — additive NTT foundations](notes/015-additive-ntt-subspace-polynomials.md): subspace polynomials, normalization, novel basis, and the butterfly identity.
- [Lesson 016 — four-point NTT and LDE](notes/016-four-point-ntt-and-lde.md): complete forward/inverse buffers, pair indexing, transformed domains, and the actual 64→128 extension.
- [Lesson 017 — batching and re-randomization](notes/017-batching-and-rerandomization.md): equality-weighted point switching, a full toy transcript, degree-three batching, padding, and implementation differences.
- [Lesson 018 — Zero reduction](notes/018-zero-reduction-and-unified-point.md): direct zero evaluation, challenge reuse, extended row points, full-matrix padding, and downstream operand batching.
- [Lesson 019 — IntMul exponent encoding](notes/019-intmul-exponent-encoding-and-wraparound.md): multiplicative-group identity, exact range proof, wraparound parity, generator validation, and the code's limb/lookup route.
- [Lesson 020 — bit-selected powers](notes/020-bit-selected-powers-and-product-trees.md): exponent bits select powers or1; exact product-tree layout, padding, logical sizes and the boundary before GKR.
- [Lesson 021 — one GKR layer](notes/021-one-gkr-layer-and-line-reduction.md): weighted parent claim, degree-two MLE-check, two child evaluations, fresh line combination, and a complete transcript.

- [Lesson 022 — Frobenius](notes/022-frobenius-automorphism-and-fixed-fields.md): invertible bit-linear squaring, helper implementation and the completed subfield existence/uniqueness proof.

- [Lesson 023 — twisting MLE claims](notes/023-frobenius-twists-of-mle-claims.md): move the value and point together, derive bit-specific selector tables, and resolve the helper comment.
- [Lesson 024 — selector batching](notes/024-selector-batching-and-common-point.md): six coordinates combine 64 claims; degree-three sumcheck moves their columns to one row point; packed selector kernel and output recombination.
- [Lesson 025 — fixed-base limbs](notes/025-fixed-base-limbs-and-transparent-power-table.md): four 16-bit limbs, three fixed-base trees, eight precomputed tables, Frobenius reuse and the verifier's compact table evaluation.

- [Lesson 026 — weighted pushforward](notes/026-weighted-pushforward-for-indexed-lookups.md): collect interpolation weights by address, derive the transpose identity and trace12-looker power batching.
- [Lesson 027 — rational lookup proof](notes/027-logup-rational-identity-and-fraction-gkr.md): derive residue uniqueness, pole/root bounds, paired GKR and two openings against one pushforward commitment.
- [Lesson 028 — IntMul index binding](notes/028-intmul-index-to-bit-binding-and-final-unification.md): basis-linear limb reconstruction, four-coordinate claim folding and the final index/parity/exponent sumcheck.
- [Lesson 029 — BinMul](notes/029-binmul-packing-and-factored-zerocheck.md): pack six word columns into three GHASH values, use factored degree-two zerocheck and reconstruct the endpoint.

- [Lesson 030 — shift indicators](notes/030-shift-indicators-and-bit-carry-evaluation.md): bit routing, two-shift composition, transparent carry recurrence and transpose weight movement.
- [Lesson 031 — five-factor shift proof](notes/031-five-factor-shift-sumcheck-and-witness-closure.md): degree-two factor graph, exact round layout, sparse wiring and public/private witness obligations.

- [Lesson 032 — sparse shift keys](notes/032-sparse-shift-keys-and-witness-weighted-rows.md): group word/sequence references, stable builder layout and masked bit-row accumulation.
- [Lesson 033 — implicit shift weights](notes/033-implicit-shift-weights-and-sparse-sumcheck-rounds.md): avoid the full24-variable table, derive sparse round samples and preserve additive duplicate rows.
- [Lesson 034 — segmented witness folding](notes/034-segmented-shift-product-and-exceptional-extraction.md): sparse first-round correction, in-place selector folding and precise zero-factor extraction limits.

- [Lesson 035 — packed bits and linearity](notes/035-packed-bits-and-the-linearity-obstacle.md): lossless word-pair packing and why bit extraction does not commute with challenge-field scalars.
- [Lesson 036 — transpose and row batching](notes/036-ring-switch-transpose-and-fresh-row-batching.md): derive the packed linear relation, seven-coordinate challenge and conditional local soundness bound.
- [Lesson 037 — tensor indicator](notes/037-tensor-algebra-and-the-ring-switch-indicator.md): two field copies, a compact off-cube evaluator and a concrete counterexample to naive bit projection.
- [Lesson 038 — ring-switch kernels and closure](notes/038-ring-switch-tables-and-public-private-closure.md): subset tables, fused coefficient generation, direct public closure and the private oracle boundary.

- [Lesson 039 — deferred oracle relations](notes/039-linear-oracle-queues-and-relation-batching.md): pending obligations, same-oracle powers batching and the single padding factor in shared sumcheck.
- [Lesson 040 — Reed–Solomon encoding](notes/040-reed-solomon-encoding-and-bit-reversal.md): message MLE versus coding polynomial, Gao–Mateer domain, distance and bit-reversed interleaving.
- [Lesson 041 — BaseFold synchronization](notes/041-basefold-butterflies-and-the-terminal-check.md): inverse-butterfly fold, shared MLE-check challenges and mandatory terminal consistency.

- [Lesson 042 — unequal-size oracle placement](notes/042-unequal-oracles-lift-repeat-and-first-fold-routing.md): zero-lift versus repeat, first-fold challenge windows and virtual query translation.
- [Lesson 043 — Merkle authentication](notes/043-merkle-leaves-domain-separation-and-shared-layers.md): coset leaves, the actual SHA-256 parent function, shared layers and terminal-vector checking.
- [Lesson 044 — queries and proof bytes](notes/044-fri-query-counts-arities-and-proof-size-accounting.md): query counts, rate/arity trade-offs, parameter selection and scoped FRI size accounting.

## Session log

2026-10-23 — Created the notebook and persistent workspace instructions. Located official docs, successor and archived repositories, current whitepaper, historical papers, and author resource lists. Inspected constraint definitions and core proof orchestration; confirmed separate transparent and ZK entry points. Saved selected source snapshots locally. This is an initial orientation, not an exhaustive implementation or security review.

2026-10-23 — Lesson 001 introduces polynomial-basis arithmetic in F₄, compares field multiplication with integer multiplication and AND, and derives why the chosen polynomial is irreducible. Inspected pinned GHASH source examples and verified all 16 toy-field products independently. Recall questions added; understanding not yet checked.

2026-10-23 — Lesson 002 derives the univariate root bound, works P(T) = T² + T over F₄, and records the degree and message-ordering conditions. Inspected the pinned sumcheck verifier's receive-before-sample order and its final evaluation-claim boundary. Verified the four toy evaluations. No claim of full protocol soundness analysis or user mastery.

2026-10-23 — Lesson 003 works an F₄-to-F₁₆ embedding and inspects the actual Rijndael-to-GHASH conversion table. Verified the toy mapping and independently reconstructed all 256 conversion entries from the image of the AES generator. No Rust tests run; understanding unchecked.

2026-10-23 — Lesson 004 constructs the original tower's F₁₆ over F₄, derives pair multiplication, and distinguishes embeddings, coordinate packing, and SIMD. Inspected current ExtensionField and PackedField interfaces. Verified all 256 toy products against an independent field representation and all nonzero inverses. User understanding unchecked.

2026-10-23 — Lesson 005 derives two-entry and four-entry MLE interpolation, uniqueness, indexing, and the distinction between evaluation entries and monomial coefficients. Inspected actual evaluation routines; independently checked every four-entry F₄ table against interpolation, coefficient evaluation, and folding. User understanding unchecked.

2026-10-23 — Lesson 006 derives folding and equality weights with the previous table. Traced packed fold orchestration, line interpolation, Boolean equality simplification, and scalar weight expansion. Independently checked three-variable folding and weight identities. User understanding unchecked.

2026-10-23 — Lesson 007 works a complete sumcheck transcript and derives the round-by-round soundness bound. Inspected omitted-coefficient recovery. Verified all toy challenge pairs, a false-claim transcript exposing the terminal-check requirement, and compressed polynomials through degree bound three. User understanding unchecked.

2026-10-23 — Lesson 008 derives a random equality-weighted zerocheck and the factored MLE-check endpoint equation. Inspected actual omitted-constant recovery and edge cases τ=0,1. Independently checked the root bound across all nonzero four-entry F₄ tables and a valid constraint with nonzero terminal evaluation. User understanding unchecked.

2026-10-23 — Lesson 009 traces shifted/XOR operands and the four native constraint forms through current source. Worked one BitAnd relation without extra intermediate witness words, distinguished IntMul from BinMul, and checked the arithmetic and single-bit mutations. User understanding unchecked; no Rust execution.

2026-10-23 — Lesson 010 derives the baseline BitAnd table/composition check and separates it from optimized Binius64. Inspected column assembly and BitAnd orchestration; recorded why deriving C still requires later authentication against the actual operand. Checked toy residuals and off-cube composition values. User understanding unchecked; no Rust execution.

2026-10-23 — Lesson 011 derives mixed univariate/multilinear interpolation and degrees 63/126. Inspected Lagrange weights and the fused bit-axis AND fold, including domain-point challenges and explicit C=fold(A AND B). Independently checked the toy interpolant and representative 64-point AES-domain calculations. User understanding unchecked; no Rust execution.

2026-10-23 — Lesson 012 derives why equality weights at (β,β²,β⁴) form an AES basis, and how this fixes three initial row-weighting coordinates. Confirmed code constants and both parties' coordinate assembly; checked all 256 bit-error patterns, shorter prefixes, and GHASH embedding. User understanding unchecked; no Rust execution.

2026-10-23 — Lesson 013 traces the actual zero-prefixed skip-message interpolation and its random link to row sumcheck. Recorded the honest degree-126 versus generic encoded degree-127 distinction and derived the missing parity condition for the tighter bound. Checked toy messages and a 64/128-point AES example independently. User understanding unchecked; no Rust execution.

2026-10-23 — Lesson 014 traces the actual small-field message kernel: compressed 32-KiB LDE tables, eight-row AES groups, precomputed embedding/weight maps, and 128-word windows. Checked the translation/permutation algebra and grouping/padding reference at eight column shapes. User understanding unchecked; no Rust execution or benchmarks.

2026-10-23 — Lesson015 derives subspace-polynomial recurrence/additivity, normalization, novel-basis degrees and the butterfly. Inspected actual domain constants, basis contract and reference implementation. Independently matched transforms to polynomial evaluation through dimension four. User understanding unchecked; no Rust execution.

2026-10-23 — Lesson 016 works all four-point forward/inverse stages, traces exact pair/block twiddles, and explains compatible novel bases in inverse64/pad/forward128. Independently checked all F₄ coefficient vectors, an F₁₆ shortened-domain example and AES LDE against Lagrange interpolation. User understanding unchecked; no Rust execution.

2026-10-23 — Lesson 017 derives batching and point switching, traces verifier/prover rerand and the batch driver, and records high-variable padding and degree-three messages. Independently checked 65,536 toy cases and explicit cancellation/terminal-authentication limits. Source differs from the proposed two-table optimization; imprecise adaptor comments were checked against mul_by_eq. User understanding unchecked; no Rust execution.

2026-10-23 — Lesson 018 derives the direct Zero claim and its local degree bound, traces both parties' point extension and padding scales, and identifies four fresh operand-axis batching coordinates for fourteen claims. Verified toy residuals and direct padded-MLE identities independently. Witness/channel/PCS authentication remains incomplete; no Rust execution or user understanding check.

2026-10-23 — Lesson 019 derives IntMul's exponent identity and proves its sole128-bit endpoint alias is eliminated by low-bit parity. Inspected the real generator, parity batch and wire/operand orders; identified the code's16-bit fixed-base limb lookups as distinct from the Blueprint's bit-selector presentation. Independently verified small-word equivalence and actual GHASH generator order. No Rust execution or user understanding check.

2026-10-23 — Lesson 020 derives variable-base selector leaves and traces scalar/packed construction, row-low/leaf-high indexing, high-half tree pairing and multiplicative padding. Independently checked240 toy base/exponent pairs and all nodes at four64-leaf row shapes. Saved why six tree levels entail6ℓ+15 MLE-check rounds in this reduction. No Rust execution or user understanding check.

2026-10-23 — Lesson 021 derives a full GKR layer and local error bound, traces split-half product/MLE-check/line code and works a complete F₄ transcript. Independently checked16,384 one-variable and1,024 two-variable honest layers plus a finite false-parent strategy search under exact final authentication. No Rust execution or user understanding check.

2026-10-23 — Lesson 022 derives Frobenius and its inverse, checks helper outputs against an independent GHASH inverse bit matrix, and completes the fixed-field/subfield proof left open in Lesson003. Small-field properties verified exhaustively. No Rust execution or user understanding check.

2026-10-23 — Lesson 023 proves the MLE/Frobenius identity, works a complete F₄ claim and derives each IntMul bit-specific selector. Verified against actual phase3 use of separate exponent bits and shared base, resolving the abbreviated shared-P helper comment. Independent small-field and64-leaf GHASH identities checked; no Rust execution or user understanding check.

Next: Lesson 024, selector-sumcheck batching: six-coordinate γ weights, outer scalar batching and common-point base/exponent outputs.

2026-10-23 — Copied the existing notebook and source/reference snapshots into the new Binius workspace. Persisted the user's TeX preference and converted Lessons 015–023 from pseudo-equations to inline/display TeX, preserving originals. Lesson 024 derives six-coordinate selector batching, the outer scalar batch, common-row outputs and fresh bit recombination; traces the selector round/fold kernel and storage transitions. Independent mathematical checks passed; an exceptional interpolation coordinate is documented for further investigation. No Rust execution or user understanding check.

2026-10-23 — Lesson 025 derives the implemented fixed-base limb path and distinguishes it from the Blueprint's bit-selector route. Inspected table/gather construction, phase4 caller and closing interface, limb twists and the compact transparent-table evaluator. Independent scalar identities/layout checks passed, including two full65,536-row GHASH MLE evaluations. Logup* internals and index-to-bit authentication remain next; no Rust execution or user understanding check.

2026-10-23 — User authorized continuing until all lessons are complete. Enabled the existing-chat heartbeat continue-binius-lessons every30minutes and created COMPLETION.md. Saved Lessons026–029 with lookup pushforward/rational/GKR/index binding and BinMul; added cards213–236. Independent scalar checks passed and TeX renders. Recorded precise source-comment/pole limitations without claiming a protocol break. No Rust or user-mastery check.

2026-10-23 — Continued through Lessons030–031. Derived transparent shift indicators, carry recurrence, transpose weight movement and the five-factor witness relation; traced native wiring discharge and public/private handoffs. Independent miniature and actual-index-width arithmetic checks passed. The next work is sparse prover phases, then ring-switching. Autonomous continuation remains active; no Rust or user-mastery check.

2026-10-23 — Heartbeat continued with Lessons032–034: sparse key collection, implicit shift-weight tables, exact sparse messages and public/private selector folding. Added cards249–266, checked128key fixtures,96sparse rounds,320segmented points and3,840nonsingular extractions. Corrected verifier/prover division scope and recorded the current prover's exceptional quotient cases. TeX syntax passed. Next is ring-switching; no Rust or mastery check.

2026-10-23 — Continued through Lessons035–038: packed-bit scalar-linearity obstacle, transpose/row-batch proof, tensor-algebra transparent indicator, factored/subset-table prover path and complete public packed endpoint. Added cards267–290. Independent algebra/layout/transcript models passed;1,274TeX expressions render syntactically. Recorded actual seven-coordinate batching versus Blueprint summary, unread paper proof and private oracle authentication boundary. Next is concrete linear-oracle batching/BaseFold; continuation remains active, with no Rust or user-mastery check.

2026-10-23 — Goal continuation made concrete progress with Lessons039–041: deferred relation contracts and two levels of batching, exact off-cube product padding, bit-reversed/interleaved RS encoding, synchronized inverse-butterfly/MLE folds and query/terminal flow. Added cards291–308 and independent validation. TeX passes1,418expressions. Extended the alpha-one honest-prover interpolation caveat to the actual degree-one opening evaluator. Next: unequal-size lift/repeat routing, detailed Merkle authentication and parameters. No Rust or user-mastery check; course remains in progress.

2026-10-23 — Goal continuation completed Lessons042–044: unequal-size lift/repeat routing and virtual queries, exact Merkle hash/path/shared-layer behavior, and query/rate/arity/FRI-size accounting. Added cards309–326. Checked136F16/24GHASH mixed batches,160Merkle batches with720mutations,18pinned size fixtures and168cost comparisons. TeX passes1,545expressions. Core lookup/wiring/commitment mechanism coverage is now complete; runtime, full security and later course stages remain. Next: transcript and statement binding. No Rust, benchmark, whole-repository review or user-mastery claim.
