# ERA V5 · Session 3 — a 40B India-first model: data, cleaning, evaluation, tokenizer

**Live report:** https://india-first-40b.surge.sh/

A design brief for a 40B India-first model: what data to collect and why at each
training stage, how to clean it for those objectives, how to test it so a strong
English score cannot hide a weak Odia, and what per-language fertility implies
for vocabulary size.

Nothing here was trained. Every figure is a planning estimate, and the report
says so on its face.

---

## The assignment, verbatim

> # The assignment
>
> Assume that you have to train a 40B model, which is as good as Gemma 4, is really good in coding, agentic work, Indic languages and is India first (views the world from Indian perspective). Do your research with your agents and decide:
> - how your data would look like, what would you collect and why (for pre-training, post-training, RL/alignment)
> - how would you clean you data for the objectives defined
> - how would you test your models against the objective
> - what fertility would you target for the different languages (which languages would you focus on), coding, science, math and agentic tasks, and based on these number, what would be your tokenizer size (token vocab size)
>
> Once you're sure of your number, work with your agent to write a report, upload it on netlify and share back with us. Our evaluation would be based on how much have you thought through. Longer submissions will result is lower scores.

Also in [`ASSIGNMENT.md`](ASSIGNMENT.md), with a map from each bullet to the
section that answers it.

---

## The committed numbers

| Decision | Value | Derived from |
|---|---|---|
| Token budget | **14T**, 350 tok/param | Compute floor (1.48× the parity target) ∩ supply ceiling (2.9 epochs on 4.8T unique) |
| Partition | 12.67T scored / 1.12T floor / 210B anneal | 8% always-on floor from the V4 precedent |
| Stage A | 10.8T across 9 buckets | See `docs/02-data-mixture.md` |
| Native Indic | **0.76T**, 7% of Stage A | 95B verified supply × 4-epoch cap, doubled by a 50% synthetic ceiling |
| Code | 17.6% of the run | Bounded above by published morphology/recall damage |
| Post-training | ~1M SFT pairs / ~500K preference pairs / ~100K RL tasks | Top of the published per-stage bands |
| Fertility | 13 per-language targets, Malayalam ≤2.20 binding | Measured against a published 200K Indic tokenizer |
| Vocabulary | **196,608** = 94K + 76K + 12K + 10K | Fertility targets, then a rejection chain |

---

## How this was actually made

Read this section before judging the rest.

**The report was written with an AI coding agent**, across a single long session.
The agent fetched the course material, archived and parsed 92 peer submissions,
computed every arithmetic figure, ran adversarial review passes against its own
numbers, inspected rendered screenshots for layout defects, and deployed the
site.

### Parts that began as copying

- **The first token budget (15T) was copied from peers.** 15 of 64 submissions
  with a stated budget used 15T — the most crowded answer in the cohort — and it
  was adopted without derivation. It was later replaced with 14T derived from
  first principles, after review showed the page's stated justification
  ("midpoint of the 10–30T band") was simply false.
- **A dense-vs-MoE comparison table was copied structurally from a peer's page**
  and later deleted outright. The assignment gives 40B as a premise, so the
  comparison answered a question nobody asked and changed nothing downstream.
- **The table format** (`bucket / % / ~tokens / why`) and the idea of an explicit
  real-versus-synthetic Indic ledger are patterns taken from peer submissions,
  judged better than the original slide deck, and reused deliberately. The
  numbers inside are ours, and several contradict the submissions the format came
  from.

### Parts derived independently

The 14T budget, the 4.8T unique-supply estimate, the 95B real Indic supply, the
0.76T Indic bucket ceiling, all 13 fertility targets, the 192K vocabulary
allocation, and every evaluation bar.

### It was wrong repeatedly

Five review rounds produced roughly 25 corrections, including several where the
adversarial reviewer's own replacement was itself wrong and had to be rejected.
The full log is in
[`docs/05-process-and-corrections.md`](docs/05-process-and-corrections.md).
Representative examples:

- Stages summed to 16.2T instead of 15.0T because the always-on floor was
  double-counted as an extra row.
- A claimed ~200B Indic legal/government corpus was roughly an order of magnitude
  too large **and in the wrong language** — Indian judgments are overwhelmingly
  English, so the same text was being counted twice.
- SWE-bench Verified was gated at ≥40% when a published 24B model already scores
  68%.
- Vocabulary blocks summed to 200,704 against a total claimed to be 128-aligned.
- The adversary's proposed 20T budget confused matching a tokens-per-parameter
  *ratio* with matching *quality*, and needed 4.2 epochs against a stated hard cap
  of 4.

---

## Repository layout

```
session-3-assignment/     the deployed report (index.html + style.css)
docs/                     how every number was derived
  01-token-budget.md      14T, and the two budgets that came before it
  02-data-mixture.md      stages, buckets, the Indic supply ledger
  03-fertility-and-vocab.md  per-language targets → 196,608
  04-evaluation-gates.md  why each bar is set where it is
  05-process-and-corrections.md  full corrections log and known weaknesses
transcript/               complete session transcript (Git LFS)
  *.jsonl                 raw session log
  *.html                  rendered transcript
ASSIGNMENT.md             the brief, verbatim, plus a map to the answers
```

### Not in this repository

Course lecture notes, the lecture's embedded widgets, the session transcript from
the course platform, and archived peer submissions are **deliberately excluded**.
They are the instructor's and other students' material. Where a lecture figure is
used, it is cited in the docs rather than reproduced.

---

## Known weaknesses

1. **The 4.8T unique-supply figure gates the entire budget** and is assembled from
   published corpus sizes rather than measured through our own pipeline. At 6T,
   18T becomes defensible; at 3.5T, the budget drops to ~10T. This is the largest
   single risk in the plan.
2. The ~10B Indic-script gazette and state-portal estimate is a collection target,
   not a catalogued corpus.
3. The Dravidian cost-equity gap (1.42–1.69× English, against a 1.4× principle
   that holds for the other eight languages) is stated but not solved.
4. Nothing was trained. Mixtures, thresholds and vocabulary would be frozen only
   after 1B and 3B equal-compute ablations rank them.

---

## Design

The report reuses the Nike-derived design system (`npx getdesign@latest add nike`)
built for the Session 2 tokenizer assignment, whose report is at
https://india-bpe-tokenizer.surge.sh/ — Bebas Neue display, Inter body,
monochrome palette, tables over decoration.
