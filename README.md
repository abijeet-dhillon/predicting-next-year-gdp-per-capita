# Predicting Next-Year GDP per Capita

This project explores whether a country's current-year economic, energy, infrastructure, urbanization, environmental, and country-level information can predict its GDP per capita in the following year. The analysis covers data cleaning, exploratory visualization, feature selection, regression modelling, and error analysis in a single [Jupyter notebook](notebooks/DATA601_Project.ipynb).

The final model uses XGBoost with a log-transformed target. It achieves a test R² of **0.9782**, but its mean absolute error is slightly higher than a simple forecast that carries the current year's GDP per capita forward. Most of the model's predictive power comes from current-year GDP per capita, with limited additional contribution from the other indicators.

## Data

The data comes from the [World Bank's World Development Indicators](https://databank.worldbank.org/source/world-development-indicators#advancedDownloadOptions).

- **Coverage:** 193 countries and economies over 2006–2021.
- **Processed dataset:** 3,088 country-year observations with 24 indicators, plus country name and year.
- **Target:** Next-year GDP per capita, adjusted for purchasing power parity (PPP), in constant 2021 international dollars.
- **Predictors:** Current-year GDP per capita, inflation, energy use, electricity access and consumption, emissions, internet use, population and urbanization measures, trade, and other development indicators.

The original Excel export is reshaped into one row per country and year. The analysis focuses on 2006–2021 because indicator coverage declines after 2021, and excludes countries with any missing GDP per capita or GDP growth values during that period. Shifting GDP per capita forward within each country produces **2,895 modelling observations**, using predictor years 2006–2020 to predict outcomes in 2007–2021.

## Methodology

1. **Explore the data:** Examine distributions, missing values, trends, and correlations between GDP per capita and development indicators.
2. **Prepare predictors:** Forward-fill missing values within each country, fill remaining gaps with training-set medians, and retain missing-value indicators.
3. **Select features:** Use correlation analysis, Random Forest permutation importance, and chronological validation to select 10 indicators, then add their missing-value flags and one-hot encoded country identifiers.
4. **Compare models:** Evaluate Linear Regression, Decision Tree, Random Forest, and XGBoost using mean absolute error (MAE), root mean squared error (RMSE), and R².
5. **Tune and evaluate:** Tune Random Forest and XGBoost with five expanding chronological cross-validation folds, test log-transformed targets, and evaluate the selected model on later years.
6. **Inspect errors:** Compare against a persistence forecast and examine errors by year, GDP level, country, and size of annual GDP change.

The initial split uses earlier years for training and later years for validation and testing:

| Split | Predictor years | Target years | Observations |
|---|---|---|---:|
| Training | 2006–2015 | 2007–2016 | 1,930 |
| Validation | 2016–2017 | 2017–2018 | 386 |
| Test | 2018–2020 | 2019–2021 | 579 |

After model selection, XGBoost is refitted on the combined training and validation data. The selected parameters are 200 trees, maximum depth 5, learning rate 0.05, and subsample 1.0. Predictions are converted back from the log scale before evaluation.

## Results

The notebook reports the following final results. MAE and RMSE are in constant 2021 international dollars per person.

| Model / evaluation set | MAE | RMSE | R² |
|---|---:|---:|---:|
| Final XGBoost + log target — training and validation combined | 426.45 | 841.13 | 0.9989 |
| Final XGBoost + log target — test | 1,476.45 | 3,753.32 | 0.9782 |
| Persistence forecast: next year equals this year — test | 1,399.39 | 3,837.94 | 0.9772 |

Current-year GDP per capita accounts for roughly **97% of XGBoost's total feature gain**. The model largely captures persistence in GDP levels, and the other indicators add little predictive information in this setup. This does not establish causal relationships between those indicators and GDP.

Errors are much larger when predicting the 2020 downturn and the 2021 rebound than when predicting 2019. High-income economies also have larger absolute errors, while relative errors are more similar across GDP levels. A sample-weighting experiment produced only small, mixed changes, so the original model was retained.

The model exceeds the notebook's R² goal of 0.95 but misses its MAE goal of less than 1,000. Its performance is consistent with forecasting stable GDP levels, with limited improvement over the persistence benchmark and difficulty anticipating sudden economic shocks. The test evaluates later observations for the same countries, rather than generalization to entirely unseen countries.

## Repository Structure

```text
.
├── README.md
├── requirements.txt
├── data/
│   ├── Data.xlsx                    # Original World Bank indicator export
│   └── df_final.csv                 # Cleaned country-year dataset
├── notebooks/
│   └── DATA601_Project.ipynb        # Full analysis and saved outputs
├── report/
│   └── DATA601_Report.docx          # Written analysis
└── presentation/
    └── DATA601_Presentation.pptx    # Presentation of findings
```

## Running the Analysis

The notebook was developed in **Google Colab** and currently uses Google Drive mounts and personal Drive paths. The source data is included in this repository.

To run it in Colab:

1. Open [notebooks/DATA601_Project.ipynb](notebooks/DATA601_Project.ipynb) in Google Colab.
2. Copy `data/Data.xlsx` to your Google Drive and update every Drive read/write path in the notebook to your own location, including the paths used for intermediate CSV and joblib files.
3. Install `xgboost` if it is unavailable in the runtime, then run the cells in order. The notebook generates the processed dataset and intermediate modelling files before reloading them later.

For a local Python environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m pip install xgboost notebook
python -m notebook notebooks/DATA601_Project.ipynb
```

Before running locally, remove the `google.colab` imports and Drive mount calls, and replace all Drive paths with local input and output paths. From the notebook directory, the supplied inputs are `../data/Data.xlsx` and `../data/df_final.csv`; intermediate outputs can also be written to `../data/`. XGBoost and the Jupyter Notebook interface are installed separately above because they are not listed in `requirements.txt`.

The main libraries are pandas, NumPy, Matplotlib, seaborn, scikit-learn, XGBoost, openpyxl, and joblib. Saved notebook outputs can be viewed without rerunning the analysis.
