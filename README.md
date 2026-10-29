
# Patient Experience Analysis (HCAHPS)

What actually drives a good patient experience in US hospitals? A real-data analysis of national HCAHPS survey results across 4,790 hospitals.

## The problem

I approached this as a data analyst for a hospital network. The Chief Experience Officer asked: patient experience scores affect how much Medicare pays us, so which specific parts of a patient's stay matter most to their overall opinion of us, and where should we focus first?

## Data source

**CMS Patient Survey (HCAHPS) - Hospital**, pulled live via the [CMS Provider Data API](https://data.cms.gov/provider-data/dataset/dgck-syfz) a national, standardized survey every Medicare-participating hospital must run. Covers 4,790 hospitals and 9 patient-experience measures, for surveys completed October 2024–September 2025.

No file download needed the data was pulled directly from CMS's live API.

## Steps I took

1. Connected to CMS's Provider Data API and pulled the 9 star-rating measures for all hospitals (43,110 rows out of 325,720 total — filtering to just what was needed).
2. Hit and solved a real API problem: the system rejected multiple filter approaches (wrong distribution ID, wildcard characters getting corrupted by URL encoding, unsupported list-style filters) before finding that the API only accepts one simple condition at a time. Fixed it by pulling each of the 9 measures separately and stacking the results.
3. Checked the data confirmed all 9 measures were present, and understood that "Not Applicable" scores are intentional: CMS suppresses ratings for hospitals with too few survey responses to trust.
4. Cleaned the data, then reshaped it from "one row per hospital per question" into "one row per hospital with 9 columns" so measures could be compared side by side for the same hospital.
5. Loaded into SQLite and used SQL and Python to answer five business questions.
6. Charted the results.

## Key code

```python
# The data arrives as one row per hospital per question (9 rows per hospital).
# Reshaping it into one row per hospital with 9 columns lets me compare
# measures against each other for the same hospital.
wide = d.pivot_table(
    index=["facility_id", "facility_name", "state"],
    columns="measure",
    values="star_rating"
).reset_index()
```

```sql
-- Compare average overall hospital rating by state
-- (only states with 10+ hospitals reporting, so one hospital can't skew a state average)
SELECT state,
       COUNT(*) AS hospitals,
       ROUND(AVG("Overall hospital rating"), 2) AS avg_overall_rating
FROM hospitals
GROUP BY state
HAVING COUNT(*) >= 10
ORDER BY avg_overall_rating DESC;
```

## What I found

| Measure | Avg. star rating (1–5) | Correlation with overall rating |
|---|---|---|
| Nurse communication | 3.43 | 0.79 |
| Communication about medicines | 2.71 | 0.72 |
| Doctor communication | 3.12 | 0.71 |
| Discharge information | 3.31 | 0.65 |
| Quietness | 3.27 | 0.62 |
| Cleanliness | 3.44 | 0.58 |

- **Communication about medicines scores lowest nationally** (2.71/5) but it's not the measure that matters most.
- **Nurse communication is most closely linked to a hospital's overall rating** (0.79 correlation) the clearest answer to "what should we fix first?"
- **Ratings vary widely by state:** South Dakota, Nebraska, and Wisconsin average highest (3.8–4.0 stars); New York, New Jersey, and Puerto Rico average lowest (2.2–2.8 stars) states with 10+ hospitals reporting only.
- **Smaller hospitals tend to rate higher:** 100–300 surveys → 3.75 stars average, dropping to ~3.1–3.2 for hospitals with 300+ surveys.
- **"Overall rating" and "would recommend" almost always agree**  only 0.2% of hospitals show a real (2+ star) gap between the two.

## What I'd recommend

1. Start with nurse communication, not the lowest-scoring measure it's what's most linked to overall patient opinion.
2. Don't over-interpret small hospitals' extreme scores a group with only 3 hospitals in this data showed a misleadingly high average and was excluded from the final analysis.
3. Use state-level patterns to know where to look closer, not to judge individual hospitals.
4. Track "would recommend" alongside "overall rating" a gap between them can reveal specific dissatisfaction a single score would hide.

## Limits of this analysis

- Suppressed ratings aren't random CMS hides scores for hospitals with too few responses, so smaller or less-visited hospitals are underrepresented.
- **1,607 of 4,790 hospitals excluded entirely** every one of their 9 measures was suppressed.
- Correlation is not causation.
- Covers one survey year only (Oct 2024–Sept 2025) can't show change over time.
- State averages blend very different hospitals together.

## Tools used

Python (pandas), SQL (SQLite), CMS Provider Data API, Google Colab, Looker Studio

## Links

- Live dashboard: https://datastudio.google.com/reporting/8e94a9f3-c65c-4652-ac50-0f2d51dbf42e
- Case study write-up: *(add Notion link)*
- Notebook: `hcahps_patient_experience_analysis.ipynb` (in this repo)

---
*Data source: Centers for Medicare & Medicaid Services (CMS), Patient Survey (HCAHPS) — Hospital, pulled live via the CMS Provider Data API.*
```

