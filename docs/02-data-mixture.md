# How we got the data mixture

Everything here is a share of the 14T budget derived in `01-token-budget.md`.

## Partition before mixture

Three pools are separated *before* any bucket percentage is chosen, because two
of them must be invisible to the quality selector.

```
main scored pool   12.67T   90.5%   ranked by an OPUS-style batch-quality selector
always-on floor     1.12T    8.0%   selector never sees it
anneal reserve      0.21T    1.5%   held out of both, spent only at LR cooldown
                   ------
                   14.00T
```

The 8% floor is taken from V4, where an English-trained selector systematically
under-scored Brahmic text. The published fix reserved 8% of every batch across
10 scripts (66.9B tokens of guaranteed Indic inside that channel). We keep the
percentage and the mechanism.

**Important accounting note.** The floor is *not* a fifth stage. Its tokens are
already counted inside the bucket percentages below. An earlier draft listed it
as an additional row, which made the stages sum to 16.2T instead of 15.0T — a
double-count caught in self-review.

## Stages

```
A  breadth        10.80T
B  capability      2.60T
C  long context    0.40T
D  anneal          0.21T
                  ------
                  14.01T ≈ 14.0T
```

## Stage A — 10.8T

| Bucket | % | ~Tokens | Why |
|---|---|---|---|
| English web, Indian-authored upweighted | 30% | 3.24T | General capability; the English distribution itself becomes India-centric |
| Code + PR/issue threads | 18% | 1.94T | SE patterns; agentic priors start here, not at SFT |
| India-context **English** | 15% | 1.62T | Judgments, gazettes, Parliament, RBI/SEBI, NCERT, Census |
| Math + science | 12% | 1.30T | Structured reasoning; supplies RL's verifiable subset |
| Other multilingual | 7% | 0.76T | Cross-lingual transfer into low-resource Indic |
| **Indic native script** | **7%** | **0.76T** | Supply-capped — see ledger |
| Books / long-form | 6% | 0.65T | Discourse coherence, long-range dependency |
| Verified synthetic reasoning | 4.5% | 0.49T | Kept only when the final answer is checkable |
| Romanized real UGC | 0.5% | 0.05T | All the genuinely human romanized text sourceable |

Percentages sum to 100.0; tokens sum to 10.8T.

### Why code is 18%

Two published results bound it from both sides:
- code lifts non-code reasoning by up to +8.2% in the best case (Aryabumi et al.)
- an excessive share measurably hurts morphology and factual recall (Petty et al.)

Run-wide code share is **17.6%**, not 18%: `(1.94T Stage A + 0.52T Stage B) / 14T`.
An earlier draft advertised "code capped at 18%" while Stage B pushed the run
above it — corrected to state the run-wide figure.

## The Indic ledger — the number that constrains the design

This is the most consequential correction in the whole document.

An earlier draft claimed **~285B** of real Indic supply: Sangraha 64B verified +
IndicCorp v2 21B + **~200B** projected legal/government scrape. Adversarial
review destroyed the 200B on two grounds:

1. **Magnitude.** Published Indian legal corpora are roughly an order of
   magnitude smaller. InLegalBERT's pretraining corpus is ~27GB (~7B tokens);
   Indian Kanoon holds ~30M judgments (~35–40B tokens at that document size).
2. **Language.** Indian judgments, RBI/SEBI circulars and Parliament records are
   *overwhelmingly written in English*. The draft was booking the same corpus
   twice — once as India-context English, once as native Indic.

Corrected supply:

```
Sangraha, human-verified tier                       64B
IndicCorp v2                                        21B
Indic-script gazette / state portal / Hindi judgments  ~10B
                                                   -----
real verified native supply                        ≈95B unique
```

Sangraha's 251B headline is ~65% machine-translated and transliterated. Only the
64B verified tier counts as native human text.

### From 95B to a bucket size

```
real native after repetition = 95B × 4 epochs (hard cap) = 0.38T
synthetic ceiling inside a scarce-language bucket = 50%
→ maximum bucket = 2 × 0.38T = 0.76T
```

So the bucket is **0.76T: 50% real, 50% quality-gated synthetic**. Going higher
breaks either the epoch cap or the synthetic ceiling.

**This is budget-independent.** When the budget fell from 20T to 14T, the bucket
stayed at 0.76T and its Stage A *share rose from 5% to 7%*. Scarcity, not
preference, sets it.

### Total Indic exposure

The bucket understates it:

```
native script                       0.76T
real romanized                      0.05T
~60% of the always-on floor         0.67T
Stage B top-decile Indic            0.26T
anneal native Indic (25% of 0.21T)  0.05T
                                    -----
                                    1.79T = 12.8% of the run
plus India-context English          1.62T
```

### Romanized text — a second correction

An earlier draft booked 0.32T of romanized text *outside* the synthetic line,
which is what kept the bucket under the 50% ceiling. Review found no
non-synthetic source at that scale: the only published romanized Indic corpus of
that size is Sangraha's transliteration tier, which is machine-produced
(IndicXlit) and sits inside the 65% synthetic padding. Corrected to **0.05T of
genuinely human tagged text**, with machine transliteration moved into the
synthetic line where it belongs.

## Stage B — 2.6T

| Content | % | Tokens |
|---|---|---|
| Math, verified solutions | 25% | 0.65T |
| Code, repo-level | 20% | 0.52T |
| India-domain English | 20% | 0.52T |
| Agentic traces | 15% | 0.39T |
| Top-decile Indic and romanized | 10% | 0.26T |
| Long verified reasoning | 10% | 0.26T |

**Stage B Indic was cut from 25% to 10%.** At 25% it would have drawn 0.65T from
a top decile of roughly 0.08T — about 8 epochs of that decile alone, before
counting repetition already spent in Stage A. That contradicted the epoch cap
stated two sections earlier. At 10% it is ~3 passes and must still be widened
past a strict decile, which is stated on the page.

## Stage D — anneal, 0.21T

| Content | Share |
|---|---|
| Math + code | 40% |
| Native Indic | 25% |
| Verified long reasoning | 25% |
| Gold agent trajectories | 10% |

Sized small deliberately. OLMo 2 moved grade-school math from **24.1% to 67.5%**
using a cooldown reserve of exactly this shape. The reserve must be set aside
before training starts; spending the best data early makes the lift unavailable.

## Post-training

| Stage | Volume | Published band | Status |
|---|---|---|---|
| SFT | ~1M pairs | 100K–1M | at ceiling |
| Preference | ~120K prompts × 4 ≈ **500K pairs** | 50–500K pairs | at ceiling |
| RLVR | ~100K tasks | 10–100K | at ceiling |

**Unit correction.** An earlier draft booked "400K prompts × 3–5 responses" and
claimed it sat at the top of the band. The band is denominated in *pairs*; 400K
prompts at 4 responses is ~2.4M pairwise comparisons, i.e. 3–5× above the
ceiling. Restated in pairs.

## Sources

- Sangraha tiers (64B verified / 24B unverified / 162B synthetic), Telugu
  16.3B→3.7B, Odia 12.5B→1.2B — AI4Bharat IndicLLMSuite, arXiv 2403.06350
- OLMo 2 anneal result — OLMo 2 report
- Code share effects — Aryabumi et al.; Petty et al. (TMLR 2025)
- Per-stage volume bands — Session 3 lecture widget `widget_1_data_lifecycle_map`
- V4 always-on channel, 8% / 66.9B / 10 scripts — Session 3 lecture widgets 15 and 22
