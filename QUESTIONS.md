# Open questions

## Learning preferences not yet established

- Prior familiarity with finite fields, polynomial commitments, sumcheck, and Rust.
- Whether the user prefers original-paper chronology or current-code-first. Current default: foundations plus Binius64 implementation, with historical comparisons.

## Technical investigation queue

- Trace GHASH multiplication kernels and representation conversions; the polynomial-basis toy example and source assertions are covered in Lesson 001, but machine instruction paths remain untraced.
- Basic quadratic towers are in Lesson004; current packing, binary transpose and tensor-algebra identities are derived in Lessons035–037. Historical tower PCS comparison and the complete original/ring-switching paper proofs remain open.
- Reconcile the current Blueprint and code per reduction, including any recently changed interfaces.
- Trace soundness errors and concrete parameters; avoid treating field size as the full security argument.
- Formalize transcript challenge generation and binding, beyond the receive-before-sample call order inspected in Lesson 002.
- Trace sumcheck integration's challenge-coordinate ordering at each caller; generic verification pushes challenges in sampling order.
- Identify exactly what the transparent transcript reveals and what each ZK layer masks.
- Trace statement binding: circuit, public inputs, configuration, transcript domain separation, and any indexed relations.
- Verify circuit-size contribution to verification time from actual code.
- Run a minimal prove/verify example when the user reaches the implementation track.
- Trace circuit lowering, full constraint-system validation, and shift canonicalization; Lesson 009 covers the native data structures and direct relation checks only.
- Trace lower-level FieldBuffer splitting and machine-specific packed multiplication; Lesson 006 inspected fold orchestration and scalar equality expansion, not all backend execution.
- Trace downstream authentication of rerand output values. Lesson 018 establishes operation ordering at the constraint-reduction call boundary and the shift entry's four-coordinate operand batching, but the full shift/wiring/PCS check and concrete transcript binding remain unverified. A local scalar closing identity does not independently certify every value.
- Lesson 025 derives the actual four-16-bit-limb fixed-base path, eight precomputed power tables, 12 gathered columns, phase4 caller/closing interface, claim twists and compact public-table MLE. The Blueprint's 64-bit-selector presentation remains a separate path. Lessons026–028 now derive weighted buckets, rational/GKR checks and actual index-to-bit closure. Concrete deferred oracle openings and full soundness accounting remain; product consistency alone does not authenticate witness bits.
- Lesson 024 covers variable-base selector batching, execute/fold/finish, equality tracking and output recombination. Full switchover transpose/lookup internals, lower-level split-half/quadratic kernels, packed runtime and batch/unequal-depth prodcheck remain unreviewed.
- Trace the concrete challenge sampler and completeness/error accounting for the optimized prime interpolation at $\alpha=1$. Its missing-constant formula divides by $1-\alpha$ via invert_or_zero, and the helper tests exclude one. Lesson 024 records an exact local information-loss example; no full-protocol failure or attack is established.

## Resolved in initial survey

- Original repository versus successor: original is archived; Binius64 is the successor.
- Core soundness versus privacy: plain proof entry points are non-ZK; separate ZK wrapper exists.
- Which specification to consult: the docs identify the Blueprint whitepaper as canonical and warn that web chapters lag.
- Toy field arithmetic: F₄ example, irreducibility, inverses, and the GHASH polynomial relation recorded in Lesson 001. Mathematical examples verified; user recall remains unchecked.
- Large-challenge motivation: bounded-degree root argument and its limitations recorded in Lesson 002. Full protocol error accounting remains open.
- Field embedding example and actual Rijndael-to-GHASH table inspected in Lesson 003; all 256 table entries algebraically verified independently. Call-site behavior and runtime performance remain untraced.
- Quadratic tower construction and all 256 products independently verified in Lesson 004. Current field abstraction interfaces inspected; concrete SIMD implementations and CPU kernels remain untraced.
- MLE existence, uniqueness, and four-entry interpolation derived in Lesson 005. All four-entry F₄-valued tables independently checked against monomial evaluation and folding; equality weights continued in Lesson 006.
- Equality weights and folding mechanics derived in Lesson 006; scalar expansion and packed fold orchestration inspected. Lower-level SIMD/memory details and Rust runtime validation remain open.
- Basic sumcheck transcript, soundness propagation, and omitted-coefficient recovery covered in Lesson 007. Full transcript security and terminal-claim integration remain open.
- Generic zerocheck and factored MLE-check derived in Lesson 008, including constant recovery and nonzero honest terminal values. Actual BitAnd specialization and composition openings remain to be traced.
- Native word constraint language covered in Lesson 009, including segmented indices and two-shift/XOR operand evaluation. Frontend compilation, full validation, and shift normalization remain untraced.
- Baseline BitAnd polynomial construction and table/orchestration interfaces covered in Lesson 010; oblong interpolation and fused bit-axis folding continued in Lesson 011. Univariate message, rerandomization and full shift authentication remain open.
- Oblong interpolation, Lagrange weight sweeps, and fused bit-axis folding covered in Lesson 011. Derive the subspace-polynomial recurrence fully later; byte lookup construction, univariate messages, rerandomization, and shift authentication remain open.
- Deterministic coordinates, skip interpolation, and the small-field message kernel are covered through Lesson014. Additive NTT mathematical basis and engine details are next.
- Reconcile the specification's126/|K| skip discrepancy constant with the inspected generic degree127 message space. A conservative127/|K| root bound follows locally; full batched soundness remains unproved here.
- Additive NTT normalized basis and reference butterfly were derived in Lesson 015. Lesson 016 completes a numerical transcript and checks GenericOnTheFly full-layer indexing, a shortened image-domain example, and compatible contexts for the BitAnd LDE. Partial-layer skips, other context types and optimized kernels remain open; image domains are not interchangeable with nested prefix spans.
- Lesson 017 derives different-point batching and traces local rerand prover/verifier, ordinary sumcheck driver, equality adaptor and high-variable padding. The spec's two combined operand tables are a proposed optimization; the pinned code maintains individual operand columns. Global IntMul/BinMul ordering and output authentication remain to be traced. Zero reduction is next.
- Lesson 018 introduces direct Zero evaluation, fresh output-point reuse and the local total-degree bound; reads actual shared point extension, prefix-to-full matrix-padding scales,14-slot claim assembly and4-coordinate downstream batching. The reduction call order is IntMul/BinMul→BitAnd→Zero→shift. Full multiplication internals and shift authentication remain open. Native Zero is a constraint family; zerocheck is a general protocol technique.
- Lesson 019 derives full-order exponent encoding, exact128-bit range/parity equivalence and the wider-field case. Independently verified the declared GHASH generator order and selected actual parity/array-order bodies. The exponentiation tables are interpolated after Boolean-row exponentiation; off-cube exponentiation of an MLE value is not the same operation. Next derive variable-base bit-selector leaves and their product-tree proof.
- Lesson 020 derives variable-base bit-selected powers and reads compute_b_leaves scalar/packed branches plus ProdcheckProver::new/single-prove and the verifier recursion. Confirms row-low/index-high layout, high-half pairing, all-one padding, retained layers and6ℓ+15 single-tree MLE-check rounds. Full layer proof is next; GKR consistency alone does not authenticate leaf semantics.
- Lesson 021 derives parent weighted sum→factored degree-two MLE-check→two children→fresh selector line, with full numerical transcript, exact next-point order and local reduction error bounds. Product equality alone does not authenticate child values; Frobenius/leaf discharge and witness checks remain required.

- Lesson 022 derives Frobenius bit-linearity, invertibility, cycle order and inverse-helper outputs. Completes Lesson003 subfield converse/unique-image proof via fixed-field cardinality 2^gcd(d,m). Full MLE twisting/selector reduction and user understanding remain unchecked.

- Lesson 023 proves simultaneous value/point Frobenius transformation and bit-specific selector semantics; actual phase3 closure agrees. Shared-P helper wording is imprecise for these leaves, not an established implementation defect. Next trace γ/outer batching, selector round/fold kernel and output recombination.

- Reconcile logup* module comments using $(n+m)/|K|$ with definitions of n,m as variable counts. Lesson027 derives local residue bounds in terms of distinct address support M and records the pole event; full combined soundness, out-of-range indices, root-denominator handling and multi-table accounting remain open. The primary paper PDF was unavailable, so its lemma was not reviewed.
- Full BinMul primitive interface and endpoint are covered in Lesson029; WordAxisFolder internals, packed kernels, concrete transcript and downstream witness authentication remain open.

- Lessons030–031 cover mathematical shift routing, transparent evaluation, five-factor sumcheck and wiring/public/private handoff. Lessons032–034 cover representative sparse phase/key bodies; actual ring-switch/oracle openings and lower helper/runtime behavior remain.

- Lesson034 traces two final shift-prover quotient extractions despite the verifier's division-free endpoint interface. Zero selector/scale may cause honest rejection later; source comments acknowledge incompleteness. The extraction-only ideal-coin exceptional-set bound is49/|K|; combine it with the earlier alpha=1 interpolation and other protocol exceptional events in the later completeness/security lesson. No code fix or Rust reproduction was performed.
- Sparse key/outer/segment stages are covered through034. Specialized ShiftChallengePoint partial_eval helper recurrences, SIMD mask edge cases, serialization validation and runtime behavior remain explicit engineering follow-ups.

- Lessons035–038 resolve the current GHASH packing/transpose/tensor identities and public packed closure. The actual row-batching sample dimension is seven (logarithm of128), even for a one-element packed table; preserve the distinction from Blueprint §4.8's summary dimension. Local ideal-coin bounds derived: row batch at most $7/|K|$, public path at most $(7+2m_{\mathrm{pub}})/|K|$. Global challenge/commitment/security composition remains open.
- The ring-switching paper's PDF returned403. Its full §2/Construction3.1 proof remains unread; the notebook's derivation is independently checked against code and finite-field examples. Historical comparison must not silently present this as a reviewed paper proof.
- Next: derive and authenticate the private trace and logup*-Y linear-oracle relations through actual channel batching, BaseFold, codeword tests and Merkle openings. Understand unequal sizes and terminal checks before marking indexed lookup or witness authentication complete.
- Plain ring-switch auxiliary values are witness-dependent. Explain how the actual ZK wrapper represents/masks these and constrains its verifier replay; row batching by itself is not zero knowledge.

- Lessons039–041 cover deferred relation queues, same-oracle batching, implemented single-factor product padding and ordinary coding/folding/terminal control flow. Differentiate lambda (relations), theta (oracle sumcheck), outer tensor coordinates and optional mask gamma. Full unequal-size/mixed first-fold/Merkle/parameter authentication remains next; do not yet mark lookup or private witness closure complete.
- The Blueprint's two-zero-padded-operands presentation produces a squared high-coordinate factor off cube. Current PaddedSumcheckDecorator pads the whole product once, matching the verifier endpoint's one factor. This convention is derived and independently checked in039; preserve it in later protocol maps.
- The non-ZK IntMul path adds a pushforward commitment to the private trace commitment. Current compiler supports per-oracle ZK flags; old single-oracle/always-ZK comments are not universal contracts. Trace actual oracle specs/counts in the end-to-end runtime example.
- Lesson041 extends the alpha=1 honest-prover interpolation issue: MultilinearEvalEvaluator samples R(1), uses RoundEvals::interpolate_eq and loses R(0) at1. Message[1,0] gives a direct scalar-path reproduction. Verifier recovery is sound and division-free. Confirm sampler behavior and a pinned Rust reproduction later; do not report an exploited soundness vulnerability or assume the singular coordinate is excluded.

- Lessons042–044 resolve unequal-size first-fold routing and Merkle mechanisms: high zero-lift coordinates contribute eq-zero factors while repeat coordinates do not; PhaseA/PhaseB products intentionally differ. Early/outer/later suffix windows and virtual indices were checked with mixed GHASH fixtures. Underlying adaptive proximity theorem and total soundness accounting remain open.
- Query helper targets96bits with (1+rate)/2 and has a count linear in requested bits at fixed rate, unlike Blueprint5.2 prose. Exact FRI component size excludes BaseFold/inner reduction messages. Distinguish code-derived sizes and butterfly counts from measured proof files and runtime.
- Parameter search uses an estimate and early-stop unimodality assumption; root counts are added by exact accounting.168single-oracle comparisons found no full-cost gap, but global optimality and all parameter bounds remain unproved. Full security accounting must include algebraic folding, hash/ROM losses and earlier reduction errors.
- Native-word parent-compression conventions and fixed LE field serialization were read. Independent raw compression matches the pinned little-endian vector. Rust architecture parity and cross-endian interoperability remain engineering follow-ups.
- Concrete indexed lookup and private witness opening mechanism coverage is complete through044. Next: determine exactly which circuit/configuration/public values and proof tapes feed the transcript, which are decommitment advice, and what final validation rejects. Zero knowledge, full runtime and user understanding remain unchecked.
