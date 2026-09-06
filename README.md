# Jumia-Product-Analysis-2026
# Jumia Product Performance Dashboard
## Introduction

This project analyzes a sample of product listings from Jumia, an e-commerce marketplace, to understand how price, discount depth, ratings, and customer engagement (review counts) relate to one another and to users- insights that a seller or marketplace team could use. The entire workflow data cleaning, threshold definition, flagging, ranking, correlation analysis, and visualization was built in Microsoft Excel, using table-based formulas, named ranges, quartile-based thresholds, and native charts.

What we need to consider is: Do price and discounting actually drive customer engagement and satisfaction?

## Dataset Overview
- **Source:** Product listings exported from Jumia (Kenya), covering household, electronics, and lifestyle categories.
- **Size:** 115 products, 6 original columns: `Product`, `Current price`, `old price`, `Discount`, `Review`, `Ratingd`.
- **Price range:** KSh 38 – KSh 3,750.
- **Discount range:** 1% – 64%.
- **Data completeness:** Price and discount data are complete for all 115 products. **Review count and rating are missing for 58 of 115 products (50%)** this gap is preserved and clearly labeled throughout the analysis rather than treated as zero.

Raw data issues found and corrected during cleaning:
- Prices stored as text with currency symbols and thousands separators (e.g. `"KSh 1,980"`).
- One product listed as a **price range** rather than a single value (e.g. `"KSh 1,620 - KSh 1,980"`).
- Ratings stored as text (e.g. `"4.5 out of 5"`) instead of numeric values.
- Review counts stored as **negative numbers** (e.g. `-14`), which appears to be a data extraction artifact rather than a true negative quantity.
- Discount stored as text with a `%` sign.

## Data Enrichment
Each raw data issue was resolved with a specific understanding of data:

| Field | Issue | Resolution |
|---|---|---|
| `Current price` / `old price` | Text with `KSh` and commas; one row was a price range | Stripped currency text and commas, converted to numeric; the single range row was resolved to its **midpoint** |
| `Ratingd` | Text like `"4.5 out of 5"` | Extracted the numeric portion, converted to a decimal number |
| `Discount` | Text with `%` | Stripped the `%` and converted to numeric |
| `Review` | Stored as negative integers | Derived a new `Engagement` field = `ABS(Review)`, since review counts cannot be negative; the true engagement measure is this absolute value |

**Threshold definition (quartile-based, not arbitrary):**

All high/low/strong/weak categories in this project are defined using the **first (Q1)** and **third (Q3) quartiles** of the actual dataset, calculated with `QUARTILE.INC`, so the thresholds adapt to the data rather than relying on a fixed guess:

| Metric | Q1 | Q3 |
|---|---|---|
| Price | KSh 500 | KSh 1,663 |
| Discount | 27% | 49% |
| Rating | 3.0 | 4.6 |
| Engagement (reviews) | 5 | 14 |

**Derived flags** (each computed with an Excel formula referencing these named threshold cells):
- **Price Tier:** Low / Medium / High
- **High Discount & Low Rating:** Discount ≥ 49% AND Rating ≤ 3.0
- **High Discount & Low Engagement:** Discount ≥ 49% AND Reviews ≤ 5
- **Many Reviews & Average Rating:** Reviews ≥ 14 AND Rating between 3.0–4.6
- **Excellent Rating:** Reviews ≥ 14 AND Rating ≥ 4.6

Every flag return `"Missing"` instead of `0` or blank, When the underlying rating or review data doesn't exist, so the dashboard never silently misrepresents a data gap as poor performance.
## Key Insights

1. **Demand is concentrated, not broad-based.** Only 16 of 57 rated products (28%) meet the "Strong Engagement" threshold (≥14 reviews).
2. **Discounting does not reliably drive demand.** 13 of the highest-discount products (≥49%) still fall below the engagement threshold.
3. **A handful of high-discount products carry real reputation risk.** 6 products combine deep discounts with weak ratings (≤3.0); one of them also has the highest review count in the whole dataset.
4. **Price, discount, and rating are essentially uncorrelated.** All three Pearson correlations tested came out near zero (|r| < 0.15, R² < 0.02).
5. **The catalog has a solid base of "reliable, average" sellers** 10 products combine strong engagement with a respectable but not exceptional rating, representing the best low-effort improvement opportunity.

## Discount Analysis

Products were split into Low (≤27%), Medium (27–49%), and High (>49%) discount tiers using the quartile thresholds above.

| Discount Tier | Avg. Reviews | Avg. Rating | Count |
|---|---|---|---|
| Low (≤27%) | 16.7 | 4.19 | 13 |
| Medium (27–49%) | 12.8 | 3.89 | 33 |
| High (>49%) | 7.6 | 3.53 | 11 |

Contrary to what a common belief may suggest, sales spikes are not because of a discount pattern: the number of reviews for the least discounted products is more than double in comparison with the most discounted products, with their average rating also being higher. The Pearson correlation between discount and review count is equal to **r=-0.14** and **R²=0.02**, which is a very weak negative correlation, indicating that it is consistent with the tier breakdown.

13 products are in the "Promotion Inefficiency" flag (discount ≥49%, reviews <14) nearly half of the High discount tier. Six products combine a high discount with a low rating (≤3.0), the clearest quality-risk segment in the dataset.

## Ratings and Reviews Analysis

Rating distribution across the 57 products with data:

| Rating Category | Count | % of Rated Products |
|---|---|---|
| Low (≤3.0) | 17 | 30% |
| Average (3.0–4.6) | 26 | 46% |
| Excellent (>4.6) | 14 | 25% |

Average review count by rating band tells an important story: **Average-rated products (16.0 avg. reviews) actually out-engage Excellent-rated products (8.0 avg. reviews)**. The correlation between rating and review count is essentially flat (**r = 0.06**), meaning higher-rated products are not systematically the most-reviewed ones on this platform. This suggests visibility and quality are currently decoupled — a real opportunity, since improving quality on already-visible "Average" products is likely a faster win than trying to generate visibility for untested high-quality products.

Price and rating also show no meaningful relationship (**r = 0.11**, R² = 0.01) — higher-priced items are not rated meaningfully better or worse than lower-priced ones.

Average review count by rating band tells an important story: **Average-rated products (16.0 avg. reviews) actually out-engage Excellent-rated products (8.0 avg. reviews)**. The correlation between rating and review count is essentially flat (**r = 0.06**), meaning higher-rated products are not systematically the most-reviewed ones on this platform. This suggests visibility and quality are currently decoupled a real opportunity, since improving quality on already-visible "Average" products is likely a faster win than trying to generate visibility for untested high-quality products.

Price and rating also show no meaningful relationship (**r = 0.11**, R² = 0.01) higher-priced items are not rated meaningfully better or worse than lower-priced ones.

## Products

A few individual products stand out from the ranked analysis:
- **Highest risk: 120W Cordless Vacuum Cleaner** — the single highest review count in the dataset (69) paired with a weak 2.8 rating and a 49% discount. High visibility with a poor quality signal makes this the top candidate for a quality/fulfillment investigation.
- **Most expensive: 32PCS Portable Cordless Drill Set** — KSh 3,750, the top of the Price tier, but with very limited engagement (5 reviews), raising a visibility question rather than a quality one.
- **Strong performers (Strong Engagement + Excellent Rating):** 6 products meet both criteria — the smallest, most valuable segment in the catalog, and the best candidates for promotional placement since they carry the lowest risk of disappointing a new customer.
- **Reliable, improvable sellers:** 10 products combine high engagement with an "Average" rating — proven demand, with room to move into the "Excellent" tier through targeted quality or listing improvements.
## Conclusion
This study has revealed that there is no significant linear correlation between price, discount depth, customer rating, or customer reviews on the 115 Jumia products studied. The data does not support the premise that deeper discounts create more attraction among customers or that higher prices represent better quality. However, this dataset reveals two specific patterns that can be put into real use:

1. A **small set of high-discount, low-rating products** (6 items) a cordless vacuum cleaner with the platform's highest review count represent a concentrated reputation risk that deserves direct quality investigation rather than further discounting.
- **Highest risk: 120W Cordless Vacuum Cleaner** the single highest review count in the dataset (69) paired with a weak 2.8 rating and a 49% discount. High visibility with a poor quality signal makes this the top candidate for a quality/fulfillment investigation.
- **Most expensive: 32PCS Portable Cordless Drill Set**  KSh 3,750, the top of the Price tier, but with very limited engagement (5 reviews), raising a visibility question rather than a quality one.
- **Strong performers (Strong Engagement + Excellent Rating):** 6 products meet both criteria the smallest, most valuable segment in the catalog, and the best candidates for promotional placement since they carry the lowest risk of disappointing a new customer.
- **Reliable, improvable sellers:** 10 products combine high engagement with an "Average" rating proven demand, with room to move into the "Excellent" tier through targeted quality or listing improvements.

**Recommended next steps**:
1. Begin investigating the high discount/low rating items concerning their quality, fulfillment, or listing accuracy. The highest review count is the first to be worked on.
2. The 13 "promotion inefficiency" items should be audited for listing content and targeting before any further discounts are put in place.
3. Proceed with making any extra quality or content modifications for the 10 high-engagement and average-rating products. 
4. Try to push for an entire collection to be available for the review & rating to be done, as half the catalog is currently invisible to this analysis.
5. Price & discount should be thought of as secondary tactics, as they do not seem to be efficient forms of measuring engagement/rating level in this research. Need to make any extra quality or content modifications for the 10 high-engagement and average-rating products.

**Limitations:** 
The dataset does not include sales volume, conversion rate, listing age, product category, or complaints and returns. The correlations mentioned above are drawn from a single data collection and must be validated through experiments, that is, by conducting experiments on controlled discounts. Moreover, it must be understood that only 50% of products have existing data on ratings and reviews.


