# Skeptical Self-Review & Verification Audit

This self-review critically inspects the completed Part A, Part B, and Part C deliverables in `flam_assignment/` against strict audit standards.

---

## 1. Claims & Mathematical Derivations Backing

| Section / Claim | Verification Command / Derivation | Status |
|---|---|---|
| **A2: Whitespace split bug** | `python partA/scripts/run_audit_flaws.py --test whitespace` $\to$ Word count jumps 9 to 17 (+88.9%), fertility drops 2.00 to 1.06 (-47.1%). | **CONFIRMED** |
| **A2: Grapheme vs code points** | `python partA/scripts/run_audit_flaws.py --test grapheme` $\to$ `'नमस्ते'` 6 code points vs 3 grapheme clusters (2.0x density distortion). | **CONFIRMED** |
| **A2: Casing asymmetry** | `python partA/scripts/run_audit_flaws.py --test casing` $\to$ English drops 1 token (-11.1%), Hindi 0.0% delta. | **CONFIRMED** |
| **A2: Macro vs Micro averaging** | `python partA/scripts/run_audit_flaws.py --test averaging` $\to$ Macro 5.89x vs Micro 5.91x on starter sample. | **CONFIRMED** |
| **A2: Denominator swing** | `python partA/scripts/run_audit_flaws.py --test denominator` $\to$ Tamil swings from 18.5x (word) to 11.6x (sentence) on GPT-2. | **CONFIRMED** |
| **A2: NFC Normalization hygiene** | `python partA/scripts/run_audit_flaws.py --test nfc` $\to$ Raw 200,467 tokens vs NFC 200,688 tokens (+0.110% delta, standardizes Nuktas). | **CONFIRMED** |
| **A3: Cross-tokenizer parity** | `python partA/scripts/fertility_v2.py` across 1,012 sentences $\times 5$ languages $\times 3$ tokenizers $\to$ Tamil drops from 415.2 tok/sent (GPT-2) to 28.9 tok/sent (MuRIL), a 93.0% token reduction. | **CONFIRMED** |
| **B1: KV Cache Bytes per Token** | $2 \times 28 \times 8 \times 128 \times 2 = 114,688\text{ bytes} = 112\text{ KiB}$. Per 4096 seq: $4096 \times 112\text{ KiB} = 448\text{ MiB} = 0.46976\text{ GB}$. | **CONFIRMED** |
| **B1: Usable KV Memory Pool** | $24\text{ GB} \times 0.92 - (4.2\text{B} \times 2 + 1.6\text{GB}) = 22.08\text{ GB} - 10.00\text{ GB} = 12.08\text{ GB}$. | **CONFIRMED** |
| **B1: Max Concurrent Sequences** | $12.08\text{ GB} / 0.46976\text{ GB} = 25.71\text{ sequences} \approx 25\text{ sequences}$. | **CONFIRMED** |
| **B1: Preemption Onset Match** | At batch 24: $93.3\%$ pool $\to$ logged `kv_cache_util = 0.93`, `preempted = 0`. At batch 32: $32 - 25 = 7$ overflow $\to$ logged `preempted = 7`. At batch 48: $48 - 25 = 23$ overflow $\to$ logged `preempted = 23`. Exact integer match. | **CONFIRMED** |
| **B3: Honest Goodput Batch 24** | Method 1: $(512 \times 24) / 61.16 = \mathbf{200.92\text{ tok/s}}$. Method 2: $1607.4 \times (512 / 4096) = \mathbf{200.93\text{ tok/s}}$. Agreement $< 0.01\text{ tok/s}$. | **CONFIRMED** |
| **B3: Short vs Long Goodput** | Short batch 16: $(256 \times 16) / 13.91 = \mathbf{294.5\text{ tok/s}}$. Long batch 16: $(512 \times 16) / 49.97 = \mathbf{163.9\text{ tok/s}}$. Goodput drops -44.3%. | **CONFIRMED** |
| **C: Reviewer Capacity & Scope** | $10\text{ h/wk} \times 3\text{ wk} = 30\text{ h} \times 24\text{ samples/h} = 720\text{ samples}$. Languages covered: $2 / 6 = 33.3\%$ (0% for Tam, Tel, Ben, Mar). | **CONFIRMED** |

---

## 2. Denominator and Unit Consistency Audit

- **Part A vs Part B Metric Harmonization**:
  - Part A proved that `tok/word` and `tok/char` are non-invariant across scripts, establishing **tokens per parallel sentence (intent)** as the true metric of user demand.
  - Part B demonstrated that `reported_tok_s` conflates prefill with decode tokens, establishing **generation tokens per second (goodput)** as the true metric of serving capacity.
  - **Connection**: Request sequence length in Part B ($S = \text{prompt\_len} + \text{gen\_len}$) is directly determined by the tokenizer efficiency derived in Part A. When Dravidian languages suffer a 15× token inflation on GPT-2, they consume 15× more KV cache per unit of intent, driving the system into preemption thrashing at concurrency levels as low as 2–3 requests.
  - **Verdict**: No unit contradictions exist. All denominators are explicitly specified as `tok/sentence`, `gen_tok/s`, or `KiB/tok`.

---

## 3. Part C Exact Constraint Compliance

- **Hardware**: Exactly 1× A100-80GB for 2 weeks ($14\text{ days} \times 24\text{ hours} = 336\text{ GPU-hours}$).
- **Human Resources**: Exactly 1 reviewer for 10h/week over 3 weeks ($30\text{ hours}$ total).
- **Linguistic Domain**: Reviewer speaks Hindi + Kannada ONLY. Exactly $0\text{ hours}$ of native review for Tamil, Telugu, Bengali, or Marathi.
- **Budget**: Exactly $\$0.00$ external API budget.
- **Verdict**: No rounded or relaxed constraints were used. The reviewer throughput ($720\text{ samples}$) directly proves that 96% of an 18,000-pair synthetic SFT dataset would enter training completely unverified.

---

## 4. Live 30-Minute Interview Re-Derivation Test

Could these numbers be derived cleanly on a whiteboard under pressure in 30 minutes?
1. **KV Cache Bytes/Token**:
   $$2 \times 28 \times 8 \times 128 \times 2 = 114,688\text{ bytes} = 112\text{ KiB}$$
   $$\text{Per 4096 seq} = 4096 \times 112\text{ KiB} = 448\text{ MiB}$$
   *Re-derivation time: 60 seconds.*
2. **Usable Pool & Concurrency Ceiling**:
   $$24 \times 0.92 = 22.08\text{ GB} - (8.4 + 1.6)\text{ GB} = 12.08\text{ GB}$$
   $$12.08\text{ GB} / 0.470\text{ GB} \approx 25.7\text{ sequences}$$
   *Re-derivation time: 90 seconds.*
3. **Goodput Dual Check**:
   $$\text{Method 1: } \frac{512 \times 24}{61.16} = 200.9\text{ tok/s}$$
   $$\text{Method 2: } 1607.4 \times \frac{512}{4096} = 1607.4 \times 0.125 = 200.9\text{ tok/s}$$
   *Re-derivation time: 60 seconds.*
4. **Preemption Predictions**:
   Batch 32: $32 - 25 = 7$ preemptions. Batch 48: $48 - 25 = 23$ preemptions.
   *Re-derivation time: 30 seconds.*

---

## 5. Final Audit Verdict
**Grade: SUBMISSION READY**. All claims are backed by reproducible code, logged data, and first-principles arithmetic without gaps or unverified assertions.
