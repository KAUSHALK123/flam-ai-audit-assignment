# Part A Completion Summary

All tasks for Part A of Flam "The Audit" have been completed and verified. All artifacts reside cleanly within `flam_assignment/`, leaving `starter_kit/` and `playground/` untouched as baseline reference.

---

## 1. Inventory of Created Files

| File Path | Description & Contents |
|---|---|
| `flam_assignment/NOTEBOOK.md` | Chronological research notebook tracking hypotheses, experiments, results, and revisions. Records genuine dead ends (`ai4bharat/indic-bert` gated 401, TinyURL preview redirect trap, Windows `cp1252` stdout encoding). |
| `flam_assignment/AI_USAGE.md` | AI usage log declaring what the model generated, what was verified independently via CLI execution, and what corrections were made. |
| `flam_assignment/partA/corpus/` | Clean parallel eval corpus extracted from FLORES-200 `devtest` (5 languages × 1,012 sentences): `eng.txt`, `hin.txt`, `kan.txt`, `tam.txt`, `tel.txt`. |
| `flam_assignment/partA/corpus/README.md` | Comprehensive corpus documentation: exact source (Meta AI NLLB), CC-BY-SA 4.0 license, sentence counts, Wikipedia/news domain, and a frank critical boundary analysis of what the benchmark cannot tell us (informal register, Hinglish/code-mixing, prompt template overhead). |
| `flam_assignment/partA/scripts/run_audit_flaws.py` | Automated audit test suite isolating each of the 6 baseline claims with minimal reproducible inputs, before/after numbers, and exact commands. |
| `flam_assignment/partA/results/audit_evidence.md` | Formatted evidence table with columns: `CLAIM \| ISOLATION METHOD \| COMMAND \| BEFORE \| AFTER \| DELTA \| VERDICT`, plus detailed technical notes for each flaw and the NFC normalization hygiene check. |
| `flam_assignment/partA/results/audit_evidence.csv` | Raw tabular data of all audit isolation tests. |
| `flam_assignment/partA/scripts/fertility_v2.py` | Production-grade corrected benchmarking pipeline supporting multiple tokenizers (`gpt2`, `hf:xlm-roberta-base`, `hf:google/muril-base-cased`), multiple denominators (sentences, words, graphemes, bytes), and micro/macro averages. |
| `flam_assignment/partA/results/corrected_analysis.md` | Empirical cross-language evaluation matrix across 3 tokenizers and 5 languages. Conclusively answers the decisive architectural question: **tokens per parallel sentence (intent)** is the only invariant metric that reflects serving cost. |
| `flam_assignment/partA/results/corrected_analysis.csv` | Complete measurement matrix containing token counts, tokens/unit, and relative ratios for all 15 (tokenizer × language) combinations. |
| `flam_assignment/partA/memo.md` | Concise ($\le 1$ A4 page) engineering memo for executive leadership detailing corrected headline numbers, script-aware routing strategy, the core production caveat, and the key monitoring metric. |

---

## 2. Key Empirical Findings (Headline Summary)
- **The v0 "5.89× Hindi cost penalty" is an artifact of tokenizer vocabulary bias, not script morphology**:
  - Under `gpt2` (50k vocab, English-biased BPE), Hindi requires **198.3 tok/sentence** (7.42× vs English), while Dravidian languages explode: Kannada **363.0 tok/sentence** (13.58×) and Tamil **415.2 tok/sentence** (15.54×).
  - Under `google/muril-base-cased` (197k vocab, Indic-specialized), Hindi drops to **31.6 tok/sentence** (**1.16× of English**) and Tamil drops to **28.9 tok/sentence** (**1.06× of English**, virtually 1:1 parity).
  - Switching to an Indic-aware tokenizer slashes Dravidian token sequences by **90.2% to 93.0%**, reducing KV-cache and prefill costs by **10×–14×**.
- **Denominator choice distorts reality**:
  - In agglutinative languages like Tamil, `tok/word` reaches **25.05 tok/word** (20.28× of English), exaggerating the cost penalty by nearly 5× because Tamil words pack multiple prepositions and case markers into single tokens.
  - **Tokens per parallel sentence** holds semantic payload invariant and is the single metric that should drive serving infrastructure capacity planning.

---

## 3. Open Gaps & Transition to Part B
- **No Open Gaps in Part A**: All requirements (A1 through A4) are completely fulfilled, standalone, and verifiable.
- **Ready for Part B**: Serving throughput, batch size scaling, KV-cache math, and goodput analysis will be addressed in the subsequent prompt.
