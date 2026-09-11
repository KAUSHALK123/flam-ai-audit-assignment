# Executive Decision Memo: Multilingual Assistant Casualization Strategy

**To:** Leadership Team  
**From:** LLM Systems & Inference Audit  
**Date:** September 11, 2026  
**Subject:** Technical Decision: Tone Casualization Across 6 Indic Languages under Resource Constraints  

---

### ASSUMPTIONS
1. **Target Scope**: Tone casualization across 6 languages: Hindi (`hin`), Kannada (`kan`), Tamil (`tam`), Telugu (`tel`), Bengali (`ben`), Marathi (`mar`).
2. **Compute Budget**: Exactly 1× NVIDIA A100-80GB GPU for 2 weeks (14 days = 336 GPU-hours).
3. **Human Review Capacity**: Exactly 1 native speaker qualified in **Hindi + Kannada ONLY** for 10 hours/week over 3 weeks (30 hours total). Zero native evaluation capacity for Tamil, Telugu, Bengali, or Marathi.
4. **API Budget**: $0.00 external API spend (no GPT-4/Claude distillation or external MT services).
5. **Timeline**: Hard launch deadline in exactly 3 weeks (21 days).

---

### ARITHMETIC

#### 1. Human Review Bottleneck
- Review pace: 2.5 minutes per conversation turn (evaluating query + formal baseline + casual output for tone, grammar, and safety) = **24 samples/hour**.
- Total reviewable volume: $30\text{ hours} \times 24\text{ samples/hour} = \mathbf{720\text{ samples total}}$ ($360$ Hindi, $360$ Kannada).
- **Scope Coverage**: Exactly **$2 / 6$ languages (33.3%)**. 4 out of 6 languages have **0.0% human audit capacity**.

#### 2. Path (a): SFT on Synthetic Pairs (REJECTED)
- Data needed: Minimum 3,000 pairs/lang $\times 6 = \mathbf{18,000\text{ pairs}}$.
- Self-generation without API teacher: Prompting the base 4.2B model locally produces hallucinated, stiff translations (model collapse).
- Audit ratio: 720 reviewable pairs / 18,000 generated = **only 4.0% data audited**; **96% unvetted data** enters training weights. Catastrophic grammatical degradation on the 4 unreviewed languages is mathematically guaranteed.

#### 3. Path (b): $\le 1\text{B}$ Inference-Time Rewriter (REJECTED)
- Serving latency: Adds a secondary autoregressive decode phase ($200\text{ tokens}$), increasing end-to-end latency by **+40–60%** ($+800\text{ms}$ on L4).
- VRAM collapse: Co-hosting a 1B rewriter ($2\text{GB}$) alongside FLM-4B ($8.4\text{GB}$) shrinks the L4 KV cache pool from 12.08GB to 8.48GB (**-30% capacity**), dropping concurrency ceiling from 25 to 18 sequences and worsening preemption thrashing.

#### 4. Path (c): Prompt Engineering with Few-Shot Caching (RECOMMENDED)
- Data needed: 5 high-quality few-shot exemplars per language = **30 total exemplars**.
- Reviewer load: Crafting and verifying 10 exemplars (Hindi + Kannada) requires **$< 4\text{ hours}$**, leaving $> 26\text{ hours}$ for rigorous end-to-end output validation.
- Serving impact: With vLLM Prefix Caching (`enable_prefix_caching=True`), the 250-token system prompt is stored in KV cache once, yielding **$0\text{ms}$ decode latency penalty** and **0 additional parameters**.

---

### RECOMMENDATION
**Execute Path (c): System Prompting with Native Few-Shot Exemplars + vLLM Automatic Prefix Caching.**  
- Reject SFT and 1B Rewriter: Under zero API budget and a single Hindi/Kannada reviewer, fine-tuning guarantees unmonitored model degradation across 4 languages and severe inference memory bloat.
- Implementation: Deploy language-specific system prompts containing 3–5 curated native conversational turns and explicit regional honorific rules (e.g. *tum* vs *aap* in Hindi, *neenu* vs *neevu* in Kannada).

---

### SUCCESS METRIC
- **Metric**: Blind side-by-side win rate on casual tone against baseline formal output.
- **Threshold**: **$\ge 75\%$ win rate** on audited Hindi and Kannada test sets ($n = 300$ each), with **$< 1.0\%$ safety/grammar regression rate**, verified by Day 16.

---

### KILL CRITERION
- **Criterion**: If by **Day 8 (End of Week 1)**, side-by-side win rate in either Hindi or Kannada is **$< 60\%$**, or if the model exhibits code-switching degradation into English/gibberish on $\ge 5\%$ of prompts.
- **Action**: Immediately abort 6-language broad launch; descope launch to Hindi+Kannada only, and utilize the remaining A100 GPU compute to train a targeted 2-language LoRA adapter.

---

### DAY-1 EXPERIMENT
On 1× A100-80GB, evaluate FLM-4B-Instruct on 50 baseline prompts across Hindi and Kannada under 3 prompting variants:
1. *Zero-Shot*: System instruction only (*"Respond casually as a close peer"*).
2. *Few-Shot (3 turns)*: System instruction + 3 curated conversational pairs with colloquial particles (*yaar*, *bhai*, *ri*).
3. *Few-Shot + Negative Constraints*: Added explicit negative rules forbidding archaic formal suffixes.

Have the reviewer score all $2 \times 50 \times 3 = 300$ responses within 10 hours on Day 2 to establish the winning prompt template.
