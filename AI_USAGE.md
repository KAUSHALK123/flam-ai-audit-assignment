# AI Usage Log — Part A Implementation

## Overview of Tools & High-Level Strategy
- **ChatGPT**: I explained my complete conceptual approach to GPT and had it formulate two highly structured prompts: the first for rigorous experimentation (uncovering flaws, hardware constraints, and root-cause reasoning), and the second for implementing the final fixes.
- **Claude**: I used Claude to generate the initial implementation plan and structure the overall architectural approach before executing the code.

## 1. Initial Prompting & What the Agent Generated
Instead of writing code immediately, the initial stages were heavily focused on planning and constraint mapping:
- **Implementation Planning**: I prompted the agent to generate a strict implementation plan before any code was written. This ensured we adhered to the assignment constraints (like not modifying the starter kit).
- **Finding Flaws & Constraints**: I asked the agent to deeply audit the baseline `fertility.py` script and the `bench_log.csv` data to identify mathematical and structural flaws, hardware constraints, and logic errors.
- **Code & Artifact Generation**: Only after the planning and flaw-finding phases were complete did the agent generate the final Python scripts (`fertility_v2.py`, `run_audit_flaws.py`) and the corresponding Markdown reports/CSV tables mapping out the exact empirical differences.

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
