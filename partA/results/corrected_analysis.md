# Part A3: Corrected Multilingual Tokenizer & Metric Analysis

## Executive Summary

This study evaluates three distinct tokenizer architectures across 5 parallel languages (1,012 sentences each from FLORES-200 `devtest`):
1. **`gpt2`** (tiktoken BPE, vocab: 50,257) — Baseline English-centric tokenizer.
2. **`xlm-roberta-base`** (SentencePiece BPE, vocab: 250,002) — Massively multilingual 100-language tokenizer.
3. **`google/muril-base-cased`** (WordPiece, vocab: 197,285) — Indic-specialized model pretrained on 17 Indian languages and English.

## Full Benchmark Measurement Matrix

| Tokenizer | Vocab Size | Lang | Tokens | Tok/Sentence | Sent Ratio (vs Eng) | Tok/Word | Word Ratio | Tok/Grapheme | Grapheme Ratio | Tok/Byte | Byte Ratio |
|---|---|---|---|---|---|---|---|---|---|---|---|
| `gpt2` | 50,257 | **eng** | 27,044 | 26.72 | **1.0x** | 1.23 | 1.0x | 0.205 | 1.0x | 0.205 | 1.0x |
| `gpt2` | 50,257 | **hin** | 200,688 | 198.31 | **7.42x** | 7.83 | 6.34x | 2.335 | 11.39x | 0.595 | 2.9x |
| `gpt2` | 50,257 | **kan** | 367,366 | 363.01 | **13.58x** | 22.82 | 18.48x | 4.065 | 19.84x | 0.979 | 4.78x |
| `gpt2` | 50,257 | **tam** | 420,171 | 415.19 | **15.54x** | 25.05 | 20.28x | 4.213 | 20.56x | 0.997 | 4.87x |
| `gpt2` | 50,257 | **tel** | 350,748 | 346.59 | **12.97x** | 20.71 | 16.77x | 4.581 | 22.35x | 0.992 | 4.84x |
| `xlm-roberta-base` | 250,002 | **eng** | 30,661 | 30.3 | **1.0x** | 1.4 | 1.0x | 0.232 | 1.0x | 0.232 | 1.0x |
| `xlm-roberta-base` | 250,002 | **hin** | 38,221 | 37.77 | **1.25x** | 1.49 | 1.06x | 0.445 | 1.91x | 0.113 | 0.49x |
| `xlm-roberta-base` | 250,002 | **kan** | 41,459 | 40.97 | **1.35x** | 2.58 | 1.84x | 0.459 | 1.97x | 0.11 | 0.48x |
| `xlm-roberta-base` | 250,002 | **tam** | 41,354 | 40.86 | **1.35x** | 2.47 | 1.76x | 0.415 | 1.78x | 0.098 | 0.42x |
| `xlm-roberta-base` | 250,002 | **tel** | 40,372 | 39.89 | **1.32x** | 2.38 | 1.7x | 0.527 | 2.27x | 0.114 | 0.49x |
| `google/muril-base-cased` | 197,258 | **eng** | 27,581 | 27.25 | **1.0x** | 1.26 | 1.0x | 0.209 | 1.0x | 0.209 | 1.0x |
| `google/muril-base-cased` | 197,258 | **hin** | 31,924 | 31.55 | **1.16x** | 1.24 | 0.99x | 0.371 | 1.78x | 0.095 | 0.45x |
| `google/muril-base-cased` | 197,258 | **kan** | 29,382 | 29.03 | **1.07x** | 1.82 | 1.45x | 0.325 | 1.56x | 0.078 | 0.37x |
| `google/muril-base-cased` | 197,258 | **tam** | 29,204 | 28.86 | **1.06x** | 1.74 | 1.38x | 0.293 | 1.4x | 0.069 | 0.33x |
| `google/muril-base-cased` | 197,258 | **tel** | 33,190 | 32.8 | **1.2x** | 1.96 | 1.56x | 0.433 | 2.07x | 0.094 | 0.45x |

## Cross-Tokenizer Comparison on User Intent (Tokens / Parallel Sentence)

| Language | GPT-2 Tok/Sent | XLM-RoBERTa Tok/Sent | MuRIL Tok/Sent | XLM vs GPT-2 Efficiency Gain | MuRIL vs GPT-2 Efficiency Gain |
|---|---|---|---|---|---|
| **eng** | 26.72 | 30.3 | 27.25 | **-13.4%** | **-2.0%** |
| **hin** | 198.31 | 37.77 | 31.55 | **+81.0%** | **+84.1%** |
| **kan** | 363.01 | 40.97 | 29.03 | **+88.7%** | **+92.0%** |
| **tam** | 415.19 | 40.86 | 28.86 | **+90.2%** | **+93.0%** |
| **tel** | 346.59 | 39.89 | 32.8 | **+88.5%** | **+90.5%** |

## The Decisive Architectural Question

### Which single number should drive a routing-and-cost decision, and why?

> **Answer: Tokens per parallel sentence (tokens per unit of user intent), evaluated alongside total request sequence length.**

### Rigorous Technical Justification

1. **Invariance of the Semantic Payload**:
   - Cost in LLM inference is billed per token generated and processed. However, users do not pay or prompt in units of words or characters; they prompt in units of **intent** (e.g. answering a question, summarizing a document, executing a task).
   - In a parallel corpus, every sentence expresses the *exact same proposition*. Therefore, dividing total tokens by parallel sentences isolates tokenizer efficiency from typographical script artifacts.

2. **Failure of `tok/word` as a Cost Driver**:
   - `tok/word` implicitly assumes that 'word' is an invariant unit across languages. In reality, morphological typology invalidates this assumption:
     - English is predominantly analytic and isolating (e.g. *'with the child'* = 3 words).
     - Hindi has postpositions (e.g. *'बच्चे के साथ'* = 3 words).
     - Tamil, Kannada, and Telugu are highly agglutinative, suffixing prepositions, cases, and verb inflections into a single orthographic word (e.g. Tamil *'குழந்தையுடன்'* = 1 word).
   - On GPT-2, Tamil exhibits **25.05 tok/word** compared to English **1.23 tok/word** (a staggering **20.28x ratio**). But on a sentence basis, Tamil requires **415.19 tokens** vs English **26.72 tokens** (**15.54x ratio**). The word metric artificially exaggerates the cost penalty by nearly 5x simply because Dravidian words are agglutinative and information-dense.

3. **Failure of `tok/char` as a Cost Driver**:
   - Unicode code point characters count vowels and viramas as separate characters, creating a false perception of low fertility per character that does not correspond to visual reading speed or token generation throughput.

4. **Implications for Routing & Infrastructure**:
   - Under GPT-2, Dravidian queries consume **13.0x to 15.5x** more tokens per sentence than English, causing massive KV cache bloat, rapid context window exhaustion, and quadratic attention slowdowns.
   - Switching to an Indic-aware tokenizer (`google/muril-base-cased` or `xlm-roberta-base`) slashes token count by **81.0% to 93.0%** across Hindi, Kannada, Tamil, and Telugu.
   - Under MuRIL, Tamil requires only **28.86 tok/sentence** vs English **27.25 tok/sentence** (**1.06x parity**), and Hindi requires **31.55 tok/sentence** (**1.16x**).
   - Routing Indic traffic to an Indic-specialized model/tokenizer achieves a direct **10x–14x reduction in prefill latency, KV-cache footprint, and per-query serving cost** for Dravidian languages.
