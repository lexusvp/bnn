# nanoLM.cpp — independent research and engineering evidence review

**Status:** static source review; implementation not authorized.  
**Repository:** `vitrixLab/bnn`  
**Reviewed baseline:** `192d10e520f2e086bc218d296b2f505c65ca725a`; nanoLM blob `5f0c599d1ea3b00f7d725115fe3f65dbd3f44753`.  
**Review date:** 2026-11-04.

## Evidence classifications

- **SOURCE-VERIFIED:** directly observed in source at the stated revision; not a runtime reproduction.
- **LITERATURE-VERIFIED:** publication metadata/official proceedings/identifier checked; not proof of nanoLM applicability.
- **EMPIRICAL FINDING:** a measured result limited to the cited experiment.
- **HEURISTIC:** an approximation or practical method, not an exact identity or theorem.
- **UNKNOWN / NOT RUN:** not established or not executed.

## Corrected claim register

| ID | Prior claim | Corrected claim | Verifying source / status |
|---|---|---|---|
| NLM-001 | “The attention cache makes backward incorrect.” | Source shows `AttentionCache::att` and `att_pre` are each `T×T`; each head overwrites the same matrices, and backward reads the same cache for every head. This is a **source-verified defect candidate**: probabilities from the last processed head are reused for all heads. Runtime gradient failure remains **NOT REPRODUCED**. | `src/nanolm.cpp` at blob `5f0c599d1ea3b00f7d725115fe3f65dbd3f44753`, `AttentionCache`, `attention_fwd`, `attention_bwd`. Vaswani et al., arXiv:1706.03762, [record](https://arxiv.org/abs/1706.03762), supports head-specific attention architecture, not this code defect. |
| NLM-002 | “The final sampled next-token target is omitted.” | Training samples (T+1) tokens but passes only the first (T) as context to the forward/loss call. The extra sampled token is not available to that call as a target; the loss loop covers only within-context next-token positions. Static source finding; test **NOT RUN**. | Same nanoLM source blob, `train` and `forward` loss loop. |
| NLM-003 | “Checkpoint serialization is portable/versioned.” | The format writes native `Config` bytes and raw float arrays; source comments state there is no magic/version and native packing/byte order is used. Portability across ABI/endianness is **not established**. | Same source blob, `save_weights` / `load_weights`. Compatibility tests **NOT RUN**. |
| NLM-004 | “Configuration validation is complete.” | Some positive-dimension and divisibility checks exist, but complete field validation, upper bounds, overflow-safe tensor sizing and all pre-allocation checks are not established. This is a static validation gap/risk, not a reproduced exploit. | Same source blob, `load_weights`. Boundary tests **NOT RUN**. |
| NLM-005 | “Training errors propagate to the process exit code.” | `train` returns `void`; missing/short corpus paths return from `train`, after which `main` can return 0. Save paths do not consistently propagate write failures. Static operational defect candidate; CLI tests **NOT RUN**. | Same source blob, `train`, `save_weights`, `main`. |
| NLM-006 | “eval_every performs evaluation.” | It prints the current training loss; the inspected path does not implement held-out validation, BPC or an evaluation manifest. Call it a training-loss print interval, not validation. | Same source blob, `train`; see [Issue #27](https://github.com/vitrixLab/bnn/issues/27). |
| NLM-007 | “Padding and unknown-character semantics are distinct.” | Generation initializes context with token ID 0; tokenizer unknown-character handling maps to ID 0. Padding/unknown/ordinary-token semantics are therefore conflated in the current implementation. Impact requires tests. | Same source blob, `Tokenizer::encode`, `generate`. Tests **NOT RUN**. |
| NLM-008 | “nanoLM is always 14,208 parameters.” | Parameter count is configuration-dependent; `count_params()` hardcodes a four-layer multiplier. 14,208 applies only to the stated configuration with vocabulary size 65; it is not universal. | Same source blob, `Config`, `ModelWeights::count_params`. |
| CI-001 | “Sanitizer CI validates nanoLM runtime behavior.” | The workflow builds an ASan/UBSan binary but does not execute it. Cppcheck is static analysis. Build success does not establish functional correctness. | `.github/workflows/nanolm-ci.yml`, blob `f0eede9cec76c166fab02339edd3aa024b9fbcc6`; workflow source inspected, no run result asserted. |
| CI-002 | “The no-argument CTest smoke test passes.” | CMake registers `nanolm_no_args` with `WILL_FAIL FALSE`, while `main` returns 1 without arguments. CTest therefore expects success from a failing CLI path. The workflow does not run CTest. This is a misconfigured-test candidate, not an executed failure. | `src/CMakeLists.txt`, blob `eacd8f50f947a09fda426bd44d015bcb378639be`; same nanoLM source blob. |
| BIN-001 | “BIN1/INT4 integration will improve nanoLM inference.” | No nanoLM-specific integration, parity, quality or throughput result supports this. Keep FP32 as reference; require layer-level and end-to-end comparisons before any claim. | Current repository integration boundary; no quantized nanoLM experiment in this review. |
| BIN-002 | “BNN demonstrates a 7× nanoLM speedup.” | The paper reports 7× for a particular binary matrix-multiplication GPU kernel against an unoptimized GPU kernel. It is empirical and setup-specific, not a nanoLM or CPU speedup. | Hubara et al., NeurIPS 2016, [official proceedings](https://proceedings.neurips.cc/paper/2016/hash/d8330f857a17c53d217014ee776bfd50-Abstract.html). |
| BIN-003 | “XNOR-Net found 58× fewer high-precision operations for Transformers.” | The source reports 58× faster convolutional operations and 32× memory savings in binary convolutional image-classification workloads. Not a Transformer/nanoLM result. | Rastegari et al., arXiv:1603.05279, [record](https://arxiv.org/abs/1603.05279). Empirical and workload-specific. |
| BIN-004 | “STE is the exact derivative of sign.” | Bengio et al. describe STE as a heuristic gradient estimator for stochastic/non-smooth units. It is not the exact derivative of a discrete sign function. | Bengio, Léonard & Courville, arXiv:1308.3432, [record](https://arxiv.org/abs/1308.3432). Heuristic, not a nanoLM convergence theorem. |
| EVAL-001 | “15.6K tokens/s is the nanoLM baseline.” | This is historical reported sandbox throughput, not a newly reproduced or project-hardware baseline. Do not cite as current performance without exact-SHA repeat measurements and raw samples. | Earlier project audit / Issue #27 comment; **NOT REPRODUCED** in this review. |

## Verified literature and evidence class

- Vaswani et al. (2017), “Attention Is All You Need,” arXiv:1706.03762 — architectural paper; per-head operations are distinct. Not proof of a nanoLM defect.
- Courbariaux, Bengio & David (2015), “BinaryConnect,” arXiv:1511.00363 — preprint; binary weights during propagation with higher-precision stored weights for updates; task-specific empirical results.
- Hubara et al. (2016), “Binarized Neural Networks,” NeurIPS 2016 official proceedings — peer-reviewed conference paper; 7× GPU-kernel result is a specific empirical comparison.
- Rastegari et al. (2016), “XNOR-Net,” arXiv:1603.05279 — preprint record; convolution/image-classification results, not Transformer inference.
- Bengio, Léonard & Courville (2013), “Estimating or Propagating Gradients Through Stochastic Neurons for Conditional Computation,” arXiv:1308.3432 — preprint; STE is a heuristic estimator.
- Magnusson et al. (2024), “Paloma,” NeurIPS 2024 Datasets and Benchmarks Track, DOI [10.52202/079017-2052](https://doi.org/10.52202/079017-2052) — benchmark paper motivating domain-sensitive language-model evaluation; does not validate nanoLM.
- Balloccu et al. (2024), “Leak, Cheat, Repeat,” EACL 2024, DOI [10.18653/v1/2024.eacl-long.5](https://doi.org/10.18653/v1/2024.eacl-long.5) — empirical analysis of contamination/evaluation in its studied closed-source-LLM paper sample; do not generalize its prevalence to nanoLM.
- Kingma & Ba (2015), “Adam,” ICLR 2015, arXiv:1412.6980 — publication record verified; does not certify nanoLM's optimizer implementation.

## Unverified claims — mark or remove; do not cite

- BIN1 improves nanoLM quality, throughput, memory, or rule generalization.
- Quantized inference is integrated into nanoLM or achieves parity.
- Historical 15.6K tokens/s is a reproducible project baseline.
- NLM-001 through NLM-008 are runtime-reproduced failures; current evidence is static source inspection.
- The supplied *Algebraic Mind* note's chapter/page-level account: the book text was not consulted, so claims remain **UNKNOWN against the text**.
- The supplied System 3-A scoping review is preliminary and does not establish empirical nanoLM claims.
- Hinton et al. (2015) distillation, BitNet / BitNet b1.58, and Fodor & Pylyshyn references remain **[UNVERIFIED — DO NOT CITE]** until exact acceptable records are verified.

## Minimal verification plan — not implementation authorization

1. Test NLM-001 with per-head cache fixtures and finite-difference gradients for Q/K/V/output projection.
2. Test NLM-002 with an exact target-count fixture including the final sampled target.
3. Add focused tests for checkpoint bounds/truncation, CLI error propagation, tokenizer unknown/padding behavior, and parameter counting under non-default layer counts.
4. Correct the CTest contract and explicitly run CTest.
5. Execute sanitizer-instrumented functional tests, not only a sanitizer build.
6. Establish held-out BPC and run-manifest requirements before quantization experiments.
7. Keep INT8/INT4/BIN1 experiments separate and preserve FP32 reference.

**Disposition: HOLD for research validation.** No implementation, training, benchmark, merge or release is authorized by this review.
