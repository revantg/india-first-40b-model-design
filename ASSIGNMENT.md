# The assignment

Verbatim from the ERA V5 course platform, Session 3 (Data Collection and
Sourcing). Quoted here so this repository can be judged against the actual
brief rather than a paraphrase.

> # The assignment
>
> Assume that you have to train a 40B model, which is as good as Gemma 4, is really good in coding, agentic work, Indic languages and is India first (views the world from Indian perspective). Do your research with your agents and decide:
> - how your data would look like, what would you collect and why (for pre-training, post-training, RL/alignment)
> - how would you clean you data for the objectives defined
> - how would you test your models against the objective
> - what fertility would you target for the different languages (which languages would you focus on), coding, science, math and agentic tasks, and based on these number, what would be your tokenizer size (token vocab size)
>
> Once you're sure of your number, work with your agent to write a report, upload it on netlify and share back with us. Our evaluation would be based on how much have you thought through. Longer submissions will result is lower scores.

## How the submission maps to the brief

| Brief asks | Where it is answered |
|---|---|
| Data: what to collect and why, for pre-training / post-training / RL-alignment | Section A1 — four pre-training stages with per-bucket token counts, the Indic supply ledger, and a post-training table covering SFT, preference and RLVR |
| Cleaning for the objectives defined | Section A3 — a separate pipeline per bucket, plus two India-specific rules generic pipelines get wrong |
| Testing against the objective | Section A4 — a numeric bar per objective pinned to a live competitor or versioned harness, plus the three-tier contamination firewall |
| Fertility per language / coding / science / math / agentic, and the vocab size it implies | Section A5 — 13 per-language targets against a measured 200K reference, non-word targets for code/math/agents, then the vocabulary rejection chain and slot allocation |

Two constraints in the brief shaped the structure directly:

- **"Assume ... a 40B model"** — parameter count is a premise, not a decision. An
  early draft included a dense-vs-MoE comparison; it was deleted because the
  assignment settles model size in its first sentence and the choice changed
  nothing downstream.
- **"Longer submissions will result is lower scores"** — the report is one page
  of tables. Prose is used only where a number needs a reason.

The one thing not asked for but included: a **token budget**. It is unavoidable,
because every figure in the data mixture is a share of it. Its derivation is in
`docs/01-token-budget.md`.
