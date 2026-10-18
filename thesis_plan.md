# Master's Thesis Plan: Polish Attitudes toward Ukraine in YouTube Comments (2022–2026)

## 1. Working title

*Target-based sentiment and emotion analysis of Polish YouTube comments on Ukraine-related news, 2022–2026*

## 2. Research questions

- **RQ1:** How did comment sentiment toward each target (see §3) change between 2022 and 2026?
- **RQ2:** Do changes match key events (see §7) and the trends in opinion polls (CBOS and others)?
- **RQ3:** Do left- and right-leaning channels differ in the level and timing of these changes?
- **RQ4 (optional):** Did negativity move from governments to Ukrainians as people? Which topics drive it (benefits, history, agriculture, ...)?
- **RQ5 (methodological):** How does a fine-tuned HerBERT model compare with LLM zero-shot/few-shot labelling for target-based stance in Polish?

## 3. Targets (aspects)

Each comment can mention several targets (multi-label). Each target in a comment gets its own stance.

| # | Target | Covers |
|---|--------|--------|
| T1 | Polish government | Government, ministers, president, ruling parties |
| T2 | Ukrainian government | Zelensky, ministers, specific Ukrainian politicians and their decisions or statements |
| T3 | Ukrainians (people) | Ukrainians as a nation, refugees, migrants in Poland, Ukrainians in Ukraine |
| T4 | Ukraine (state) | Ukraine as a country, its army, the war effort, military aid, EU/NATO accession |
| T5 | Russia | Russia as a state, Putin, the Russian army (control / reference target) |

**Boundary rules (to refine after the pilot):**

- T2 vs T4: specific people or decisions → T2. The country, army, aid or accession → T4. If agreement stays low after the pilot, merge them.
- T1 changes meaning over time: PiS government until Dec 2023, Tusk coalition after that. The president changed from Duda to Nawrocki in Aug 2025. Store the period with each label and interpret results per period and per channel.
- A plain "Ukraińcy" counts as T3.

**Labels for each target mention:**

- Stance: `positive` / `negative` / `neutral`
- Emotion (optional, Plutchik basic set): anger, fear, sadness, joy, trust, disgust, surprise, anticipation
- Topic tag (optional, mainly for T3): benefits/800+, labour market, history/Volhynia, agriculture/grain, war/security, crime, other

## 4. Data collection

**Channels:** TVN24, TVP Info, Republika, wPolsce24, plus possibly Polsat News and RMF/Onet for balance. Check that each channel existed across the whole period; wPolsce24 in its current form did not.

**Collection rules:**

- Choose videos with a clear, repeatable rule, e.g. every video whose title or description matches a keyword list (`Ukraina`, `Ukraińcy`, `uchodźcy`, `Zełenski`, `Wołyń`, `zboże`, `800+`, `Rosja`, `Putin`, ...) in each month.
- Do not choose only the most-viewed videos, or at least report that choice as a source of bias.
- Use the official **YouTube Data API v3** (`commentThreads.list`, `comments.list`) to get exact `publishedAt` timestamps.
  - ⚠️ `youtube-comment-downloader` returns relative times ("2 lata temu"), so `time_parsed` is only accurate to roughly a year for old comments. That isn't precise enough for time-series analysis.
- Store: comment id, video id, channel, video publish date, comment publish date, text, like count, reply flag, parent id, pseudonymised author id.
- Record the collection date. Deleted or moderated comments are missing, so report this as a limitation.

**Quota estimate:** `commentThreads.list` costs 1 unit per call and returns up to 100 comments. The default 10,000 units/day is enough for hundreds of thousands of comments over a few days.

## 5. Pre-processing

1. Remove duplicates and spam (identical texts, link-only comments).
2. Detect language; keep Polish and decide how to handle Ukrainian and Russian comments.
3. Normalise text (URLs, mentions, repeated characters). Keep emojis, because they carry sentiment.
4. Filter by topic so only Ukraine/Russia-related comments remain (keywords + BERTopic or a classifier).
5. Pseudonymise authors (hashed IDs) for GDPR.

## 6. Annotation and modelling

### 6.1 Annotation

- **Pilot:** about 500 comments to check target frequencies, class balance and the boundary rules.
- **Gold set:** about 3,000–5,000 comments, stratified by channel, year and target.
- **Second annotator** on about 300–500 comments; report Cohen's kappa for each target and stance.
- Write annotation guidelines with examples; include them as an appendix.
- Possible tools: Label Studio, Doccano.

### 6.2 Models

1. **Baselines:** `Voicelab/herbert-base-cased-sentiment` (overall sentiment, no target), lexicon-based.
2. **Main model:** fine-tune HerBERT (`allegro/herbert-base-cased` or large):
   - target detection (multi-label), then stance for each target, or
   - a single multi-task model with one classification head per target.
3. **LLM approach:** zero-shot / few-shot labelling (Bielik, GPT-class models), evaluated on the gold set.
4. **Optional distillation:** use LLM labels to train a smaller HerBERT model, then compare.
5. **Emotions:** fine-tune on PolEmo 2.0 and/or your own labels.

**Metrics:** macro-F1 for each target and stance, confusion matrices, error analysis focused on sarcasm and irony.

## 7. Analysis

- **Time series:** share of negative/positive stance per target, per channel, per week or month. Normalise by comment volume.
- **Change point detection:** e.g. `ruptures` (PELT, binary segmentation).
- **Event alignment.** Candidate events:
  - Feb 2022: full-scale invasion, refugee wave
  - 2023: grain import dispute; Zelensky's UN speech (Sept 2023)
  - Nov 2023 – early 2024: truckers' border blockade, farmers' protests
  - Dec 2023: change of government in Poland; TVP takeover
  - 2025: Volhynia exhumations debate, presidential campaign, debate over 800+ benefits for Ukrainians, Nawrocki's veto (Aug 2025)
  - Add further events through 2026
- **Validation against polls:** correlate monthly sentiment toward T3 and T4 with CBOS attitudes toward Ukrainians and support for helping Ukraine. Test lead/lag (cross-correlation, Granger causality).
- **Channel comparison:** TVN24 vs Republika/wPolsce24. TVP before and after Dec 2023 as a natural experiment.
- **Topic drivers:** BERTopic or topic tags within T3 negativity.
- **Optional:** cross-target patterns, e.g. anti-Ukrainian and pro-Russian in the same comment, as a signal of possible troll or disinformation accounts.

## 8. Limitations and threats to validity

- Commenters are a self-selected group, not representative of Polish society. Frame findings as online discourse.
- Bots, trolls and coordinated campaigns.
- Moderation and deletion by channels; survivorship bias.
- Video selection bias (keywords and popularity).
- Sarcasm and irony; implicit targets.
- T1 changes meaning when governments change.
- Channel availability across the full period.

## 9. Ethics and legal

- Comments contain personal data under GDPR. Pseudonymise, minimise stored data, and don't publish raw identifiable data.
- Check YouTube API Terms of Service (data storage and refresh rules).
- Ask the faculty whether ethics committee approval is needed.
- Share only aggregated results or comment IDs, not full texts with authors.

## 10. Pipeline overview

```
YouTube Data API ─► raw comments (per channel/video)
        │
        ▼
cleaning + dedup + language filter + pseudonymisation
        │
        ▼
topic filter (keywords + BERTopic) ─► Ukraine/Russia-related comments
        │
        ├─► pilot annotation ─► guidelines ─► gold set (+ 2nd annotator, kappa)
        │
        ▼
models: baseline │ fine-tuned HerBERT (target + stance + emotion) │ LLM zero/few-shot
        │
        ▼
evaluation on gold set ─► best model labels the full corpus
        │
        ▼
time series per target × channel ─► change points ─► events ─► CBOS correlation
        │
        ▼
results, discussion, limitations
```

## 11. Proposed thesis structure

1. Introduction: motivation, research questions, contributions
2. Background: Polish–Ukrainian relations 2022–2026; sentiment/stance/emotion analysis; Polish NLP resources
3. Related work: social media opinion analysis on the war in Ukraine, target-based sentiment
4. Data: collection, corpus statistics, ethics
5. Annotation: scheme, guidelines, agreement
6. Methods: models, training, evaluation
7. Results I: model evaluation
8. Results II: analysis of the time series, channels, events and polls
9. Discussion and limitations
10. Conclusions and future work

## 12. Rough timeline

| Phase | Tasks |
|-------|-------|
| 1 | Literature review, final research questions, channel and keyword list |
| 2 | Data collection with the API, cleaning, topic filtering |
| 3 | Pilot annotation, guidelines, gold set |
| 4 | Model training and evaluation |
| 5 | Analysis: time series, events, polls, channels |
| 6 | Writing and revisions |

## 13. Current state of the code (this folder)

- `youtube_top_videos_by_month.py`: top videos per month via the API. Switch selection from "most viewed" to "keyword match", or report the bias.
- `youtube_channel_top_comments_scraper.py`, `yt_scrapper.py`: use `youtube-comment-downloader`, which only gives relative timestamps. Replace with `commentThreads.list` from the API.
- `polish_sentiment.py`: HerBERT sentiment baseline. Useful as Baseline 1.
- `sentiment_translation.py`: translation + VADER. A possible extra baseline for showing why native Polish models are needed.

## 14. Next steps

- [ ] Finalise the channel list and keyword list
- [ ] Rewrite the comment collector to use the YouTube Data API with exact timestamps
- [ ] Collect a pilot corpus (e.g. 2 channels × 3 months)
- [ ] Label the 500-comment pilot and refine the target boundaries
- [ ] Get CBOS reports on attitudes toward Ukrainians, 2022–2026
- [ ] Discuss the scope with your supervisor
