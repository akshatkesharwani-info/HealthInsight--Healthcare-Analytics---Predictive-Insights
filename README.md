# HealthInsight: Healthcare Analytics & Predictive Insights

Hospital population analytics on **made-up patient data**: which conditions affect which age groups, which patients are likely to be readmitted within 30 days, and how to rank them so a care team can call the riskiest ones first. An AI-written brief turns the numbers into a short report.

Built in Google Colab with Groq (`openai/gpt-oss-120b`).

> **All 6,000 patients are randomly generated.** This project shows an analytics method. It is not medical advice, and none of the findings describe real patients. The notebook also accepts your own `patients.csv` with the same columns.

## What it does

1. **Checks the data** (missing values, duplicates, summary statistics).
2. **Conditions by age group and department:** the diagnosis mix per age band and the readmission rate per department and diagnosis.
3. **Tests whether departments really differ** from the overall rate (a z-score check), so random noise is not mistaken for a pattern.
4. **Risk factors:** correlations and readmission rate by prior admissions and smoking.
5. **Trains two models** (Logistic Regression and Gradient Boosting), keeps the one with the higher AUC, and ranks drivers with permutation importance.
6. **Ranks patients by risk** with a decile lift table and Low / Medium / High tiers, and checks whether predicted risk matches actual risk.
7. **AI population-health brief (Groq):** the AI is only given calculated numbers, told to say rates are rates inside a group (not shares of all readmissions), not to suggest department-specific actions when no department stands out, and not to give individual medical advice.

## Results from the run

| Measure | Result |
|---|---|
| Patients / overall 30-day readmission | 6,000 / 23.3% |
| Highest-risk diagnosis | Cardiac, 34.7% readmitted (1,042 patients), against 19.5% for "Other" |
| Readmission by prior admissions | 11.8% with none, 19.9% with one, 33.2% with two, 47.3% with three, 66.3% with four |
| Departments | all within about 0.5 of the overall rate (z-scores from -0.51 to +0.48), so **no department stands out** |
| Model AUC | Logistic Regression **0.758**, Gradient Boosting 0.733 (the simpler model won) |
| Top-decile lift | **2.61x** (60.7% actually readmitted; the model predicted 59.8%) |
| Risk tiers (actual readmission) | High (300 patients) 50.3%, Medium (600) 24.5%, Low (600) 8.5% |
| Coverage | contacting the top 20% riskiest patients reaches **43.3%** of all readmissions |

**Biggest model drivers** (drop in AUC when a factor is shuffled): prior admissions 0.108, age 0.083, then diagnosis "Other" 0.018.

## What the evaluation showed

- **Prior admissions are by far the strongest signal.** The readmission rate rises from 11.8% to 66.3% as prior admissions go from 0 to 4.
- **Department differences are noise.** On a chart the departments look different, but none is statistically distinguishable from the overall rate. Acting on them would be chasing randomness.
- **The simpler model is enough.** Logistic Regression beat Gradient Boosting here, so the notebook uses it and says so.
- **Rank patients, don't classify them.** At a 0.5 cut-off the model catches only 24% of readmitted patients (precision 0.68). Ranking by risk and calling the top tier is far more useful: the top decile is readmitted 2.6x more often than average.
- **The scores are well calibrated at the top.** Predicted and actual risk match closely in the highest-risk decile (59.8% vs 60.7%).

## Limitations

- **Made-up data, made-up relationships.** The risk factors were built into the generator, so the drivers describe the generator and not real clinical patterns.
- The brief is written by an AI from calculated numbers. It is a draft for a human to read, and it should never be used for individual medical decisions.
- No time element, no discharge details and no clinical coding. A real readmission model needs those and a fairness review.
- The model should be used for ranking follow-up calls, not as exact probabilities for individuals.

## Tech stack

scikit-learn, pandas, matplotlib, seaborn, Groq API.

## How to run

1. Open the notebook in Google Colab.
2. Run the cells from top to bottom.
3. When asked, paste your own Groq API key (free at console.groq.com). It is hidden while typing and is never saved in the notebook.

## Files the notebook creates

- `healthinsight_patients_scored.csv`: every patient with a risk score and tier
- `healthinsight_lift_table.csv`: the decile lift table

---

Built by **Akshat Kesharwani** | [GitHub](https://github.com/akshatkesharwani-info) | [Portfolio](https://akshatkesharwani-info.github.io/)
