# Multilingual Evaluation Corpus (FLORES-200 Devtest)

## 1. Exact Source & License
- **Source**: Canonical Meta AI NLLB (No Language Left Behind) FLORES-200 benchmark evaluation dataset.
- **Archive URL**: `https://dl.fbaipublicfiles.com/nllb/flores200_dataset.tar.gz`
- **Citation**: NLLB Team et al., "No Language Left Behind: Scaling Human-Centered Machine Translation", arXiv:2207.04672 (2022).
- **License**: Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0).

## 2. Corpus Size & Language Coverage
The corpus consists of professionally translated, strictly parallel sentences across five languages (English + 1 Indo-Aryan language + 3 Dravidian languages):

| File | Language | Script | ISO 639-3 / FLORES Code | Sentence Count | Size (Bytes) | Size (KB) |
|---|---|---|---|---|---|---|
| `eng.txt` | English | Latin (`Latn`) | `eng_Latn` | 1,012 | 133,108 | 130.0 KB |
| `hin.txt` | Hindi | Devanagari (`Deva`) | `hin_Deva` | 1,012 | 338,106 | 330.2 KB |
| `kan.txt` | Kannada | Kannada (`Knda`) | `kan_Knda` | 1,012 | 376,492 | 367.7 KB |
| `tam.txt` | Tamil | Tamil (`Taml`) | `tam_Taml` | 1,012 | 422,653 | 412.7 KB |
| `tel.txt` | Telugu | Telugu (`Telu`) | `tel_Telu` | 1,012 | 354,723 | 346.4 KB |
| **Total** | **5 Languages** | **4 Distinct Scripts** | - | **5,060** | **1,625,082** | **1.55 MB** |

Each line $i$ across all five files corresponds to the exact same source document and semantic unit.

## 3. Domain
- **Origin**: Wikipedia articles, Wikinews, and Wikivoyage spanning diverse topics: geography, history, politics, science, culture, and travel.
- **Translation Quality**: Commissioned professional human translations followed by multi-stage quality control and human post-editing checks by native speakers.

## 4. Preprocessing Applied
- Extracted directly from the `devtest` split of the official distribution.
- Encoding: Pure UTF-8 without Byte Order Mark (BOM).
- Line endings standardized to Unix LF (`\n`).
- Stripped empty lines; preserved original Unicode code points and punctuation. No lossy lowercasing or whitespace modification was applied to the source files, enabling fair evaluation under various tokenizer preprocessing regimes.

## 5. What This Corpus CANNOT Tell You (Critical Boundary Analysis)
While FLORES-200 provides a gold-standard parallel baseline for measuring linguistic compression across standardized scripts, it has distinct operational boundaries that must not be overlooked:
1. **Domain & Register Bias**: The texts are formal, edited, expository prose written in standard literary registers. In customer-facing production systems (e.g. conversational LLM assistants, customer support chats, WhatsApp messaging), Indic text overwhelmingly features informal grammar, non-standard spellings, phonetic transliterations, and heavy colloquialisms.
2. **Code-Switching & Romanized Transliteration (Hinglish/Tanglish)**: A dominant fraction of real-world Indian digital traffic is written in Roman script (e.g. *"Kya haal hai"*, *"Naan nalla irukken"*), which exhibits vastly different tokenization dynamics than native Devanagari or Dravidian scripts. FLORES-200 contains zero code-switched or Romanized Indic data.
3. **Syntactic Payload vs. Interactive Query Distribution**: Benchmark sentences average 20–30 words with complex subordinate clauses, whereas real LLM serving workloads often comprise concise user prompts (3–10 words) paired with system prompts containing English Markdown, JSON schemas, code snippets, and few-shot formatting tokens. Consequently, this corpus reveals intrinsic script-level tokenizer efficiency, but does not simulate prompt-template overhead or chat dialog distributions.
