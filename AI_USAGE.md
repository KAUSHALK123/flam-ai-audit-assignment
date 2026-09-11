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

| Metric | Original Starter Kit | Our Verified Experiment | Description |
| :--- | :--- | :--- | :--- |
| **Corpus Data** | 10 lines (Toy Sample) | 1,012 lines (FLORES-200) | We validated the integrity of exactly 1,012 parallel sentences to run statistically significant tests. |
| **Averaging Method** | Macro-averaging (5.889x) | Micro-averaging | We proved macro-averaging heavily skews results and switched to micro-averaging for accuracy. |
| **Tokenizer Overhead** | GPT-2 baseline | MuRIL / XLM-R models | We verified that Indic-aware tokenizers reduce Dravidian/Hindi token overhead by 90-93%. |
| **Text Normalization** | Ignored | NFC Normalization | We verified NFC composition altered only 93 lines, causing a negligible 0.110% token shift. |
| **KV-Cache Memory** | No theoretical derivation | 112 KiB / token | We mathematically derived the KV cache footprint to prove a hard limit of 25 concurrent sequences. |
| **Preemptions** | Ignored in capacity planning | Exact integer prediction | We perfectly predicted 7 preemptions at Batch 32 and 23 at Batch 48 based on memory math. |
| **Throughput (Goodput)** | 1607.4 tok/s (includes prefill) | 200.9 tok/s | We proved the original report hallucinated generation speed by failing to subtract 82k prefill tokens. |
| **Reviewer Arithmetic** | Not considered | 720 pairs (33% coverage) | We verified the math showing a single human reviewer can only cover 2 out of 6 languages in 30 hours. |

## 3. What the Agent Got Wrong or Had to Correct
1. **Gated Repository Failure**: The initial plan considered evaluating `ai4bharat/indic-bert`. Execution failed immediately with an HTTP 401 Gated Repo Error requiring Hugging Face token authentication. The agent corrected course by selecting the fully open and accessible `google/muril-base-cased` and `xlm-roberta-base` models, logging the gated access as a genuine dead end.
2. **TinyURL Automation Trap**: The agent initially attempted to stream the dataset archive directly from `https://tinyurl.com/flores200dataset`. TinyURL's service returned an HTML preview landing page instead of the binary archive. The agent resolved this by tracing the underlying canonical URL hosted directly on Meta AI's CDN (`https://dl.fbaipublicfiles.com/nllb/flores200_dataset.tar.gz`).
3. **Windows Terminal Encoding Crash (`cp1252`)**: When printing Hindi/Tamil test outputs to `stdout` during `run_audit_flaws.py`, Python raised a `UnicodeEncodeError` because the default Windows shell encoding was `cp1252`. The agent diagnosed the issue and added `sys.stdout.reconfigure(encoding="utf-8")` to all script entry points.
4. **Draft Metric Discrepancy**: In an early iteration of `corrected_analysis.md`, estimated draft numbers for Tamil were briefly drafted before the multi-tokenizer execution completed. The agent audited the generated markdown against the verified CSV output and updated the narrative text to reflect the exact measured figures (e.g. 415.19 tok/sentence on GPT-2 vs 28.86 on MuRIL).
5. **Part B GiB vs Decimal GB Reconciliation**: An early draft calculation used pure binary GiB ($1024^3$) for all quantities without noting that the hardware spec specifies decimal 24 GB and 4.2B parameters. The agent reconciled the arithmetic by providing the exact byte-level breakdown ($114,688\text{ bytes/tok}$, $12.08\text{ GB}$ pool), proving that the decimal calculation yields 25.71 sequences, which matches the empirical preemption boundary in `bench_log.csv` with exact integer fidelity.

## 4. Final Implementation Deltas (Starter Kit vs. Final Submission)

| Domain | Original Starter Kit | Final Submission | Detailed Impact |
| :--- | :--- | :--- | :--- |
| **Code Structure** | Naive `.split(" ")` and `len()` counts | Regex graphemes (`\X`) and byte metrics | Prevented double-spaces and Unicode combining marks (matras) from breaking the token-to-word math. |
| **Fertility Metrics** | Flawed macro-averaging (5.889x) | Rigorous micro-averaging | Corrected the statistical skew to reveal the true 90%+ token reduction for Dravidian languages. |
| **Model Selection** | Western-centric GPT-2 baseline | Indic-aware models (XLM-R, MuRIL) | Shifted from an unfair baseline to native vocabularies for fair architectural evaluation. |
| **KV-Cache Math** | Ignored memory footprints | Derived exactly $112 \text{ KiB}$ per token | Proved that the 12.08 GB usable pool hits a hard physical ceiling at exactly 25.7 concurrent sequences. |
| **Throughput Formula**| Misread `reported_tok_s` as speed | Derived exact Goodput formula | Corrected formula to `(gen_len * batch) / wall_s`, proving real generation speed is only 200.9 tok/s. |
| **Systems Approach** | Guesswork for batch 32 collapse | First-principles hardware limits | Replaced anecdotal observation with mathematical proof predicting the exact integer preemption counts. |
| **Business Strategy** | Vague deployment goals | Zero-budget prefix-caching | Rejected expensive Fine-Tuning (SFT) in favor of zero-cost prompt engineering to respect reviewer limits. |
