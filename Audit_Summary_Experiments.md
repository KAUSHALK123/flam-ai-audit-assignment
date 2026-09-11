# Audit Summary: Flaws, Constraints & Hypothesis Testing

## 1. Flaws Identified in the Original Pipeline
- **Whitespace Splitting:** `line.split(" ")` treats consecutive spaces as empty strings. A double space artificially inflates the word count and incorrectly drops the fertility ratio.
- **Character Counting Flaw:** `len(line)` counts Python Unicode code points, ignoring how Indic abugidas use combining marks (matras) and viramas. This severely penalizes Indic scripts when comparing tokens per character.
- **Punctuation Conflation:** Trailing punctuation attaches to words under space-splitting, conflating punctuation tokens with actual lexical tokens.
- **Macro-Averaging Distortion:** `fertility.py` averages the ratios of individual lines (`sum(ratios)/n`) rather than taking a true micro-average (`total_tokens / total_words`), allowing short lines to heavily skew the overall metric.
- **Throughput Misinterpretation:** The intern read `reported_tok_s` as generation speed, but it actually includes prefill tokens. Real generation goodput is much lower for long prompts.
- **Capacity/Preemption Ignorance:** Scaling batch size to 48 hits 97% KV cache utilization and triggers massive sequence preemption, destroying throughput.

## 2. Constraints Encountered
- **Starter Kit Immutability:** Strict rule against modifying any original files (`fertility.py`, corpus, logs). 
- **Windows Console Encoding:** Python stdout on Windows defaults to `cp1252`, which crashed on Indic text. Fixed by explicitly setting `$env:PYTHONIOENCODING="utf-8"`.
- **Grapheme Cluster Support:** Python's standard library lacks native UAX #29 grapheme clustering. Had to import the third-party `regex` module (`\X`) to properly count visual syllables.
- **Tiny Sample Size:** The starter corpus only had 10 sentences per language, making the macro-averaging highly sensitive to single-line quirks (like the double spaces in line 7/10).

## 3. Things Changed & Methodological Adherence
- **Zero Modifications to Source:** No starter files were patched or fixed, even to correct obvious bugs.
- **Isolated Playgrounds:** All new test scripts were isolated entirely in a new `playground/` directory (`run_phase_c.py`, `run_phase_d.py`, etc.).
- **Output Artifacts:** All documentation and analysis were written cleanly into a separate `notes/` directory to prevent workspace contamination.

## 4. AI Usage (AI_USAGE Summary)
- **What AI solved:** Accelerated the drafting of isolated python testing scripts (`playground/run_phase_c.py`, etc.), formatted complex Markdown tables, and mapped out the exact math for KV cache byte footprint calculation.
- **What AI was NOT used for:** AI was strictly prohibited from fabricating numeric outputs, executing unauthorized bug fixes on the starter code, or making final definitive business recommendations. Every recorded number was natively generated via CLI execution.

---

## 5. Phase B Hypotheses & Corresponding Experiments
The table below maps the variables flagged as `HYPOTHESIS — REQUIRES EXPERIMENT` during the Phase B code walkthrough directly to the experiments conducted to test them.

| Code Variable | Phase B Hypothesis | Corresponding Experiment Done | 1-Line Result Description |
| :--- | :--- | :--- | :--- |
| `words` | Double spaces create empty strings inflating word count. | **Double Space Test** (`"  book"`) | Double spaces generated 3 words under `.split(" ")`, artificially dropping fertility. |
| `words` | Punctuation attaches to words, distorting counts. | **Trailing Punctuation Test** (`"book."`) | ASCII punctuation adds 1 token; Indic danda (`।`) adds 2 tokens, conflating punctuation overhead with word fertility. |
| `tokens` | GPT-2 lacks Indic merges, relying on heavy byte fallback. | **Single Word Test** (Across 5 languages) | English "book"=1 token; Dravidian words collapse to exactly 1 token per raw UTF-8 byte (e.g., 24 bytes = 24 tokens). |
| `tokens` | Non-Latin numerals suffer severe token fragmentation. | **Numerals Test** (`"123"` vs native digits) | Latin "123" takes 1 token; native Kannada/Tamil/Telugu digits take 9 tokens (3 per digit). |
| `chars` | `len(line)` overcounts code points vs actual visual characters. | **Matra/Grapheme Test** (e.g. `"किताबें"`) | "किताबें" has 3 visual syllables (graphemes) but 7 Unicode code points, invalidating `tokens/char` comparisons. |
| `line` (lower) | `.lower()` alters English BPE but does nothing for Indic scripts. | *(Identified for future testing)* | Modifies case-sensitive GPT-2 token mappings for Latin text but is a no-op for casing-free scripts. |
| `per_line_...` | Macro-averaging line ratios distorts overall metrics. | **Report Reproduction** (Phase E) | Unrounded macro-averages perfectly reproduced the intern's numbers, verifying the mathematical skew. |
| `ratio` | Denominator selection drastically alters the final penalty ratio. | **Metric Denominator Swing** (Phase D) | On a parallel sentence, the Hindi penalty ratio swung from 3.0x (bytes) up to 11.4x (graphemes) based purely on the chosen denominator. |
