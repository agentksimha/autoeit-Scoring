# AutoEIT: Rule-Based Automated Scoring Pipeline

A consistent, rule-driven, reproducible scoring engine for the Spanish Elicited Imitation Task (EIT). Automatically scores learner transcriptions against stimulus sentences using NLP metrics grounded in the Ortega et al. (2002) EIT scoring rubric.

## Features

| Feature | Rubric Criterion |
|---|---|
| WER Similarity | Word-level accuracy — primary metric |
| Semantic Similarity | Meaning preservation |
| LCS Similarity | Word order and syntactic preservation |
| Length Ratio | Response completeness |

Features are aggregated using **plain mean** and scaled to **0–120** to match the standard EIT score range.

## Requirements

```bash
pip install jiwer sentence-transformers pandas numpy seaborn matplotlib
```

## Usage

1. Output saved to `eit_prediction_final.csv`

## Limitations

Full validation requires human rater ground truth labels. Equal weighting is a defensible baseline pending supervised calibration.

## References

- Ortega et al. (2002). An investigation of elicited imitation tasks in crosslinguistic SLA research. 20th SLRF, Toronto.
- Kostromitina & Plonsky (2022). EIT as a measure of L2 proficiency: A meta-analysis. *SSLA*, 44(3), 886–911. https://doi.org/10.1017/S0272263121000395

---
*Submitted as part of the HumanAI GSoC 2026 evaluation test for the AutoEIT project.*
