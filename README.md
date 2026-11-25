# Bank Term Deposit Subscription Prediction

Predicts whether a bank client will subscribe to a term deposit after a phone marketing campaign, and ranks clients so a marketing team can prioritize calls. Built as a capstone project (PRCP-1000) on the Portuguese bank marketing dataset.

## Key results

The selected model is **logistic regression without macroeconomic inputs** (C = 10). On the held-out chronological test set (8,236 clients):

| Measure | Value (95% CI) |
|---|---|
| ROC-AUC | 0.700 (0.689–0.712) |
| PR-AUC | 0.495 (base rate 0.308) |
| Top 20% called: recall / lift | 36.3% / 1.82 |

Within a single time period, the signal is weak (fold-level ROC-AUC about 0.67–0.69), and most of the change in subscription rates over the campaign is not explained by client characteristics.

## Important limitations

- **Probabilities are under-calibrated** for the later period (mean predicted 0.18 vs observed 0.31). Recalibrate on recent data before using them as response rates.
- **Use a capacity-based call list** (top k% by score) rather than the fixed probability threshold, which flags about 38% of clients under the observed shift.
- **Lower reliability for clients aged 66+** (within-group ROC-AUC 0.576).
- **Earlier results were inflated.** An earlier random split mixed time periods; the figures above come from the corrected chronological design.

This is an analytical tool for prioritizing outreach. It must not be used for credit, pricing or eligibility decisions.

## Data

- Source: `bank-additional-full.csv` (semicolon-separated), from the Portuguese bank direct-marketing campaigns, May 2008 to November 2010.
- 41,188 records, reduced to 41,176 after removing exact duplicates.
- Target: `y` (subscribed to a term deposit), 11.3% positive.
- **Call duration is excluded** from all models, because it is known only after the call. It appears only in a labelled benchmark.

Dataset citation: S. Moro, P. Cortez and P. Rita (2014). A Data-Driven Approach to Predict the Success of Bank Telemarketing. *Decision Support Systems*. doi:10.1016/j.dss.2014.03.001

## Method

1. **Data validation:** duplicate removal, unknown-category analysis, and a pre-call availability audit of every input.
2. **Exploratory analysis:** descriptive charts, each with numerical observations.
3. **Validation design:**
   - *Chronological protocol:* first 60% train, next 20% validation, final 20% test (used once).
   - *Random protocol:* 5-fold CV inside the first 80% of the data, kept separate from the test period.
4. **Feature selection:** made on training data only, with a tie rule for the `pdays` encoding.
5. **Model selection:** on validation evidence only. The selection rule was revised after the first rule selected a model with validation AUC below 0.5; both outcomes are reported in the notebook.
6. **Evaluation:** bootstrap confidence intervals, targeting analysis at 10/20/30%, calibration, and subgroup checks.

## Repository contents

```
├── PRCP-1000_Bank_Term_Deposit_v2_Final.ipynb   # main notebook (executed)
├── bank-additional-full.csv                     # dataset (semicolon-separated)
└── README.md
```

## Running the notebook

Open the notebook in Google Colab and choose **Runtime → Run all**. In Colab you will be asked to upload `bank-additional-full.csv`. Locally, place the file in the same folder as the notebook.

Requirements: Python 3.10+, pandas, numpy, scikit-learn, matplotlib, seaborn.

```bash
pip install pandas numpy scikit-learn matplotlib seaborn
jupyter notebook PRCP-1000_Bank_Term_Deposit_v2_Final.ipynb
```

The notebook runs in about five minutes on a single CPU.

## Reproducibility

- Random seed: 42 for all splits, models and resampling.
- Bootstrap resamples: 1,000.
- Validation and test blocks are fixed by record position, so the file must keep its original date order.

## Business recommendations (association, not causation)

- Prioritize clients with a previous campaign success (65.1% subscription rate).
- Test call caps, channel and timing changes in randomized pilots before scaling them.
- Review the model for older clients before any operational use.

## License and attribution

The dataset is publicly available for research. Please cite the publication above if you use it.
