# How we got the fertility targets and 196,608

## Why fertility drives the design

Fertility is tokens per unit of meaning. Under a fixed 14T budget, a language
with high fertility silently receives fewer documents and less knowledge for the
same token spend. The lecture's phrasing: size the Indic corpus **in
information, not in tokens**.

## From grouped rows to per-language targets

An earlier draft used five grouped rows (English, Hindi/Pa/Ur, Bn/Mr/Gu/Or/As,
Dravidian, Hinglish). Adversarial review showed each group paired a solved
language with an unsolved one, so every group target was simultaneously slack
and unreachable.

Measured reference is a published 200K Indic tokenizer (MUTANT-Indic /
IndicSuperTokenizer, arXiv 2511.03237, Table 3). Its per-language figures:

| Language | Measured at 200K | Our target | Status |
|---|---|---|---|
| English | 1.34 | ≤1.30 | small gap |
| Hindi | 1.23 | ≤1.25 | already met |
| Punjabi | 1.39 | ≤1.40 | already met |
| Urdu | 1.44 | ≤1.45 | already met |
| Marathi | 1.63 | ≤1.60 | small gap |
| Odia | 1.65 | ≤1.65 | met |
| Bengali | 1.74 | ≤1.70 | small gap |
| Gujarati | 1.77 | ≤1.75 | small gap |
| Assamese | 1.85 | ≤1.80 | small gap |
| Telugu | 1.88 | ≤1.85 | small gap |
| Tamil | 2.12 | ≤2.05 | 3% improvement needed |
| Kannada | 2.19 | ≤2.10 | 4% improvement needed |
| **Malayalam** | **2.30** | **≤2.20** | **binding constraint, 4.3%** |

**Malayalam, not Tamil, is the binding number.** An earlier draft quoted Tamil
2.12 as "the ceiling" while grouping Malayalam into the same ≤2.00 row — which
would have required a 13% improvement on Malayalam while advertising 6% on
Tamil. Non-word domains:

| Domain | Target | Why |
|---|---|---|
| Code | ≥3.3 chars/token | Words are the wrong unit; indentation runs handled explicitly |
| Math/science | ≥3.0 chars/token | Digits split 0–9 — arithmetic accuracy beats compression |
| Agentic/tool traces | ≤1.50 tokens/word | Reserved role, tool-call, result, FIM tokens keep JSON cheap |

## The generic tax being removed

An earlier draft claimed generic tokenizers cost Indic users "3–8× English".
That is wrong at the low end. Against the LLaMA-4 row of the same table
(English 1.34):

| Language | Generic | Ratio to English |
|---|---|---|
| Urdu | 1.76 | 1.31× |
| Hindi | 1.83 | 1.37× |
| Bengali | 2.93 | 2.19× |
| Punjabi | 3.23 | 2.41× |
| Gujarati | 3.37 | 2.51× |
| Tamil | 5.87 | 4.38× |
| Odia | 10.51 | **7.84×** |

Corrected to **1.3–7.8×**. Hindi and Urdu are already near parity on the very
tokenizer being criticised, so **dispersion, not the floor, is the problem**.
Overstating a 3× floor would have made the Hindi target look like a large win
when the published gap is 0.03 tokens per word.

The chart on the page also had both tails drawn in the wrong rows — Odia's 10.51
plotted in a row rendered at 6.53, Tamil's 5.87 in a row rendered at 9.07. Now
each language has its own bar at its own measured value, 60px per token/word.

## Cost equity, stated honestly

Principle: an Indian-language user should pay at most **1.4×** the English token
count for the same content. Against English at 1.30, the threshold is 1.82.

| Holds (8 of 12) | Ratio |
|---|---|
| Hindi | 0.96× |
| Punjabi | 1.08× |
| Urdu | 1.12× |
| Marathi | 1.23× |
| Odia | 1.27× |
| Bengali | 1.31× |
| Gujarati | 1.35× |
| Assamese | 1.38× |

| Breaks (4 of 12) | Ratio |
|---|---|
| Telugu | 1.42× |
| Tamil | 1.58× |
| Kannada | 1.62× |
| Malayalam | 1.69× |

An earlier draft asserted the 1.4× principle universally while listing a
Dravidian target of 2.00, i.e. 1.54×. The contradiction is now stated as a
limitation: closing it needs morphology-aware segmentation, not more vocabulary.

## Vocabulary size

### Rejection chain

| Option | Vocab | Verdict |
|---|---|---|
| Sarvam-class | 68K | Indic fertility fine, no headroom for code, math, tool tokens at 40B |
| Llama/Param-class | 128K | With 10 Indic scripts + code + math competing, Dravidian starves |
| **Selected** | **196,608** | 192 × 1024. The 200K reference reaches SOTA here and beats a 262K frontier vocab on 20 of 24 languages; its own conclusion is that gains flatten past 200K |
| Gemma-class | 262K | Extra 65,536 slots cost only ~0.40B params (1.0% of 40B) — **affordable**, rejected because measured fertility does not improve |

The 262K row is deliberately *not* rejected on cost. Review confirmed the
affordability, so rejecting it on price would have been dishonest.

### Allocation

| Block | Slots | Purpose |
|---|---|---|
| English, Latin, code, math, symbols, digits | 94K | Keeps code ≥3.3 chars/token without borrowing from Indic |
| 10 Indic scripts / 12 scheduled languages | 76K | Dravidian-weighted — Malayalam and Kannada bind |
| Romanized / code-mixed | 12K | Hinglish as a first-class register |
| Other multilingual, byte fallback, reserved | 10K | 256 byte symbols (UNK structurally impossible) + ~3.2K reserved role/tool/FIM |
| **Total** | **192K = 196,608** | exact |

Two corrections here:
- An earlier draft listed 96 + 78 + 12 + 10 = 196 binary-K = **200,704**, over-allocating
  by 4,096 slots against a vocabulary claimed to be 128-aligned. Now 94 + 76 + 12 + 10 = 192K exactly.
- "12 Indic scripts" was wrong. The 12 scheduled languages span **10 scripts** —
  Devanagari carries Hindi and Marathi, Bengali-Assamese carries Bengali and
  Assamese.

### Embedding cost

```
196,608 × 6,144 = 1,207,959,552 = 1.21B parameters = 3.02% of 40B
```

Tied input/output embeddings.

## Construction

Byte-level BPE; akshara-aware pre-tokenization per Brahmic script; ZWJ/ZWNJ
preserved; digits split; superword merges evaluated as a second stage.
Tokenizer training mix ≈ 35% English+code, 45% Indic (fertility-driven
upsample), 10% math, 10% other.

Candidates at 160K, 196,608, 224K, 256K compared on equal-FLOP 1B proxies.
196,608 ships only if per-language fertility, bits-per-byte, throughput and
HumanEval/MBPP all hold. Per-language fertility tracked in CI on every release,
using the harness built for the Session 2 tokenizer assignment.

## Sources

- Per-language fertility at 200K, and the LLaMA-4 comparison row — arXiv 2511.03237, Table 3
- Fertility as a data-allocation decision — Session 3 lecture, widget 18
