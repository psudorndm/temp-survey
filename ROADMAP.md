# Learning roadmap

Status means coverage, not mastery. Orientation and Lessons 001–044 have been introduced; understanding has not yet been checked.

| Stage | Main question | Evidence / implementation target | Status |
|---|---|---|---|
| 0. Orientation | What problem is Binius solving, and which generation? | README, Blueprint introduction, constraint definitions | Introduced |
| 1. Binary fields | Why XOR? How does multiplication work? | Blueprint §§2.1–2.2; field arithmetic | Lessons 001–002 and022: arithmetic, large challenges and Frobenius introduced |
| 2. Towers and packing | How can bits stay compact while challenges live in a large field? | Original paper; ring-switching paper | Lessons 003–004,022,035–037: embeddings, fixed-field proof, tower arithmetic, actual GHASH packing, non-field-linear projection and tensor algebra covered; historical PCS comparison remains |
| 3. Multilinears | How does a table become a polynomial? | Blueprint §2; math crate | Lessons 005–006: interpolation, equality weights, folding introduced |
| 4. Sumcheck | How does a huge sum become one evaluation claim? | Blueprint §2.2; ip / ip-prover | Lessons 007–008 and 017: sumcheck, zerocheck, factored MLE-check, powers batching and unequal-dimension padding introduced; advanced variants pending |
| 5. Constraint language | How do words, shifts, and XOR operands describe computation? | Blueprint §3; core / frontend | Lesson 009 native language introduced; compiler lowering still pending |
| 6. BitAnd and Zero | How are Boolean relations checked algebraically? | Blueprint §§4.3–4.4; bitand protocols | Lessons 010–018: BitAnd through NTT/LDE and batched re-randomization; direct Zero claim and point extension introduced; end-to-end authentication pending |
| 7. IntMul and BinMul | How do integer and field multiplication differ? | Blueprint §§4.5–4.6; intmul / binmul | Lessons019–029 cover arithmetic/lookup reductions;039–044 explain concrete Y/trace relation authentication through BaseFold and Merkle. Runtime, kernel and full security/completeness analysis remain in later stages |
| 8. Wiring and shifts | How are operand claims tied to the actual witness? | Blueprint §§4.2, 4.7; shift reduction | Lessons030–038 cover wiring/shift/public closure;039–044 cover private packed authentication. Specialized helpers, runtime checks and exceptional completeness accounting remain in later stages |
| 9. Ring-switching | How are claims transferred to packed witness data? | Blueprint §4.8; ring_switch | Lessons035–038 derive packing obstacle, transpose, fresh seven-coordinate row batch, tensor indicator, subset-table fold and public/private interfaces; private oracle proof follows in Stage10 |
| 10. Commitments | How do BaseFold, coding, and Merkle commitments fit together? | Blueprint §5; iop / iop-prover | Lessons039–044 cover queues/batching, padding, coding, ordinary and mixed first folds, lift/repeat, virtual queries, Merkle paths/shared layers, terminal checks, query targets and FRI size/arity accounting. Full adaptive proximity theorem, transcript compilation and Rust/runtime validation remain explicit later work |
| 11. Public statement and transcript | What exactly is bound to the proof? | Blueprint §4.9; transcript / verification | Pending |
| 12. Zero knowledge | What leaks in the core, and how does composition hide it? | Blueprint §§6–7; Spartan and zk_config | Entry points checked only |
| 13. End-to-end code | Trace a tiny circuit from builder to verifier | frontend → witness → prove → verify | Pending |
| 14. Performance and security | Where do time, memory, and soundness error go? | Actual kernels, parameters, tests, benchmarks | Pending |

For each stage: one concise lesson, one worked example, source anchors, one misconception to avoid, and recall cards. Use more than one lesson when a stage contains several ideas.

Historical side track: original tower-field PCS → FRI-Binius / ring-switching → current Binius64. Explain which ideas persist and which interfaces/protocol layers changed.
