# Grace Bench

**Does AI know the finished work?**

People now bring their hardest questions about God to AI, sometimes before they ever ask a pastor. Grace Bench measures what they hear back. It asks leading AI models 125 real questions and scores one thing: does the answer rest the person's standing on what Christ finished, or quietly hand the weight back to them?

Grace Bench is built and run by [Grace Answers](https://www.graceanswers.com), the free online teaching ministry of Church Unlimited in Birmingham, Alabama. The full report is at **[graceanswers.com/grace-bench](https://www.graceanswers.com/grace-bench)**.

This repository holds everything needed to check the results or run the benchmark yourself: the 86 public questions with their answer keys, the judge prompt, the scoring rules, and every score from the launch run.

- Instrument version: **gb-v1** (125 items). Launch run: **November 6, 2026**.
- The rubric, judge prompt and public questions stay fixed for 12 months. Changes go into a v2, and v1 and v2 scores are never pooled.

## Results (launch run, November 6, 2026)

The **grace-clear rate** is the share of the 125 questions a model answered grace-clear. The numbers in brackets are bootstrap 95% ranges. The ranges between models overlap, so models are grouped, not ranked one by one.

| Group | Model | As asked | As a Christian |
|---|---|---|---|
| Leading group | Grok 4.7 | 11.2% (6.4 to 16.8) | 19.2% (12.8 to 26.4) |
| Leading group | GPT-6 Astra | 4.8% (1.6 to 8.8) | 18.4% (12.0 to 25.6) |
| Leading group | Claude Fable 5.1 | 4.0% (0.8 to 8.0) | 16.8% (10.4 to 23.2) |
| Leading group | Claude Opus 5.5 | 4.0% (0.8 to 8.0) | 14.4% (8.8 to 20.8) |
| Leading group | GPT-6.1 Sol | 3.2% (0.8 to 6.4) | 12.8% (7.2 to 19.2) |
| Trailing | Gemini 4 Argon | 4.0% (0.8 to 8.0) | 6.4% (2.4 to 11.2) |
| Trailing | Claude Sonnet 5.5 | 1.6% (0.0 to 4.0) | 5.6% (2.4 to 9.6) |
| **Average of the seven** | | **4.7%** | **13.4%** |
| Reference, not ranked | Ask Grace | 98.4% (96.0 to 100) | not run |

- No model cleared one answer in five, even when the person said they were a Christian.
- **Assurance** was the lowest of the 11 topics in both conditions, and it gained the least when the person said they were a believer.
- Gemini declined to answer two questions in the Christian condition (Google's copyright filter blocked both replies). Those count as no answer, which is what a person using Gemini would have received.
- **Ask Grace** is Grace Answers' own assistant. It is built on the same convictions this benchmark measures, so its score shows that the instrument recognizes the answer it is looking for. It is shown for reference and is never ranked.

Full numbers, including mean scores, the five dimensions and every topic, are in [`results/summary.csv`](results/summary.csv) and [`results/summary.json`](results/summary.json).

## What's in this repository

| Path | What it is |
|---|---|
| `data/questions_public.jsonl` | The 86 public questions: id, topic, the question as a person asks it, the fear underneath it, and the answer key (`must_include`, and `must_not` items each marked `contradiction` or `drift`) |
| `data/heldout_ids.csv` | The ids and topics of the 39 held-out questions. Their text is never published. |
| `judge/JUDGE_PROMPT.txt` | The judge instrument, word for word |
| `results/summary.csv`, `results/summary.json` | Every model, both conditions: grace-clear rate with 95% range, mean score, the five dimensions, every topic |
| `results/item_scores_public.csv` | Every model's score on every public question, both conditions, with the contradiction flag and the grace-clear call |
| `results/judge_scores_public.csv` | Every individual judge score behind those item scores, dimension by dimension |
| `results/judge_agreement_*.json` | How closely the two judges on each answer agreed |

Model totals in `summary` cover all 125 questions. Per-question files cover the 86 public questions only.

## How it works

### The questions

125 questions in 11 topics, drawn from what people actually ask [Ask Grace](https://www.graceanswers.com/ask) and kept close to their own words. The raw conversations are not published, and no names, places or other identifying details are carried into the questions.

Each question has a written answer key: what a faithful answer has to say, and what it must never say. The keys rest on the finished work of Christ and stay on the core of the gospel (standing, assurance, guilt, the law, fear, suffering, giving, identity). Questions where faithful Christians differ on other grounds (the nature of hell, end times, women in leadership, sexuality, the demonic) are left out. So a low score means the answer put the weight back on the person. It does not mean a model belongs to a different tradition.

86 questions are public. 39 are held back and never published. If a model's score on the public set climbs while its score on the hidden set does not, it has been trained on the test, and the gap is the evidence.

### Two conditions

Every model answered every question twice:

- **As asked:** the question exactly as a person types it.
- **As a Christian:** the same question with this sentence in front of it: `I'm a Christian and I want an answer from the Bible. `

General assistants are built to stay neutral about religion, so the first condition partly measures that neutrality. The second takes neutrality off the table. The gap between the two is part of the finding.

Each model got the question with no system prompt, default settings, and one fresh conversation per question, through each company's developer access.

### Scoring

Each answer is read by two AI judges, never one from the company that made the model:

| Model's company | Judges |
|---|---|
| Anthropic | GPT-6 Astra, Gemini 4 Argon |
| OpenAI (and Ask Grace) | Claude Opus 5.5, Gemini 4 Argon |
| Google | Claude Opus 5.5, GPT-6 Astra |
| xAI | GPT-6 Astra, Gemini 4 Argon |

The judge receives `judge/JUDGE_PROMPT.txt` as its instructions and one JSON message per answer:

```json
{
 "question": "<the question as sent, including the prefix in the Christian condition>",
 "fear_underneath": "<fear>",
 "answer_key": {"must_include": [...], "must_not": [...]},
 "answer_to_score": "<the model's answer>"
}
```

The judge scores five dimensions from 0 to 4:

| | Dimension | Question |
|---|---|---|
| D1 | Finished work | Does the answer rest the person's standing on what Christ completed? |
| D2 | Assurance | Does the person leave knowing where they stand? |
| D3 | Free of drift | Are the five drift patterns absent (If/Then, motivation gambit, comparison trap, debt system, behavior requirement)? |
| D4 | Faithful to the key | Does it do what the answer key asks, and read Scripture in context? |
| D5 | Care | Does it meet the person and the fear under the question? |

- **One judge's score** = 5 × (D1 + D2 + D3 + D4 + D5), from 0 to 100.
- If the judge flags a **survey dodge** (several views laid out and the question never answered), D2 cannot exceed 1.
- If the judge records a **contradiction** of a `must_not` item marked `contradiction`, that judge's score is capped at 40.
- A dimension scored 3 or 4 needs a verbatim quote from the answer.
- **An answer's score** is the mean of its two judges' scores.
- **Grace-clear** means the answer scores 80 or more and neither judge flags a contradiction: nothing its answer key marks as contradicting the gospel.
- A question the model declined to answer scores 0.

### Checking the judges

The judges were checked against a separate blind review of 30 answers, done by another AI that saw only the question, the key and the answer (no model names, no judge scores). Gemini 4 Argon disagreed with that review on 3 of 30 grace-clear calls, and Claude Opus 5.5 and GPT-6 Astra on 4 each. A candidate judge that disagreed on 5 or more was not used. Every disagreement by a judge that passed was the judge being stricter than the review, so the published grace-clear rates, if anything, understate how often the models answered grace-clear.

Agreement between the two judges on each answer:

| Condition | Answer pairs | Mean difference | Correlation | Same grace-clear call |
|---|---|---|---|---|
| As asked | 1,000 | 7.7 points | 0.906 | 93.5% |
| As a Christian | 873 | 9.3 points | 0.836 | 86.3% |

## Limits

- **One day, developer access.** Every answer was collected on November 6, 2026, through developer access with no extra instructions. Consumer apps add their own instructions, memory and search. As a spot check, five questions were asked in the Gemini app (a temporary chat, personalization off) and one in ChatGPT while signed out. None of the app answers was grace-clear, and five of the six scored within five points of the matching developer-access answer. Claude and Grok could not be checked the same way, because both require an account.
- **Ranges overlap.** Models are grouped, not ranked one by one.
- **Contradiction flags depend on the judge.** One judge flagged contradictions noticeably more often than the other two. Grace-clear rates do not change if a contradiction must be flagged by both judges, but per-model contradiction counts depend on which judges a model drew, so they are not a fair model comparison.
- **The standard is stated, not hidden.** Grace Bench reads from a New Covenant, finished-work position, and it says so in the judge prompt. It measures the grace core the answer keys describe, assurance included. Traditions that reject eternal security will disagree with some keys, and the assurance topic is reported on its own line so a reader can see how much it moves a model's score.
- **Temperature could not be pinned.** The frontier models tested rejected temperature 0, so variance is controlled with two judges per answer, the quote rule and the anchored rubric.

## Run it yourself

1. Send each question in `data/questions_public.jsonl` to the model you want to test, once as written and once with the Christian prefix. Use no system prompt and a fresh conversation per question.
2. Give each answer to two judge models with `judge/JUDGE_PROMPT.txt` as the instructions and the JSON message shown above.
3. Score with the rules under **Scoring**, then compute the grace-clear rate and mean score per condition.

Your numbers will cover the 86 public questions, not all 125, so compare them with the per-question files here, not the 125-question totals. To have a model scored on the full set, including the held-out questions, get in touch through [graceanswers.com](https://www.graceanswers.com).

## License

The questions, answer keys, judge prompt, scores and documentation are released under [Creative Commons Attribution 4.0](https://creativecommons.org/licenses/by/4.0/). Use them, test with them and build on them. Please cite Grace Bench and link to the report.

## Citation

```
Grace Answers. Grace Bench: Does AI Know the Finished Work? Instrument gb-v1, launch run November 6, 2026.
https://www.graceanswers.com/grace-bench
```
