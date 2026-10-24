# Curriculum completion and autonomous continuation

User authorization: continue working until all lessons are complete, received 2026-10-23. This authorizes researching and writing the remaining curriculum without another “continue” prompt.

The ongoing chat heartbeat is **Continue Binius lessons**, id continue-binius-lessons, scheduled every 30 minutes. It resumes this project, stays quiet on routine status and reports substantive lesson batches, blockers or completion. Disable it after the curriculum below is complete. Do not create duplicates. It is an app automation; no separate operating-system scheduler or cloud backup was created.

## Meaning of complete

Completion means a coherent, source-grounded course covers every stage below, with prerequisites, derivations, worked examples, representative implementation walkthroughs, limitations and recall cards. It does not mean the user has mastered every lesson, that every repository line was reviewed, or that a new security audit was performed. Do not use “all lessons complete” while a core stage below is still missing.

Each new lesson should have TeX equations, primary-source links with pinned code revisions, and meaningful mathematical or implementation validation appropriate to its claims. Keep delivered chat explanations short; save depth in the notebook. Follow explicit unresolved technical issues to a reasoned resolution or clearly recorded remaining limitation.

## Remaining course coverage

- [x] Finish indexed lookup: weighted pushforward, logarithmic-derivative identity, fractional-addition GKR, commitments, transparent table opening, concrete index embedding and final IntMul bit/parity unification (026–028 and039–044; runtime and complete security accounting remain below).
- [x] Derive BinMul and trace its six word-operand output claims (Lesson029; downstream authentication remains separately tracked).
- [x] Complete shift/wiring reduction, normalization/padding, public-value and witness claim authentication (030–044; specialized helper/runtime review and completeness exceptions remain in engineering/security coverage).
- [x] Derive ring-switching and the needed higher-field/tower/packing identities; trace actual parameter and coordinate conventions (Lessons035–038; packed private oracle authentication remains in the commitment stage).
- [x] Explain linear-oracle relations, coding, Merkle authentication, BaseFold folding, batching, unequal sizes and terminal tests; trace prover/verifier interfaces (039–044; full adaptive proximity proof and runtime validation are retained limitations).
- [ ] Explain transcript absorption/challenge sampling, domain separation, statement/configuration/public-input binding and relevant validation.
- [ ] Derive characteristic-two zero-knowledge masking and trace the actual ZK wrapper/Spartan composition, including what the plain proof reveals.
- [ ] Trace frontend compilation, witness generation, native constraint checks and an end-to-end tiny circuit; execute a pinned minimal example when the local environment permits, clearly record any unavoidable external limitation.
- [ ] Cover concrete field/SIMD kernels, representation conversions, allocation/memory/layout, parallelism and honest operation/proof-size/verifier-time cost accounting; distinguish measurements from source comments.
- [ ] Explain concrete soundness accounting, challenge/commitment assumptions, exceptional events and security limits. Resolve or retain explicit caveats for the skip-degree and interpolation singularity questions.
- [ ] Cover original Binius and the historical FRI-Binius/ring-switching lineage, clearly distinguishing it from current Binius64 and its ZK composition.
- [ ] Finish with a complete protocol map, a searchable notation/concept index, cross-linked review exercises and an honest coverage/limitations summary.

## Current checkpoint

Lessons000–044 are saved. Indexed lookup, multiplication, wiring/shift, ring-switching and concrete BaseFold/Merkle authentication mechanisms are covered. Coverage includes mixed-size/mask routing, terminal tests, query targets and calculated FRI costs. These coverage checkmarks do not imply Rust execution, user mastery, complete adaptive soundness proofs or a security audit. Independent models and TeX syntax checks passed; explicit limitations remain in QUESTIONS.md and the notes.

Next is045: transcript messages/decommitments, challenge sampling, statement/circuit/configuration binding, domain separation, grinding and final proof validation. The unchecked ZK, end-to-end runtime/frontend, engineering/security, historical and final-reference requirements remain active. Do not stop the heartbeat or mark the course complete yet.
