# Part B: Business Case Analysis  
## Scenario: Promotion Effectiveness at a Fashion Retail Chain  

---

## B1(a) — Problem Formulation

I’d frame this as a **supervised learning regression problem**.  

- **Target variable:** Number of items sold during the promotion period (Sales Volume).  
- **Candidate input features:**  
  - Store attributes: size, monthly footfall, location type (urban/semi‑urban/rural).  
  - Market context: local competition density.  
  - Customer demographics: age mix, income levels, loyalty membership share.  
  - Promotion type: one of the five categories (Flat Discount, BOGO, Free Gift, Category Offer, Loyalty Points Bonus).  
  - Historical performance: past promotion outcomes, seasonal trends.  

**Justification**
Why regression? Because sales volume is continuous. Later, we can convert predictions into a decision rule (pick the promotion with the highest predicted sales). That’s more flexible than forcing it into a classification upfront.  

### Regression vs Classification Comparison

| Aspect | Regression Approach | Classification Approach |  
|--------|---------------------|--------------------------|  
| **Target** | Predict continuous sales volume (items sold) | Predict categorical label: “best promotion” |  
| **Flexibility** | Can compare predicted volumes across all promotions | Directly outputs one promotion choice |  
| **Interpretability** | Shows expected uplift in items sold per promotion | Easier to explain to non‑technical stakeholders |  
| **Business Use** | Allows ranking and scenario testing (e.g., “what if we run BOGO?”) | Provides a single recommendation without magnitude |  
| **Drawback** | Requires post‑processing to select promotion | Loses detail on how much better one promotion is |  

**Conclusion:** Regression is more informative because it quantifies impact, but classification can be derived later if needed.  

---

## B1(b) — Target Variable Choice

The company currently tracks **total sales revenue**, but that’s not the best fit here. Revenue can be skewed by product mix — selling one expensive jacket doesn’t necessarily mean the promotion was effective.  

**Items sold (sales volume)** is a cleaner measure of customer response. It tells us whether promotions are actually driving engagement and purchases, regardless of price differences.  

This illustrates a broader principle: in ML projects, the target variable should directly reflect the business objective. If you optimise for the wrong metric, you risk solving the wrong problem.  

---

## B1(c) — Modelling Strategy

A single global model across all 50 stores would ignore the fact that **context matters**. Urban stores with high competition might respond differently to BOGO than rural stores with low footfall.  

A better approach is **segmented or hierarchical modelling**:  
- Build separate models for clusters of stores (e.g., urban vs rural, or high vs low footfall).  
- Or use a multi‑task learning setup where stores share some parameters but have store‑specific prediction layers.  

This way, we capture both common patterns and local differences, leading to more accurate and actionable recommendations.  

### Segmented Modelling Example

Imagine three store clusters:  

- **Urban, high competition, high footfall**  
  - Promotions like BOGO may perform better due to dense customer traffic.  
- **Semi‑urban, moderate competition, medium footfall**  
  - Category‑specific offers might resonate with targeted demographics.  
- **Rural, low competition, low footfall**  
  - Flat discounts or loyalty bonuses may be more effective in encouraging repeat purchases.  

This segmentation ensures the model reflects real‑world differences in customer behaviour across store contexts.  

### Concluding Summary  
In Part B1, the analysis establishes promotion effectiveness as a **regression problem**, with items sold as the target variable. Regression provides flexibility to compare predicted volumes across promotions, while classification can be derived later if needed. The choice of sales volume over revenue ensures alignment with the true business objective — customer response. Finally, a **segmented modelling strategy** is recommended, recognising that store context (urban, semi‑urban, rural) significantly influences promotion outcomes. Together, these decisions create a modelling framework that is both technically sound and business‑aligned.


#########################################################################################################
 
## B2. Data and EDA Strategy

---

## B2(a) — Joining Tables and Dataset Grain

The raw data comes in four tables:  
- **Transactions:** individual purchase records with product, store, date, and promotion ID.  
- **Store attributes:** static information such as store size, location type, and competition density.  
- **Promotion details:** promotion type, discount level, and start/end dates.  
- **Calendar:** flags for weekends, festivals, and seasonality.  

**Joining strategy:**  
- Join transactions with promotion details on `promotion_id`.  
- Join with store attributes on `store_id`.  
- Join with calendar on `date`.  

**Grain of final dataset:**  
- One row = **store × promotion × month** (aggregated).  

**Aggregations before modelling:**  
- Total items sold (target variable).  
- Average basket size, revenue, and discount percentage.  
- Footfall aggregated monthly.  
- Flags for proportion of weekends/festival days in the month.  
- Customer mix proportions (e.g., loyalty members share).  

This ensures the dataset is at the right level for predicting promotion effectiveness per store per month.

---

## B2(b) — EDA Strategy

Before modelling, I would perform exploratory data analysis to understand distributions, relationships, and potential issues.  

**Initial checks:**  
- **Missing values:** Identify gaps in demographics or store attributes; impute or flag them.  
- **Outliers:** Detect unusually high/low sales or footfall; decide whether to cap or transform.  
- **Scaling:** Standardise or normalise numeric features (e.g., footfall, store size) to ensure fair comparison and stable model training.  
- **Multicollinearity:** Use correlation analysis to avoid redundant predictors.  

**Key analyses and charts:**  
1. **Sales volume distribution by promotion type (bar chart):**  
   - Look for which promotions generally drive higher volumes.  
   - Influences feature engineering (e.g., dummy variables for promotion type).

2. **Footfall vs items sold (scatter plot):**  
   - Check correlation between store traffic and sales.  
   - Guides whether to normalise sales by footfall or include interaction terms.

3. **Seasonality and festival impact (line chart over time):**  
   - Identify peaks during festivals or weekends.  
   - Suggests adding calendar flags as predictive features.

4. **Store segmentation analysis (boxplots by urban/semi‑urban/rural):**  
   - Compare promotion effectiveness across store contexts.  
   - Supports decision to build segmented models rather than one global model.

5. **Feature correlation or promotion × store location heat map:**  
   - Examine relationships among numeric features (footfall, store size, competition density).  
   - Alternatively, visualise average items sold by promotion type across store clusters.  
   - Helps identify multicollinearity, feature interactions, and context‑specific promotion strengths.  
   - Guides feature selection and warns against redundant predictors.

Findings directly shape feature engineering (e.g., log‑transform skewed variables, impute missing demographics, scale continuous features, cluster stores).

---

## B2(c) — Handling Promotion Imbalance

Observation: 80% of transactions occurred without any promotion.  

**Impact:**  
- The model may learn a bias toward “no promotion” as the default, underestimating the effect of actual promotions.  
- Could reduce sensitivity to promotion features and weaken recommendations.

**Steps to address imbalance:**  
- Re‑sample the data (e.g., downsample non‑promotion transactions or upsample promotion ones).  
- Use stratified sampling to balance training sets.  
- Introduce weighting in the loss function so promotion cases carry more importance.  
- Alternatively, aggregate at the store‑month level (rather than transaction level), which naturally balances exposure to promotions.

This ensures the model captures true promotion effects rather than defaulting to “no promotion.”  

---

### B2 Concluding Summary  
In Part B2, the data preparation strategy ensures a clean, aggregated dataset at the **store × promotion × month** grain, capturing relevant attributes and calendar effects. The EDA plan is comprehensive: initial checks (missing values, outliers, scaling, multicollinearity) followed by five analyses covering distributions, relationships, seasonality, segmentation, and feature interactions. These findings directly guide feature engineering and model design. Addressing the imbalance between promotion and non‑promotion transactions further strengthens reliability. Overall, the B2 strategy ensures the modelling dataset is robust, interpretable, and ready to support the segmented regression approach outlined in B1.

###########################################################################################################
 
## B3. Model Evaluation and Deployment

---

## B3(a) — Train-Test Split and Metrics

We have monthly store-level data spanning three years across 50 stores.  

**Train-test split strategy:**  
- Use  **time-series split** rather than random.  
- Train on the first 2.5 years (e.g., Jan 2021 – Jun 2023) and test on the last 6 months (Jul – Dec 2023).  
- Random split is inappropriate because it would mix past and future data, leading to **data leakage** and unrealistic performance estimates.  

**Evaluation metrics:**  
- **RMSE (Root Mean Squared Error):** Measures average prediction error in items sold. Interpreted as “on average, predictions are off by X units.”  
- **MAPE (Mean Absolute Percentage Error):** Shows relative error as a percentage of actual sales. Useful for comparing across stores of different sizes.  
- **R² (Coefficient of Determination):** Indicates how much variance in sales is explained by the model. Higher values mean better explanatory power.  
- **Business interpretation:** RMSE tells us the scale of error, MAPE highlights proportional accuracy across stores, and R² shows overall model fit. Together, they provide a balanced view of predictive performance.

---

## B3(b) — Feature Importance and Communication

Example: The model recommends **Loyalty Points Bonus** for Store 12 in December and **Flat Discount** for Store 12 in March.  

**Investigating feature importance:**  
- Use techniques like **SHAP values** or permutation importance to see which features drove the prediction.  
- In December, features like **festival flag, high footfall, and loyalty membership share** may dominate, making Loyalty Points Bonus more effective.  
- In March, features like **competition density, lower seasonal demand, and store size** may shift importance, favouring Flat Discounts.  

**Communicating to marketing team:**  
- Present a simple chart showing top 3–5 features influencing each month’s recommendation.  
- Explain in business terms: “In December, loyalty engagement mattered most; in March, price sensitivity mattered more.”  
- This builds trust by showing the model’s reasoning is aligned with observable store and calendar dynamics.

---

## B3(c) — Deployment Process

The model must generate recommendations monthly without retraining each time.  

**End-to-end deployment steps:**  
1. **Save the trained model:**  
   - Export as a serialized object (e.g., `pickle` or `joblib` in Python).  
   - Store in a secure model registry with version control.  

2. **Prepare new monthly data:**  
   - Aggregate transactions into the same **store × promotion × month** grain.  
   - Apply the same preprocessing pipeline (scaling, encoding, feature engineering).  
   - Feed the processed data into the saved model.  

3. **Generate recommendations:**  
   - Model outputs predicted items sold for each promotion per store.  
   - Select the promotion with the highest predicted sales as the recommendation.  

4. **Monitoring and retraining triggers:**  
   - Track metrics like RMSE and MAPE on new months.  
   - Monitor drift in feature distributions (e.g., footfall patterns, promotion mix).  
   - Retrain if performance degrades beyond a threshold or if business context changes (new promotion types, major shifts in customer behaviour).  

This ensures the model remains reliable, scalable, and business‑aligned over time.

### B3 Concluding Summary  
In Part B3, the evaluation strategy uses a **time‑based train‑test split** to respect chronology, avoiding leakage that random or stratified splits would cause. Metrics such as RMSE, MAPE, and R² provide complementary insights into prediction accuracy, proportional error, and explanatory power. Model interpretability is ensured through **feature importance techniques (e.g., SHAP values)**, allowing clear communication of why recommendations differ across months and stores. Finally, the deployment plan outlines saving the trained model, preparing monthly data consistently, and monitoring both performance metrics and feature drift to trigger retraining when needed. Together, these steps ensure the model is not only accurate but also **transparent, scalable, and business‑aligned** in its ongoing use.

############################################################################################################

### Part B Overall Summary  
Across Parts B1, B2, and B3, the business case analysis builds a coherent modelling and deployment strategy for promotion effectiveness. In B1, the problem is framed as a **regression task** with items sold as the target, ensuring alignment with customer response rather than revenue alone, and adopting a **segmented approach** to respect store context. In B2, the data preparation and EDA strategy ensures a robust dataset at the **store × promotion × month** grain, with systematic checks (missing values, outliers, scaling, multicollinearity) followed by comprehensive analyses of distributions, seasonality, segmentation, and feature interactions. These findings directly guide feature engineering and strengthen model reliability. In B3, evaluation is conducted using a **time‑based split** with metrics such as RMSE, MAPE, and R², while interpretability is achieved through **feature importance techniques (e.g., SHAP values)** to explain recommendations clearly to stakeholders. Finally, the deployment plan outlines saving the trained model, preparing monthly data consistently, and monitoring both performance and drift to trigger retraining when needed. Together, these steps ensure the solution is not only technically sound but also **transparent, scalable, and business‑aligned**, providing actionable promotion recommendations across all stores.
