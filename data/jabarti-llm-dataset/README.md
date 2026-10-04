---
language:
  - ar
  - en
license: cc-by-sa-4.0
task_categories:
  - text-generation
  - question-answering
tags:
  - arabic
  - egyptian-history
  - bilingual
  - llm-training
  - from-scratch
  - wikipedia
configs:
  - config_name: pretrain
    data_files:
      - split: train
        path: pretrain/train-*
      - split: eval
        path: pretrain/eval-*
      - split: phase1_train
        path: pretrain/phase1_train-*
      - split: phase1_eval
        path: pretrain/phase1_eval-*
      - split: phase2_train
        path: pretrain/phase2_train-*
      - split: phase2_eval
        path: pretrain/phase2_eval-*
  - config_name: finetune
    data_files:
      - split: train
        path: finetune/train-*
      - split: eval
        path: finetune/eval-*
---

# jabarti-llm-dataset

Cleaned, section-chunked training corpus for a **small bilingual LLM**
(Arabic + English), combining a curated Egyptian-history collection with
general Wikipedia coverage from
[CohereLabs/wikipedia-2023-11-embed-multilingual-v3](https://huggingface.co/datasets/CohereLabs/wikipedia-2023-11-embed-multilingual-v3).

Every `pretrain` record is a contiguous span of 120-1500 characters with the
article title and section headings removed. Provenance is in `ds_source`.

## Configs and Splits

| Config | Split | Rows | Training phase | Purpose |
|--------|-------|------|----------------|---------|
| `pretrain` | `train` | 7,053,893 | Pretrain | Bilingual Wikipedia corpus, AR+EN, shuffled |
| `pretrain` | `eval` | 114,259 | Pretrain | Held-out perplexity eval (leakage-safe by article) |
| `pretrain` | `phase1_train` | 175,476 | Legacy | Previous phase-1 split, kept for compatibility (see below) |
| `pretrain` | `phase1_eval` | 2,000 | Legacy | Previous phase-1 eval split |
| `pretrain` | `phase2_train` | 94,486 | Legacy | Previous phase-2 (Egypt-domain) split |
| `pretrain` | `phase2_eval` | 2,001 | Legacy | Previous phase-2 eval split |
| `finetune` | `train` | 57,488 | Finetune | SFT pairs, loss-masked to answer at train time |
| `finetune` | `eval` | 6,379 | Finetune | Held-out QA eval, AR+EN (filter via `language`) |

## Usage

```python
from datasets import load_dataset

REPO = "bakrianoo/jabarti-llm-dataset"

pretrain = load_dataset(REPO, "pretrain", split="train")  # or split="eval"
finetune = load_dataset(REPO, "finetune", split="train")  # or split="eval"

# Optional: keep only the curated Egyptian-history records
jabarti = pretrain.filter(lambda r: r["ds_source"] == "jabarti")
```

## Corpus Composition (`pretrain`)

| `ds_source` | language | train records | share of characters |
|-------------|----------|--------------:|--------------------:|
| `cohere-wiki` | en | 3,515,381 | 50.3% |
| `cohere-wiki` | ar | 3,225,427 | 39.5% |
| `jabarti` | en | 220,445 | 7.2% |
| `jabarti` | ar | 92,640 | 3.0% |

`jabarti` records carry the full curation metadata below; `cohere-wiki`
records are null on those fields and populate only `text`, `title`, `url`,
`language`, `record_id`, `page_id`, `ds_source` and `sample_weight`.

Because `jabarti` is ~4% of records, use `ds_source` and `egypt_relevance`
with sampling weights if you want to emphasise the Egyptian-history domain.

## Eval Leakage Safety

Eval is held out by **whole article** across both sources — an article is
entirely in `train` or entirely in `eval`, never split across the boundary.
Verified: **0 of 21,989 eval articles appear in train**. Articles are matched
on the normalised Wikipedia page, so the same article under different internal
ids cannot straddle the split.

## Article Schema (`pretrain` config)

- `text` — record body; article title and section headings removed
- `ds_source` — `jabarti` | `cohere-wiki`
- `record_id` — stable per-record id
- `page_id` — groups records from the same article (used for the eval holdout)
- `sample_weight` — `1/sqrt(records from this article)`; corrects the bias
  toward long articles. Recomputed for this chunking.
- `ortho_stripped` — Arabic written without hamza forms (see caveat below)
- `language` — `ar` or `en`
- `chunk_index`, `chunk_total` — position within the source article
- `chunk_strategy` — `section` | `paragraph` | `whole`
- `egypt_relevance` — `core` | `related` | `incidental` | `none`
- `content_quality` — `high` | `medium` | `low` | `unusable`
- `is_person` — whether the article is a biography
- `topic_domain` — `history` | `biography` | `sports` | `arts` | `science` | `politics` | `geography` | `other`
- `importance` — `high` | `medium` | `low` (based on Wikipedia pageviews)
- `ortho_convention` — `native` | `stripped` | `mixed` (AR only; EN rows are empty string)
- `source_type` — `native_wiki` | `llm_generated` (`jabarti` only)

## Legacy Phase Splits (`pretrain`)

`phase1_train`, `phase1_eval`, `phase2_train` and `phase2_eval` re-present the
previous phase-organized pretrain splits, unchanged in content, from revision
[`2cad63a`](https://huggingface.co/datasets/bakrianoo/jabarti-llm-dataset/tree/2cad63ab88e5fc224397acf599edfd786bc2bb94),
so existing pipelines keep working on the latest revision:

```python
p1_train = load_dataset(REPO, "pretrain", split="phase1_train")
```

They share the current schema; differences from `train` / `eval`:

- `text` is the original chunk **with** title prefix and section headings.
- `sample_weight` keeps the original value, `1/sqrt(chunk_total)`.
- `ds_source` is `jabarti`; `section_index = 0`, `section_total = 1` (each
  record is a whole original chunk).
- `record_id` is `<split>.p<phase>.<lang>_<article#>_c<chunk>`, distinct from
  the ids in `train` / `eval`.
- `ortho_stripped` is recomputed per record (Arabic, >=50 Arabic letters,
  <12 hamza forms per 1k chars), which approximates the current flag.
- The old `section_title` column is dropped. The heading is already present in
  `text` for every record that had one.
- `phase2_train` includes phase-1 articles by design, and the legacy
  splits overlap with `train` / `eval`. Do not mix them with the new splits.

## Known Caveats

- **Arabic orthography.** The `llm_generated` portion of `jabarti` was written
  without hamza forms (~5 per 1k chars against ~28 in native Arabic Wikipedia)
  and essentially without diacritics. Those records are flagged
  `ortho_stripped = true`; filter or down-weight them if standard orthography
  matters for your model.
- **Overlapping coverage.** `jabarti` and `cohere-wiki` both draw on Wikipedia,
  so some articles appear under both sources with different chunk boundaries.
  Exact-duplicate text is removed; re-chunked duplicates are retained
  deliberately, acting as upsampling of that content.

## QA Schema (`finetune` config)

- `question`, `answer` — bilingual Q&A pair
- `language` — `ar` or `en`
- `type` — `factual` | `temporal` | `analytical` | `synthesis` | `unanswerable`
- `difficulty` — `easy` | `medium` | `hard`
- `source_article_id` — links back to the phase_2 source article
- `article_title` — title of the source article

