# Work Plan

This file splits [thesis_plan.md](thesis_plan.md) into work packages (WP) with concrete tasks, outputs and "done when" checks. Section numbers (§) refer to `thesis_plan.md`.

Estimates are in weeks of part-time work. The total is about 30 weeks, and some packages can overlap (see the dependency graph). Set real dates once the submission deadline is known.

---

## Dependency graph

```
WP0 Setup & scope ──► WP1 Collector ──► WP2 Pilot corpus ──► WP3 Pilot annotation
                                              │                     │
                                              ▼                     ▼
                                     WP4 Full collection     WP5 Gold set (+ 2nd annotator)
                                              │                     │
                                              ▼                     ▼
                                     WP6 Pre-processing ──► WP7 Models & evaluation
                                                                    │
WP9 External data (CBOS, events) ──────────────────────────────────┤
                                                                    ▼
                                                     WP8 Full-corpus labelling
                                                                    │
                                                                    ▼
                                                     WP10 Analysis (RQ1–RQ4)
                                                                    │
WP11 Writing (runs alongside everything; most of it at the end) ◄───┘
```

---

## Proposed folder layout

```
sentiment/
├── thesis_plan.md, plan.md
├── config/            channels.yaml, keywords.yaml, events.yaml
├── src/
│   ├── collect/       video selection + comment collector (YouTube Data API)
│   ├── preprocess/    cleaning, dedup, language id, pseudonymisation, topic filter
│   ├── annotate/      sampling for annotation, Label Studio import/export, kappa
│   ├── models/        baselines, HerBERT fine-tuning, LLM labelling, evaluation
│   └── analysis/      time series, change points, event alignment, polls, channels
├── data/              raw/, interim/, processed/, annotations/, external/   (git-ignored)
├── notebooks/         exploration only
├── legacy/            current scripts, kept for reference / baselines
└── thesis/            LaTeX or docs, figures/
```

Use git from the start (add `data/` and API keys to `.gitignore`).

---

## WP0: Setup and scope (week 1–2)

**Goal:** fixed scope and a working environment.

- [ ] Meet the supervisor: confirm RQs, which of RQ4/RQ5/emotions/distillation are in scope, and the deadline (§14)
- [ ] Ask the faculty about ethics committee approval (§9)
- [ ] Read the YouTube API Terms of Service on storing data and refresh rules; write down what that means for the project (§9)
- [ ] `git init`, create the folder layout, `requirements.txt` / virtualenv, `.gitignore`
- [ ] Move the current scripts to `legacy/` (§13)
- [ ] Start the literature list (Zotero or similar) for chapters 2–3

**Outputs:** scope notes from the supervisor meeting, repo skeleton.
**Done when:** the scope is agreed and the repo runs `pip install -r requirements.txt` cleanly.

## WP1: Channel/keyword lists and API collector (week 2–4)

**Goal:** a reproducible collector with exact timestamps (§4).

- [ ] Write `config/channels.yaml`: channel IDs for TVN24, TVP Info, Republika, wPolsce24 (+ Polsat News, RMF/Onet?), plus the date range each channel actually covers
- [ ] Write `config/keywords.yaml` (Ukraina, Ukraińcy, uchodźcy, Zełenski, Wołyń, zboże, 800+, Rosja, Putin, ...). Include inflected forms or match on stems
- [ ] Video selection: rewrite `youtube_top_videos_by_month.py` so it lists **all** channel uploads (via the uploads playlist, `playlistItems.list`, 1 unit/call, which is cheaper than `search.list` at 100) and filters titles/descriptions with the keyword list
- [ ] Comment collector: `commentThreads.list` + `comments.list` for replies, saving exact `publishedAt` (replaces `yt_scrapper.py` and `youtube_channel_top_comments_scraper.py`)
- [ ] Output schema: comment_id, video_id, channel, video_published_at, comment_published_at, text, like_count, is_reply, parent_id, author_hash, collected_at
- [ ] Hash author IDs at collection time (salted SHA-256, salt kept outside the repo) so raw author names are never stored
- [ ] Make it resumable: save progress per video, retry on quota errors, log quota use
- [ ] Store raw data as JSONL or Parquet, one file per channel

**Outputs:** `src/collect/`, config files.
**Done when:** a test run on one channel and one month gives a file that matches the schema and has exact timestamps.

## WP2: Pilot corpus (week 4–5)

- [ ] Collect 2 channels × 3 months (e.g. TVN24 + Republika; months from 2022, 2023 and 2025)
- [ ] Basic statistics: videos, comments, comments per video, share of replies, share of Polish comments
- [ ] Look at quota use and extrapolate to the full corpus

**Done when:** the pilot corpus exists and there is a size/quota estimate for the full collection.

## WP3: Pilot annotation and guidelines (week 5–7)

**Goal:** a tested annotation scheme (§3, §6.1).

- [ ] Set up Label Studio (or Doccano) with the scheme: targets T1–T5 (multi-label), stance per target, optional emotion and topic tag
- [ ] Draw ~500 comments from the pilot, stratified by channel and month
- [ ] Annotate them yourself; keep a log of hard cases
- [ ] Write guidelines v1 with examples for each target and the boundary rules (T2 vs T4, plain "Ukraińcy" → T3, T1 by period)
- [ ] Decide using the pilot: merge T2/T4? Keep emotions? Keep topic tags? How to handle sarcasm?
- [ ] Check target frequencies. If a target is very rare, plan to oversample it in the gold set

**Outputs:** `annotation_guidelines.md` (later an appendix), 500 pilot labels, decision log.
**Done when:** the scheme is frozen as v1.

## WP4: Full data collection (week 6–9, can run alongside WP3)

- [ ] Run the collector for all channels across 2022-02 → present (spread over several days for quota)
- [ ] Check coverage: comments per channel per month; mark gaps (channel didn't exist, comments turned off, etc.)
- [ ] Write a data card: date collected, counts, known gaps, limitations (deleted comments, selection bias)

**Done when:** the raw corpus is complete and the coverage table is in `data/` and in the thesis notes.

## WP5: Gold set and inter-annotator agreement (week 8–12)

- [ ] Draw 3,000–5,000 comments, stratified by channel × year × (predicted) target. Use keyword hits to oversample rare targets
- [ ] Annotate the gold set (the biggest single time cost; plan ~1–2 hours per 100 comments)
- [ ] Find a second annotator; brief them with the guidelines; they label 300–500 comments
- [ ] Compute Cohen's kappa per target (presence) and per stance; resolve disagreements; update the guidelines to v2
- [ ] Split into train / dev / test, stratified, with the test set frozen and not touched during development
- [ ] Consider a time-based split too (train on early years, test on later) to check robustness over time

**Outputs:** gold set, kappa table, final guidelines.
**Done when:** kappa is reported and the splits are frozen.

## WP6: Pre-processing pipeline (week 9–11, alongside WP5)

(§5)

- [ ] Dedup and spam filter (identical texts, link-only, very short)
- [ ] Language ID (fastText `lid.176` or `lingua`); decide what to do with UK/RU comments (drop, or count separately)
- [ ] Normalise text: URLs → token, @mentions → token, shorten repeated characters, keep emojis
- [ ] Topic filter: keyword match + BERTopic (or a small classifier trained on the pilot) to keep Ukraine/Russia-related comments
- [ ] Log how many comments each step removes (for the thesis "data" chapter)

**Outputs:** `src/preprocess/`, `data/processed/corpus.parquet`, a funnel table (raw → final).

## WP7: Models and evaluation (week 11–17)

(§6.2). Every model is scored on the same frozen test set.

- [ ] **Baseline 1:** `Voicelab/herbert-base-cased-sentiment` (overall sentiment, no target), reusing `legacy/polish_sentiment.py`
- [ ] **Baseline 2:** lexicon-based; optionally translation + VADER from `legacy/sentiment_translation.py`
- [ ] **Main model:** HerBERT fine-tuning, trying both
  - (a) a two-stage pipeline: target detection (multi-label) → stance per target
  - (b) a multi-task model with one head per target (none/pos/neg/neu)
- [ ] Tune hyperparameters on dev; try base vs large; run several seeds and report mean ± std
- [ ] **LLM:** zero-shot and few-shot prompts (Bielik + one GPT-class model); fixed prompt templates kept in the repo; log cost and runtime
- [ ] **Optional:** distillation (LLM labels → HerBERT); emotion model (PolEmo 2.0 and/or own labels)
- [ ] Evaluation: macro-F1 per target and stance, confusion matrices, a results table across all models
- [ ] Error analysis: sample ~100 errors from the best model; categorise them (sarcasm, implicit target, T2/T4 confusion, ...)

**Outputs:** `src/models/`, results tables, the choice of the best model (answers RQ5).
**Done when:** the comparison table is complete and the best model is chosen.

## WP8: Label the full corpus (week 17–18)

- [ ] Run the best model on the whole processed corpus; save predictions + confidence
- [ ] Spot-check: manually review ~200 random predictions across years and channels to check for drift over time

**Output:** `data/processed/predictions.parquet`.

## WP9: External data (any time from week 3 on, alongside other work)

- [ ] Collect CBOS reports 2022–2026: attitudes toward Ukrainians, support for helping Ukraine, accepting refugees. Also other polls (e.g. Opinia24, IBRiS) if CBOS has gaps. Turn them into a monthly table in `data/external/polls.csv`
- [ ] Build `config/events.yaml`: date, short name, category, source (§7 list + events through 2026)
- [ ] Note the government/president periods for interpreting T1 (PiS → Tusk Dec 2023; Duda → Nawrocki Aug 2025)

## WP10: Analysis (week 18–23)

(§7)

- [ ] **RQ1:** monthly (and weekly) share of pos/neg stance per target and channel, normalised by volume; plots with confidence intervals
- [ ] **RQ2:** change point detection with `ruptures` (PELT, binary segmentation); match change points to events; correlate T3/T4 with polls, with cross-correlation lags and Granger tests (check stationarity first)
- [ ] **RQ3:** compare channels in level and timing; TVP before vs after Dec 2023 (difference-in-differences or interrupted time series)
- [ ] **RQ4 (optional):** shift of negativity from T2 to T3 over time; topic tags / BERTopic inside T3 negativity
- [ ] **Optional:** comments that are anti-Ukrainian and pro-Russian at the same time; account-level patterns (by hashed author)
- [ ] Robustness: repeat the key results with likes as weights, without replies, and with only high-confidence predictions

**Outputs:** `src/analysis/`, figures in `thesis/figures/`, results tables.

## WP11: Writing (ongoing; focus week 20–30)

(§11). Write each chapter while the related work is fresh:

| Chapter | Draft during | Main input |
|---------|--------------|------------|
| 2 Background, 3 Related work | WP0–WP4 | literature list |
| 4 Data | WP4 + WP6 | data card, funnel table, ethics notes |
| 5 Annotation | WP5 | guidelines, kappa table |
| 6 Methods | WP7 | model setup |
| 7 Results I (models) | WP7 | comparison table, error analysis |
| 8 Results II (analysis) | WP10 | figures, tests |
| 9 Discussion & limitations | WP10 | §8 list |
| 1 Introduction, 10 Conclusions | last | everything |

- [ ] Send the supervisor a draft of each chapter as it is finished, not all at the end
- [ ] Leave 3–4 weeks at the end for revisions, formatting and appendices (guidelines, prompts, extra tables)

---

## Summary timeline

| Weeks | Work packages | Milestone |
|-------|---------------|-----------|
| 1–2 | WP0 | Scope agreed with the supervisor |
| 2–5 | WP1, WP2 | Collector works; pilot corpus ready |
| 5–9 | WP3, WP4 | Annotation scheme frozen; full corpus collected |
| 8–12 | WP5, WP6 | Gold set + kappa; clean corpus |
| 11–18 | WP7, WP8 | Best model chosen; corpus labelled |
| 18–23 | WP10 (WP9 ready) | Analysis results |
| 20–30 | WP11 | Full draft → final thesis |

## Main risks

| Risk | Mitigation |
|------|------------|
| Annotation takes longer than planned | Start the gold set early; use LLM pre-labels that you correct (record this in the methods chapter) |
| No second annotator | Ask early (WP0); 300 comments is a small, clear request |
| API quota or comments turned off on old videos | Collect early; spread runs over days; ask for a quota increase if needed |
| T2/T4 agreement too low | Merge them (already planned in §3) |
| Rare targets → weak model | Oversample with keywords in WP5; report per-target support |
| Scope too large | Optional items (RQ4, emotions, distillation, troll analysis) are the first to drop |
