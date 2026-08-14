# How we got to 14T tokens

This is the number that took the longest to settle, and it changed three times.
The history matters more than the answer, so it is recorded here in full.

## The three versions

| Version | Number | Where it came from | Why it died |
|---|---|---|---|
| v1 | 15T · 375 tok/param | **Copied.** 15 of 64 peer submissions used 15T — the single most crowded answer in the cohort. Taken without derivation. | An adversarial review found the page claimed 15T was "the midpoint of the 10–30T band". It is not; 20T is. |
| v2 | 20T · 500 tok/param | Adversary's two arguments: true band midpoint, and ratio-matching the parity reference's 519 tok/param. | Two defects. (a) Ratio-matching confuses matching a *ratio* with matching *quality*. (b) 20T needs 4.2 epochs on the available unique pool, breaking the hard cap of 4 stated on the same page. |
| **v3** | **14T · 350 tok/param** | Derived from two independent constraints — see below. | Current. |

The v1 → v2 move fixed a false claim but introduced a worse one. The v2 → v3
move is the first time the number came out of arithmetic rather than out of
someone else's page.

## Constraint 1 — the floor: what parity actually costs

Training cost is about **6 FLOPs per parameter per token**: 2 for the forward
pass (one multiply, one add per parameter), 4 for the backward pass (gradients
with respect to both the input and the weight). So `C = 6ND`.

The parity reference is Gemma 3 27B trained on 14T tokens, which is published:

```
target      C = 6 × 27e9 × 14e12 = 2.27e24 FLOPs
ours at 14T C = 6 × 40e9 × 14e12 = 3.36e24 FLOPs   → 1.48× the target
```

Matching that compute **exactly** takes only **9.4T** at 40B:

```
9.4T = 2.27e24 / (6 × 40e9)
```

We carry 29% more parameters than the 27B reference, so we need *fewer* tokens
per parameter to reach the same total compute — not the same number. 9.4T is
therefore the floor. Below it, "as good as Gemma 4" is arithmetically
unavailable.

### Why this kills the Chinchilla row

Chinchilla (arXiv 2203.15556) found that for a **fixed compute budget**, the
loss-minimising split is about 20 tokens per parameter. At 40B that is 0.8T.

```
ours at 0.8T  C = 6 × 40e9 × 0.8e12 = 1.92e23 FLOPs
ratio to target = 2.27e24 / 1.92e23 = 11.8× LESS compute
```

Using the paper's own Approach-3 fitted loss law (their equation 10, with
`E = 1.69, A = 406.4, B = 410.7`):

```
L(N,D) = 1.69 + 406.4/N^0.34 + 410.7/D^0.28

target  27B / 14.0T → 1.8910
ours    40B /  0.8T → 1.9818   (+0.091 = worse)
ours    40B / 14.0T → 1.8766   (−0.014 = better)
```

Decomposing our 0.8T model's loss shows where the damage is:

| Term | Value at 40B / 0.8T |
|---|---|
| irreducible `E` | 1.690 |
| parameter term `A/N^0.34` | 0.101 — healthy, we have parameters |
| data term `B/D^0.28` | **0.191** — the damage |

The same data term at 14T falls to 0.086. A 40B model on 0.8T is a large model
that has barely read anything.

**The deeper point:** Chinchilla optimises loss *per unit of training compute*.
Our parameter count is fixed at 40B by the brief, so serving cost is already
locked — extra training tokens are quality at zero marginal serving cost. The
compute-optimal point is the correct answer to a question we are not asking.
That is why no deployed model sits at 20 tok/param: Gemma 3 27B is at 519,
Qwen3 32B ~1,125, Llama 3 8B ~1,875.

## Constraint 2 — the ceiling: unique data supply

Repetition is nearly free to about 4 epochs and near-worthless by 16
(Muennighoff et al.). We hold an **average of ≤3 passes, hard cap 4**.

So the budget cannot exceed `3 × (unique tokens available)`. Counting published
corpora that pass our own filters:

| Source | Unique tokens |
|---|---|
| DCLM-baseline, filtered English | 2.60T |
| FineWeb-Edu, education-filtered English | 1.30T |
| The Stack v2, permissive, deduplicated | 0.65T |
| Other multilingual, quality-gated | 0.60T |
| Math and STEM: OpenWebMath, FineMath, peS2o, arXiv | 0.20T |
| Books and long-form, licence-clear | 0.15T |
| Indic native, verified | 0.095T |
| India-context English: judgments, gazettes, Parliament, NCERT | 0.05T |
| **Subtotal** | **5.65T** |
| After ~15% overlap between the English sources | **≈4.8T** |

```
ceiling = 3 × 4.8T = 14.4T
```

## Where the two constraints meet

| Option | Tokens | Tok/param | Compute vs target | Epochs on 4.8T | Verdict |
|---|---|---|---|---|---|
| Chinchilla | 0.8T | 20 | 0.08× | 0.2 | 11.8× less compute; loss 1.982 vs 1.891 |
| Parity floor | 9.4T | 235 | 0.99× | 2.0 | Bare match, no headroom |
| **Selected** | **14T** | **350** | **1.48×** | **2.9** | **Above the floor, inside ≤3 epochs** |
| Peer-common | 16T | 400 | 1.69× | 3.3 | Breaks the ≤3 average |
| Aggressive | 20T | 500 | 2.12× | 4.2 | Breaks the hard cap of 4 |
| Ceiling | 30T | 750 | 3.17× | 6.2 | Needs ~10T unique; does not exist |

14T is the largest budget that clears the compute floor while respecting the
epoch discipline. Not a preference — the intersection of two independent bounds.

## The weakest link

The 4.8T figure is assembled from **published corpus sizes**, not measured after
running our own pipeline end to end. Sensitivity:

- real yield 6T → 18T becomes defensible
- real yield 3.5T → budget drops to ~10T

Measuring it therefore gates the run. This is the lecture's "ablate before you
scale" discipline applied to the budget itself rather than only to the mixture.

## What peers chose, for context

Extracted from 91 archived submissions plus 8 re-rendered SPAs; 64 stated a
budget.

- median **14T**, mean 13.4T
- mode **15T** (15 submissions)
- 80% inside the lecture's 10–30T band
- only 2 chose 20T; 3 chose 30T

Our 14T coincides with the cohort median. That is an artifact of the
derivation, not its cause — the derivation ran before this distribution was
computed, and the v1 15T that *was* copied from peers is the one that failed.

## Sources

- Chinchilla scaling law and fitted coefficients — arXiv 2203.15556, eq. 10
- Repetition value knee/cliff — Muennighoff et al., arXiv 2305.16264
- Effective public text stock ~300T — Villalobos et al., arXiv 2211.04325
- Gemma 3 27B: 14T tokens, distillation-pretrained — Gemma 3 technical report
- Corpus sizes — DCLM (arXiv 2406.11794), FineWeb (arXiv 2406.17557), The Stack v2
