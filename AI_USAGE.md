# AI Usage Log — Part A Implementation

## 1. What the Agent Generated
- **Corpus Fetching & Extraction**: Automating the resolution of canonical FLORES-200 download endpoints and extracting the 5-language `devtest` splits (`eng.txt`, `hin.txt`, `kan.txt`, `tam.txt`, `tel.txt`).
- **Scripts**:
  - `partA/scripts/run_audit_flaws.py`: Automated framework to isolate and compute empirical deltas for all 6 known baseline claims.
  - `partA/scripts/fertility_v2.py`: Corrected, production-ready benchmarking pipeline supporting arbitrary tokenizers, Unicode grapheme clusters, UTF-8 bytes, whitespace words, and parallel sentence denominators.
- **Reports & Artifacts**:
  - `partA/corpus/README.md`: Complete metadata, licensing, domain provenance, and critical evaluation of corpus limitations.
  - `partA/results/audit_evidence.md` and `.csv`: Tabulated isolation methods, exact commands, before/after values, and verdicts.
  - `partA/results/corrected_analysis.md` and `.csv`: Full evaluation matrix across 3 tokenizers and 5 languages, along with the technical justification for intent-based routing metrics.
  - `partA/memo.md`: 1-page executive recommendation memo for leadership.
  - `NOTEBOOK.md`: Chronological experimental log following `HYPOTHESIS → EXPERIMENT → RESULT → REVISION`.

## 2. What Was Verified Independently
- **Baseline Reproduction**: Independently verified that `starter_kit/fertility.py` on the sample corpus yields unrounded fertility division of `5.889x` (1.27 vs 7.45 tok/word).
- **Corpus Sentence Integrity**: Independently checked line counts across all 5 corpus files (`1,012` lines each, exact 1:1 parallel alignment).
- **Audit Deltas**: Verified every before/after metric by running standalone Python scripts and inspecting token outputs.
- **NFC Normalization Impact**: Verified by scanning all 1,012 lines of FLORES-200 Hindi that NFC normalization altered only 93 lines by composing combining characters (such as Nuktas), resulting in a +221 token shift (+0.110%) out of 200,467 total tokens.
- **Multi-Tokenizer Parity**: Recomputed tokenizer statistics across `gpt2`, `xlm-roberta-base`, and `google/muril-base-cased` to confirm that Indic-aware tokenizers reduce Dravidian tokens by 90%–93%.
- **Part B KV-Cache Arithmetic**: Derived formulas from first principles and verified against `bench_log.csv` row values:
  - Exact match of predicted 93.3% utilization vs logged 0.93 `kv_cache_util` at batch 24.
  - Exact prediction of integer preemption counts: $32 - 25 = 7$ (logged `preempted_seqs = 7`) and $48 - 25 = 23$ (logged `preempted_seqs = 23`).
  - Independent two-method calculation of batch 24 goodput ($200.9\text{ tok/s}$).
- **Part C Reviewer Arithmetic**: Verified reviewer hours ($10\text{h/wk} \times 3\text{wk} = 30\text{h}$), reviewing throughput ($24\text{ pairs/hr} \to 720\text{ pairs}$), and exact language coverage ($2/6 = 33.3\%$).

## 3. What the Agent Got Wrong or Had to Correct
1. **Gated Repository Failure**: The initial plan considered evaluating `ai4bharat/indic-bert`. Execution failed immediately with an HTTP 401 Gated Repo Error requiring Hugging Face token authentication. The agent corrected course by selecting the fully open and accessible `google/muril-base-cased` and `xlm-roberta-base` models, logging the gated access as a genuine dead end.
2. **TinyURL Automation Trap**: The agent initially attempted to stream the dataset archive directly from `https://tinyurl.com/flores200dataset`. TinyURL's service returned an HTML preview landing page instead of the binary archive. The agent resolved this by tracing the underlying canonical URL hosted directly on Meta AI's CDN (`https://dl.fbaipublicfiles.com/nllb/flores200_dataset.tar.gz`).
3. **Windows Terminal Encoding Crash (`cp1252`)**: When printing Hindi/Tamil test outputs to `stdout` during `run_audit_flaws.py`, Python raised a `UnicodeEncodeError` because the default Windows shell encoding was `cp1252`. The agent diagnosed the issue and added `sys.stdout.reconfigure(encoding="utf-8")` to all script entry points.
4. **Draft Metric Discrepancy**: In an early iteration of `corrected_analysis.md`, estimated draft numbers for Tamil were briefly drafted before the multi-tokenizer execution completed. The agent audited the generated markdown against the verified CSV output and updated the narrative text to reflect the exact measured figures (e.g. 415.19 tok/sentence on GPT-2 vs 28.86 on MuRIL).
5. **Part B GiB vs Decimal GB Reconciliation**: An early draft calculation used pure binary GiB ($1024^3$) for all quantities without noting that the hardware spec specifies decimal 24 GB and 4.2B parameters. The agent reconciled the arithmetic by providing the exact byte-level breakdown ($114,688\text{ bytes/tok}$, $12.08\text{ GB}$ pool), proving that the decimal calculation yields 25.71 sequences, which matches the empirical preemption boundary in `bench_log.csv` with exact integer fidelity.
