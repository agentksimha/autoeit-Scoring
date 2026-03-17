# AutoEIT: Automated EIT Scoring Pipeline

**GSoC 2026 Application — AutoEIT Project**

---

## Overview

This pipeline implements a consistent, reproducible scoring engine for the Spanish Elicited Imitation Task (EIT). Each sentence is assigned a **0–4 ordinal score** following the Ortega (2000) meaning-based rubric. Sentence scores are summed to produce a **total EIT score on the 0–120 scale** (30 sentences × 4-point rubric).

The approach combines three complementary similarity features — WER, semantic embeddings, and length ratio — aggregated via equal-weighted mean and bucketed into ordinal scores. The pipeline is designed to be transparent, fully documented, and calibration-ready once human rater scores become available.

---

## Scoring Architecture

### Features

| Feature | Rubric Criterion | Method |
|---|---|---|
| WER Similarity | Word-level accuracy | `1 − WER(stimulus, transcription)`, floored at 0 |
| Semantic Similarity | Meaning preservation | Cosine similarity of multilingual sentence embeddings |
| Length Ratio | Response completeness | `min(len₁, len₂) / max(len₁, len₂)` |

**Why these three features:**

- **WER** is the most empirically validated metric for automated EIT scoring. McGuire & Larson-Hall (2024) demonstrated that WER-based automated scoring achieves close agreement with experienced human raters on Spanish EIT data.
- **Semantic similarity** addresses the core 2/3 boundary in the Ortega rubric — distinguishing grammatical changes that do and do not affect meaning. Multilingual sentence embeddings (`paraphrase-multilingual-MiniLM-L12-v2`) capture this distinction language-agnostically without requiring Spanish-specific NLP resources.
- **Length ratio** provides an explicit completeness penalty consistent with all EIT rubric variants. A response with significantly fewer words than the stimulus is necessarily incomplete regardless of word accuracy.

**Why LCS was not included:**

LCS similarity and WER showed r = 0.97 correlation in feature analysis, confirming they measure the same dimension. Retaining both would double-weight word-order/lexical overlap and undermine the equal-weighting rationale. WER is retained as the empirically validated metric; LCS is excluded.

### Aggregation

Features are combined using **plain mean (equal weighting)**. This is the most transparent and reproducible aggregation when ground truth labels are unavailable for weight calibration. Fixed weighted combinations require empirical optimisation — without labels, any non-equal weighting would be arbitrary and not reproducible across datasets.

### Score Bucketing (0–4)

The continuous composite (0–1) is mapped to the Ortega rubric scale:

| Composite Range | Score | Rubric Interpretation |
|---|---|---|
| Garbled / empty response | 0 | No response or entirely unintelligible |
| 0.00 – 0.25 | 1 | Minimal repetition, item abandoned |
| 0.25 – 0.55 | 2 | ~Half idea units preserved; meaning incomplete |
| 0.55 – 0.85 | 3 | Full meaning preserved (may be ungrammatical) |
| 0.85 – 1.00 | 4 | Exact or near-exact repetition |

Thresholds are grounded in rubric definitions. Per the protocol ("when in doubt, score 2"), the 2/3 boundary is set conservatively at 0.55. Thresholds can be refined via regression calibration once human rater scores are available.

---

## Pipeline Steps

1. **Preprocessing** — Remove memory load numbers `(7)`, bracket annotations `[pause]` `[gibberish]`, `xxx` placeholders; lowercase; strip punctuation; normalise whitespace.
2. **Garble gate** — Empty or entirely unintelligible transcriptions are assigned Score 0 before any metrics are computed.
3. **Feature extraction** — WER similarity, semantic similarity, length ratio computed for each sentence pair.
4. **Aggregation** — Plain mean of three features → composite score (0–1).
5. **Bucketing** — Composite mapped to 0–4 ordinal score per rubric thresholds.
6. **Total score** — Sum of sentence-level scores → total EIT score (0–120).

---

## Requirements

```
jiwer
sentence-transformers
pandas
numpy
seaborn
matplotlib
```

Install with:

```bash
pip install jiwer sentence-transformers pandas numpy seaborn matplotlib
```

---

## Usage

1. Update `INPUT_PATH` in Cell 3 to point to your transcriptions CSV.
2. Run all cells in order.
3. Scored output is saved to `autoeit_scored_output.csv`.

### Expected CSV format

| Column | Description |
|---|---|
| `Sentence` | Sentence number (1–30) |
| `Stimulus` | Target sentence presented to participant |
| `Transcription Rater 1` | Participant's transcribed response |

### Output columns

| Column | Description |
|---|---|
| `Score_0_4` | Ordinal sentence score (0–4, Ortega rubric) |
| `Composite` | Raw composite score (0–1, before bucketing) |
| `WER_Similarity` | WER-based similarity (0–1) |
| `Semantic_Similarity` | Embedding cosine similarity (0–1) |
| `Length_Ratio` | Response completeness ratio (0–1) |

---

## Validation

The pipeline includes 8 validation cases drawn from the Ortega (2000) rubric and protocol documentation. Expected vs. predicted scores are printed at runtime. Target: ≥ 80% exact agreement on known rubric examples.

---

## Limitations and Next Steps

| Limitation | Mitigation / Next Step |
|---|---|
| No ground truth labels available | Calibrate bucket thresholds and feature weights via regression once human rater scores are available |
| Equal weighting is a baseline assumption | Optimise weights against human rater scores; report Cohen's κ or Pearson r |
| Semantic model not fine-tuned on learner Spanish | Fine-tune on L2 Spanish EIT data if available; evaluate against language-specific models |
| Bucket thresholds are rubric-derived, not empirically fitted | Fit thresholds to maximise agreement with human raters on a held-out validation set |

---

## References

- McGuire, M., & Larson-Hall, J. (2024). Assessing Whisper automatic speech recognition and WER scoring for elicited imitation: Steps toward automation. *Language Testing*.

- Ortega, L. (2000). The relationship between gist comprehension and EIT performance. In R. Ellis (Ed.), *Form-focused instruction and second language learning*. Blackwell.


