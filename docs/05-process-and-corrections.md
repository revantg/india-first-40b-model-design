# Process, and every correction made along the way

This file exists so the assignment can be judged on its reasoning rather than
its presentation. It records how the work was actually produced, including the
parts that were wrong.

## How the work was produced

1. **Pulled the session material.** Lecture notes, all 21 embedded widgets, the
   assignment text, and the linked transcript were fetched from the course
   platform via a cookie-authenticated API. The widgets matter because they carry
   the hard numbers (Sangraha tiers, OLMo 2 anneal lift, FineWeb dedup result,
   V4 always-on channel).
2. **Read peer submissions.** 92 links from the course submission gallery; 91
   archived, 8 JS-rendered separately. Used for calibration and, initially and
   wrongly, for copying.
3. **Built a slide deck.** First attempt was a 14-slide presentation.
4. **Rebuilt as a document.** The deck hid the answers behind visual polish. On
   direct feedback it was rebuilt as one scrolling table-dense page, working
   backwards from the four assignment bullets.
5. **Ran adversarial review.** Two rounds against the numbers, plus vision review
   of rendered screenshots for layout defects.
6. **Derived the contested numbers from scratch** after the reviews showed the
   originals were copied or internally inconsistent.

## Where the work started as copying, and stopped

Being specific, because this is the part most worth disclosing.

**Copied and later replaced:**

- **15T token budget.** Taken from peers. 15 of 64 submissions with a stated
  budget used 15T — the most crowded answer in the cohort. Replaced with 14T
  derived from a compute floor and a unique-supply ceiling. See
  `01-token-budget.md`.
- **Dense-vs-MoE comparison table.** Copied structurally from a peer's page,
  which opened with exactly that table. **Deleted entirely** — the assignment
  gives 40B as a premise, so the comparison answers a question nobody asked, and
  it changed nothing downstream.

**Adopted deliberately, with attribution in this file:**

- The `bucket / % / ~tokens / why` table format, and the idea of an explicit
  real-versus-synthetic Indic ledger, are patterns seen in peer submissions and
  judged genuinely better than what the deck had. Reused as *form*; every number
  inside is our own and several contradict the sources they came from.

**Derived independently:**

- 14T budget, 4.8T unique-supply estimate, 95B real Indic supply, the 0.76T
  Indic bucket ceiling, per-language fertility targets, the 192K vocabulary
  allocation, and every evaluation bar.

## Corrections log

### Round 1 — internal self-review

| Defect | Fix |
|---|---|
| Always-on floor listed as an extra row, making stages sum to 16.2T instead of 15.0T | Restated as cross-cutting, already counted inside bucket percentages |
| "Indic ≤1.4× English" asserted while Dravidian target implied 1.54× | Stated the Dravidian exception explicitly with the arithmetic |
| ~200B legal/government scrape presented as available corpus | Relabelled projected; later cut entirely |

### Round 2 — adversarial review of the deck

| Defect | Fix |
|---|---|
| Zero diagrams across 14 slides despite six obvious opportunities | Added 8 inline SVGs |
| "66.9B tokens" attached to an 8% channel that would be 89B of 1.118T | Restated as the Indic *slice* of that channel |
| Numbers labelled "Committed" while the same slide called them untested candidates | Relabelled "Selected"; removed swagger framing |
| SFT ~2.5M and RL ~150K above the published bands | Cut to ~1M and ~100K |
| Dravidian ≤2.00 presented as fact while beating published SOTA | Restated as a gate, then re-derived per language |
| Build notes ("Nike / Bebas Neue · surge", "last-10 cherry-pick") visible on the final slide | Removed |

### Round 3 — adversarial review of the numbers

| Defect | Fix |
|---|---|
| 15T claimed as "midpoint of the 10–30T band" | False; 20T is. Justification deleted entirely — a midpoint of someone else's range is not an argument |
| Distillation used to reject the 8T option but absent from the winning recipe | Adopted explicitly in Stages A and B |
| ~285B real Indic supply, ~70% resting on a projected scrape | Cut to ~95B; the legal corpus is English and was being double-counted |
| SWE-bench ≥40% | Raised to ≥65% |
| BFCL unversioned | Pinned to v3 ≥78% |
| Grouped fertility rows hiding Malayalam 2.30 behind Tamil 2.12 | Split into 13 per-language targets |
| "3–8× generic tax" wrong at the low end | Corrected to 1.3–7.8× |
| Chart tails drawn in the wrong language rows | Redrawn per language at measured values |
| 0.32T romanized booked as non-synthetic with no source | Cut to 0.05T; machine transliteration moved into synthetic |
| Stage B Indic 25% requiring ~8 epochs of a thin decile | Cut to 10% |
| Vocab blocks summing to 200,704 against a claimed 128-aligned total | Corrected to 94+76+12+10 = 192K = 196,608 |
| "12 Indic scripts" | 10 scripts across 12 scheduled languages |
| Preference volume booked in prompts, compared against a band in pairs | Restated as ~500K pairs |
| "Code capped at 18%" while the run was 19.5% | Restated as 17.6% run-wide |

### Round 4 — self-review after the adversary

The adversary's own replacement for the budget did not survive checking:

- It proposed 20T by ratio-matching the reference's 519 tok/param. That confuses
  matching a **ratio** with matching **quality**. At 40B, ratio-matching
  overshoots parity compute by 2.19×, because we already carry 29% more
  parameters than the 27B reference.
- 20T also needs 4.2 average epochs on the 4.8T unique pool, breaking the hard
  cap of 4 stated on the same page.

Both were accepted uncritically at first. Corrected to 14T.

### Round 5 — vision review of rendered screenshots

Layout defects found by inspecting actual renders: hero glyphs clipping the left
rule, the 0.8T bar drawn 2.5× too wide against a "relative width" caption, an 8T
label overflowing its bar, a truncated sentence in the mix SVG, unreadable dark
red on dark grey in the vocabulary cards, unequal card widths, and the brand
wordmark colliding with the progress bar. All fixed.

## Known weaknesses, unresolved

1. **The 4.8T unique-supply figure is assembled from published corpus sizes, not
   measured.** It gates the entire budget. At 6T, 18T becomes defensible; at
   3.5T, the budget drops to ~10T. This is the single largest risk.
2. **The ~10B Indic-script gazette/state-portal estimate is a collection target**,
   not a catalogued corpus.
3. **Nothing was trained.** Every figure is a planning estimate. The report says
   so in the utility bar, in the trade-offs table and in the footer.
4. **Dravidian equity gap of 1.42–1.69×** is stated but not solved.

## Tooling

Written with an AI coding agent. The agent fetched course material, archived and
parsed peer submissions, ran the arithmetic (all budget, epoch, compute and
allocation figures were computed rather than estimated by hand), spawned the
adversarial reviewers, rendered and inspected screenshots, and deployed the site.
Numeric claims in the report were recomputed at least once by a separate
adversarial pass; the corrections log above is the audit trail.
