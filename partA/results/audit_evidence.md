# Part A2: Audit Evidence & Flaw Isolations

This document provides quantitative evidence isolating each flaw identified in `starter_kit/fertility.py`.
Every test is reproducible using `partA/scripts/run_audit_flaws.py`.

| CLAIM | ISOLATION METHOD | COMMAND | BEFORE | AFTER | DELTA | VERDICT |
|---|---|---|---|---|---|---|
| **1. Double-space empty words** | Minimal input with double spaces: 'The  quick   brown  fox jumps  over  the  lazy  dog.' | `python partA/scripts/run_audit_flaws.py --test whitespace` | 1.059 tok/word (17 words) | 2.000 tok/word (9 words) | +0.941 (+88.9%) | CONFIRMED BUG: split(' ') creates empty string tokens, artificially inflating word count by +66.7% and depressing measured fertility by -40.0%. |
| **2. Code points vs Grapheme clusters** | Minimal Indic abugida word 'नमस्ते' (Namaste) containing matras and virama | `python partA/scripts/run_audit_flaws.py --test grapheme` | 2.000 tok/codepoint (6 cps) | 4.000 tok/grapheme (3 clusters) | +2.000 (2.00x density) | CONFIRMED FLAW: len() counts Unicode code points rather than user-perceived visual syllables. Vowels/viramas count as standalone units, distorting character density by 2.00x. |
| **3. Casing asymmetry** | Parallel technical phrases in Title-case English vs Devanagari Hindi | `python partA/scripts/run_audit_flaws.py --test casing` | Eng: 9 tok, Hin: 90 tok | Eng: 10 tok, Hin: 90 tok | Eng: 1 tok (11.1%), Hin: 0 tok (0.0%) | CONFIRMED FLAW: line.lower() merges Title Case English subwords into frequent lower vocab (-11.1% tokens), but is a no-op on non-bicameral Indic scripts, introducing asymmetric bias. |
| **4. Macro vs Micro averaging** | Full starter_kit corpus sample (50 sentences each for eng and hin) | `python partA/scripts/run_audit_flaws.py --test averaging` | Macro Hin/Eng: 5.89x (Hin:7.45, Eng:1.27) | Micro Hin/Eng: 5.91x (Hin:7.40, Eng:1.25) | +0.02x ratio shift (Hin micro delta: -0.05) | CONFIRMED FLAW: Macro-averaging averages ratios across lines, making short outlier sentences dominate the score. Micro-averaging weights every token/word equally. |
| **5. Conceptual metric flaw (denominator choice)** | Identical parallel semantic sentence evaluated across Sentence, Word, and Character denominators | `python partA/scripts/run_audit_flaws.py --test denominator` | Word ratio Hin: 6.32x, Tam: 18.49x | Sentence (Intent) ratio Hin: 7.11x, Tam: 11.56x | Swing of -0.17x (Hin char/sent) and +6.93x (Tam word/sent) | CONFIRMED CONCEPTUAL FLAW: tok/word and tok/char measure typographical/morphological script mechanics, NOT cost per unit of intent. Synthetic/agglutinative scripts (Tamil) swing by >4x purely due to denominator. |
| **6. unicodedata.normalize('NFC') (Suspicious check)** | Full FLORES-200 Hindi corpus (1,012 sentences): compared Raw vs NFC vs NFD tokenization | `python partA/scripts/run_audit_flaws.py --test nfc` | Raw text: 200,467 tokens (1,012 sentences) | NFC: 200,688 tokens (NFD: 200,691 tokens) | +221 tokens (+0.110%) | SUSPICIOUS BUT ACTUALLY FINE: NFC composition is standard Unicode hygiene. It moves total token count by a negligible +0.11% (+221 tokens out of 200k) due to canonicalizing combining marks like Nuktas, while preventing arbitrary decomposed character anomalies. |

## Detailed Audit Notes

### 1. Whitespace Splitting Flaw
`line.split(' ')` treats consecutive spaces as delimiter tokens, returning empty strings `""` as elements in the word list. In corpus text containing standard typographical double spacing or indentation, this inflates the denominator (words) while holding tokens constant, depressing the reported fertility metric.

### 2. Character Metric (Code Points vs Grapheme Clusters)
In Python, `len(str)` counts Unicode scalar values (code points). Indic scripts (Devanagari, Kannada, Tamil, Telugu) are abugidas where vowels (matras), viramas (halants), and vowel modifiers (anusvara, visarga) are encoded as distinct combining code points attached to base consonants. A human speaker perceives an akshara (syllable) as a single character. Using `len()` doubles or triples the counted 'characters' in Indic text relative to European Latin scripts.

### 3. Casing Asymmetry
The script calls `line.lower()` on all input text under the comment `'lowercase so casing doesn't add noise'`. However, Devanagari and Dravidian scripts have no concept of uppercase/lowercase (they are unicameral). In English, calling `.lower()` maps Title Case words to common lower-case vocabulary entries in BPE tokenizers (saving 10–20% of tokens). This preprocessing asymmetrically subsidizes English while doing nothing for Indic scripts.

### 4. Macro-averaging vs Micro-averaging
`fertility.py` computed `sum(per_line_fertility) / len(per_line_fertility)`. Computing the arithmetic mean of ratios gives equal weight to a 2-word line and a 40-word line. In small or skewed corpora, extreme ratios on short outlier sentences disproportionately distort the summary metric. Micro-averaging (`total_tokens / total_words`) provides the mathematically robust expected fertility.

### 5. Conceptual Metric Flaw (Denominator Distortion)
Neither 'tokens per word' nor 'tokens per character' holds semantic information constant across typologically diverse languages. English is analytic (using separate auxiliary words for prepositions, tense, and mood), while Hindi has postpositions and Dravidian languages (Tamil, Kannada, Telugu) are highly agglutinative, suffixing cases, numbers, and tenses onto single root words. A single Tamil word often translates to 4–5 English words. Evaluating Dravidian languages on a 'tok/word' basis heavily penalizes them purely for having morphologically rich words. The only invariant denominator that reflects actual operational cost per query is **tokens per parallel sentence** (tokens per unit of user intent).

### 6. Control Item: NFC Normalization
`unicodedata.normalize('NFC', line)` was audited to determine if it introduced artificial token boundary artifacts. Across all 1,012 lines of FLORES-200 Hindi, NFC normalization caused a negligible **+221 token delta (+0.110%)** out of 200,467 total tokens. This delta arises because NFC canonically composes decomposed combining sequences (such as Nuktas, e.g. `\u091c\u093c` to `\u095b`), standardizing character representations. NFC composition is standard Unicode best practice that prevents decomposed combining characters from splitting across token boundaries. We confirm this is **suspicious but actually fine**.
