# Flight Price Analysis and Prediction (R)

Course project for *Data Mining & Knowledge Discovery* (Master MLDM, 2025-2026).
The goal is to understand **what really drives the price of a US domestic flight ticket**, and to build models that predict it.
The full report is in this repo: [`Rapport_DMKD Bilal Bessaa.pdf`](Rapport_DMKD%20Bilal%20Bessaa.pdf).

## Questions
- Do airlines price only by distance, or do stops, booking date and departure time matter too?
- Are more expensive tickets better rated by passengers?
- How well can we predict the price?

## Datasets (from Kaggle, not stored in the repo)
- **Expedia flight prices**: https://www.kaggle.com/datasets/dilwong/flightprices/data (US domestic flights, April-October 2022, 6M+ rows in total).
- **Airline reviews**: https://www.kaggle.com/datasets/juhibhojani/airline-reviews (about 129,000 passenger reviews).

Download and unzip them in the project folder (`itineraries.csv` and `AirlineReviews.csv`).

## Project structure
| File | Content |
|---|---|
| `DataPreparationPropre.Rmd` | Data preparation: cleaning, missing values, outliers, feature engineering, merging the two datasets. Writes `df_expedia_prepared.csv`. |
| `AnalysisNoteBook.Rmd` | Exploratory analysis, maps, correlations, airline clustering (k-means) and price prediction. Reads `df_expedia_prepared.csv`. |
| `Rapport_DMKD Bilal Bessaa.pdf` | Final report. |

Run the preparation notebook first, then the analysis notebook.

## Method
1. **Preparation**: matched airline names between the two datasets (and removed a false match: *Southern Airways Express* vs *China Southern Airlines*); kept the average review score per airline and cabin class (at least 15 reviews, otherwise the airline's overall score); created `nb_escales`, `days_before_flight`, `duration_minutes`, `flight_weekday`, `route`; imputed the 6% missing distances with the mean distance of the same route. Final dataset: **961,497 rows, 26 columns**.
2. **Exploration**: price by airline, cabin, number of stops, booking delay, departure hour, weekday; maps of routes and airports; correlation heatmap.
3. **Clustering**: k-means on airline profiles (median price, duration, distance), k = 3 with the elbow method.
4. **Modeling**: 80/20 train/test split, 15 variables, 5-fold cross-validation on the train set. Linear regression (baseline), Random Forest (`ranger`), XGBoost (random search for hyper-parameters: `eta` = 0.1, `max_depth` = 6, 400 rounds).

## Results (test set)
| Model | RMSE ($) | R² | MAPE (%) |
|---|---|---|---|
| Linear regression | 116 | 0.541 | 34.4 |
| **Random Forest** | **78.5** | **0.796** | **21.6** |
| XGBoost | 83.9 | 0.759 | 22.3 |
| Random Forest without the review score | 76.0 | 0.808 | 20.8 |

### Main findings
- **Flight duration and distance** are by far the most important variables for the price, then the number of stops.
- The cheapest tickets are bought **15-30 days before departure**, for **Tuesday/Wednesday** flights, during daytime. Night flights and weekend departures are more expensive.
- Airlines show 3 price levels (low-cost around $150-250, mid-range $350-400, premium above $450), but the k-means clusters follow duration and distance more than the "low-cost vs premium" label.
- **Review scores do not help the prediction**: the model without them is slightly better. This makes sense, satisfaction is a consequence of service quality, not a driver of price.
- The Random Forest beat the tuned XGBoost here. The 21% MAPE is mostly due to cheap tickets ($30-50), where a small error in dollars is a big error in percent.

## Limitations
- `DataPreparationPropre.Rmd` loads only the **first 1,000,000 rows** of the Expedia file (`nrows = 1000000`), not a random sample. If the file is ordered by date, some periods may be under-represented.
- Only April-October 2022: no Christmas or winter effects.
- No information on promotions, loyalty programs or extra fees, so predictions are limited.
- Some counter-intuitive results (more stops are not cheaper) are specific to domestic US routes.
- Notebook comments are partly in French.

## How to run
Install the R packages used in the notebooks:
```r
install.packages(c("tidyverse", "corrplot", "ranger", "randomForest", "xgboost", "caret",
                   "scales", "lubridate", "factoextra", "maps", "ggrepel", "viridis", "purrr"))
```
Then knit `DataPreparationPropre.Rmd`, then `AnalysisNoteBook.Rmd`.

## Skills shown
R, data cleaning and merging, feature engineering, EDA and visualization, k-means, Random Forest, XGBoost, cross-validation, report writing.
