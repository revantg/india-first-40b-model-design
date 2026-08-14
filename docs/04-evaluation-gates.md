# How we set the evaluation gates

## The rule

A gate that a smaller published model already clears is not a gate. Every bar is
pinned either to a live competitor or to a versioned harness.

An earlier draft failed this rule in three places. All three were caught in
adversarial review.

## Correction 1 — SWE-bench Verified: 40% → 65%

The draft set the coding gate at **≥40%** for a 40B model whose brief says
"really good in coding". Published sub-30B models already beat that
substantially:

| Model | Params | SWE-bench Verified |
|---|---|---|
| Devstral Small 2 | 24B | 68.0% |
| Qwen3-Coder | — | 69.6% |
| Qwen3-Coder-Next | — | 70.6% |

A 40B shipping at 40% would be ~28 points behind a 24B open-weights model on the
benchmark nominated as its own coding gate. The gate could not discriminate
between success and failure of the stated objective.

Set to **≥65%**, with the scaffold and harness pinned, i.e. parity with the best
published sub-30B coding models.

## Correction 2 — BFCL: unversioned → v3 ≥ 78%

The draft said "BFCL ≥ 70%" with no version. BFCL versions differ materially in
difficulty, so an unversioned number is unmeasurable. Open SOTA on v3 is ~77.8%.

Set to **v3 ≥ 78%**, with v4 reported separately because it is materially harder.

## Correction 3 — the parity referent

The draft said "within 2–3 points of the same-size dense Gemma". There is no
such model: Gemma 4's largest dense model is 31B. "Same-size" had no referent.

Set to **Gemma 4 31B dense**, and stated plainly that we are 40B against 31B, so
parity is the floor rather than the goal.

## Final gate table

| Objective | Suite → bar |
|---|---|
| General parity | MMLU-Pro, GPQA-Diamond, IFEval, Arena-Hard → within 2–3 points of Gemma 4 31B dense, identical prompts/tools/token budgets |
| Coding | SWE-bench Verified ≥65% (harness pinned) + LiveCodeBench (date-gated) + RepoBench. HumanEval is a smoke test only |
| Agentic | BFCL v3 ≥78%, τ-bench, terminal-bench, OSWorld subset → success **and** steps and cost; plus private held-out environments |
| Indic | MILU (beat Gemma 4 31B), IndicGenBench, IN22 chrF++, romanization-robustness gap <5%, Hinglish QA — per language, never averaged |
| India-first | Polity/schemes/GST factuality (UPSC-style), default-perspective probes, refusal-balance parity across religion/caste/region, quarterly Indian-rater eval |
| Safety | Scored in-language — published Indic work finds only ~13% cross-language agreement on safety decisions, so an English-only pass is a false pass |
| Continuous | Per-domain and per-language loss dashboards, tokenizer-fertility regression in CI, decontamination audit before any reported number |

## Why per-language reporting is non-negotiable

The brief asks for four capabilities at once. An averaged Indic score lets a
strong Hindi result hide a broken Odia. Every Indic number is reported per
language, and no language may regress materially.

## The contamination firewall

Three tiers, taken from the lecture's own structure:

| Tier | May it train? | Rule |
|---|---|---|
| Training pools (4 stages, SFT, preference, RL) | Yes | Decontaminated against every test set — exact, fuzzy, semantic **and cross-lingual** |
| Benchmark *train* splits | Yes | Allowed inside the always-on floor to teach answer format; matching test split never enters a batch |
| Golden proxy (test/validation) | **Never** | Immutable, stored where training jobs have no read access; used only to steer the selector and score the model |

Cross-lingual matching matters specifically here: an English benchmark item that
appears in Hindi is still contamination. A leaked item does not merely inflate
one score — it permanently invalidates that item as a test.

## India-first evaluation is sealed

~3,000 expert-authored private tasks across the scheduled languages, rotating
and independently administered.

Accepted cost: not externally reproducible. Accepted reason: a public
India-specific benchmark would be contaminated within one release cycle.

## Sources

- Published sub-30B SWE-bench Verified results — model cards for Devstral Small 2, Qwen3-Coder
- BFCL v3 open SOTA — Berkeley Function-Calling Leaderboard
- ~13% cross-language safety agreement — published IndicSafe-style evaluation
- Three-tier firewall structure — Session 3 lecture, widget 16
