# Engineering Memo: Multilingual Tokenization & Inference Cost Audit

**To:** VP of Engineering / Leadership Team  
**From:** Systems & Inference Audit Team  
**Date:** September 11, 2026  
**Subject:** Correction of v0 Fertility Analysis and Indic Traffic Routing Strategy  

---

### 1. Executive Summary & Corrected Headline Numbers
The draft v0 finding asserting that *"Hindi will cost 6× more per request than English due to an inherent script property"* is **methodologically invalid**. When evaluated on parallel semantic units (FLORES-200 benchmark, 1,012 parallel sentences across 5 languages), the cost penalty is an artifact of **tokenizer vocabulary deficiency**, not the scripts themselves.

| Language | Script Family | GPT-2 Tok/Sent (Baseline) | GPT-2 Intent Ratio (vs Eng) | Indic-Aware Tok/Sent (MuRIL) | Corrected Intent Ratio (MuRIL) | Token Reduction via Routing |
|---|---|---|---|---|---|---|
| **English** | Latin | 26.7 | 1.00× | 27.3 | 1.00× | -2.0% |
| **Hindi** | Devanagari | 198.3 | **7.42×** | 31.6 | **1.16×** | **-84.1%** |
| **Kannada** | Dravidian | 363.0 | **13.58×** | 29.0 | **1.07×** | **-92.0%** |
| **Tamil** | Dravidian | 415.2 | **15.54×** | 28.9 | **1.06×** | **-93.0%** |
| **Telugu** | Dravidian | 346.6 | **12.97×** | 32.8 | **1.20×** | **-90.5%** |

Under an English-centric tokenizer (`gpt2`), Dravidian languages suffer a devastating **13×–15.5× token inflation** (byte fallback splitting aksharas into 3 UTF-8 byte tokens). Under an Indic-aware tokenizer (`google/muril-base-cased` or `xlm-roberta-base`), Hindi is only **1.16×** and Tamil is **1.06×** of English per sentence.

---

### 2. Root Cause of the v0 Distortion
1. **Wrong Metric**: `tok/word` inherently penalizes agglutinative languages. Tamil packages case markers and prepositions into a single orthographic word (exhibiting 25.0 tok/word), yielding a fake 20.3× ratio. The only cost-invariant denominator is **tokens per parallel request (intent)**.
2. **Asymmetric Casing & Whitespace**: `line.lower()` stripped 11.1% of English tokens while doing nothing for unicameral Indic scripts; `split(" ")` distorted word counts by 40% on double spaces.

---

### 3. Routing Recommendation
- **Immediate Action**: **DO NOT** budget a flat 6× server capacity for Indic traffic on a monolithic GPT-2/Llama architecture. 
- **Tiered Architecture**: Implement **Script-Aware Language Routing** at the API gateway:
  - **Latin / Code / English**: Route to dense, English-optimized foundation models.
  - **Indic Scripts (Devanagari, Dravidian)**: Route to models with multilingual / Indic vocabularies (e.g. Gemma/Llama-3 with $\ge 128\text{k}$ vocabularies, or MuRIL/Sarvam backends). 
  - **Impact**: Slashes Indic prefill latency by **85–93%**, prevents quadratic attention slowdowns, and reduces KV-cache memory pressure by **10×–14×** on Dravidian requests.

---

### 4. The Single Biggest Caveat
**Benchmark vs Production Shift**: FLORES-200 comprises clean, formal Wikipedia text in native scripts. Real-world Indian consumer queries feature heavy **code-mixing (Hinglish/Tanglish)** and **Romanized script** (e.g., *"mera order kab aayega"*). An Indic-native tokenizer trained solely on Devanagari will fragment Romanized Hindi words into byte chunks. Tokenizer benchmarking must be validated on actual production logs before finalizing fleet sizing.

---

### 5. Production Metric to Monitor
**`p95 prompt_tokens_per_completed_task` segmented by detected input language.**  
If our routing thesis holds, this metric will show near-parity ($\le 1.30\text{x}$) between Indic and English user sessions. If this ratio spikes above $2.5\text{x}$ in production, it signals unhandled Romanized transliteration or vocabulary mismatch.
