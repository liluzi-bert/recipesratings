# Predicting Recipe Ratings

**By:** Obert Javier Sitiabudi   

## Introduction
Food plays an important role in everyday life, and many people turn to online platforms to discover and rate new recipes. Ratings often reflect not only taste and convenience, but also growing awareness around nutrition and health. In particular, sugar consumption has become a major health concern, as high sugar intake is associated with conditions such as diabetes and obesity. As a result, users may subconsciously factor nutritional content into how they evaluate recipes. In this project, we investigate whether the amount of sugar in a recipe is related to how highly it is rated by users. Specifically, we examine whether recipes with a higher proportion of sugar relative to total calories tend to receive lower ratings. To explore this question, we analyze recipe and rating data collected from food.com, originally compiled for research on personalized recipe recommendation systems. By combining nutritional information with user ratings, we aim to better understand how health-related factors may influence online food preferences.


The primary dataset, recipes, contains 83,782 rows, each representing a unique recipe collected from food.com.

| Column | Description |
|------|------------|
| `name` | Recipe name |
| `id` | Unique recipe identifier |
| `minutes` | Total minutes required to prepare the recipe |
| `contributor_id` | User ID of the person who submitted the recipe |
| `submitted` | Date the recipe was submitted |
| `tags` | Food.com tags associated with the recipe |
| `nutrition` | Nutritional information including calories, sugar (PDV), fat, sodium, protein, and carbohydrates |
| `n_ingredients` | Number of ingredients used in the recipe |

The second dataset, interactions, contains 731,927 rows, where each row represents a user interaction with a specific recipe, including ratings and reviews. This dataset allows us to analyze how users rate recipes and link those ratings back to recipe attributes.
| Column | Description |
|------|------------|
| `user_id` | Unique identifier for the user |
| `recipe_id` | Identifier linking the interaction to a recipe |
| `date` | Date the interaction was recorded |
| `rating` | Rating given by the user (1–5 scale) |
| `review` | Optional text review left by the user |


## Data Cleaning and Exploratory Data Analysis

We cleaned and merged the raw recipe and interaction datasets to prepare them for analysis.  
The recipes dataset contains recipe-level information, while the interactions dataset contains individual user ratings for each recipe.
First, we performed a left merge between the recipes and interactions tables using the recipe identifier to associate user ratings with each recipe. Ratings with a value of 0 were treated as invalid and replaced with missing values. We then computed the average rating for each recipe by aggregating across all valid user ratings and merged this value back into the recipes dataset, producing a single recipe-level dataframe.
Next, nutritional information originally stored as string representations of lists was converted into numeric columns. These values were split into individual features such as calories, total fat, sugar, sodium, protein, saturated fat, and carbohydrates. This allowed us to directly analyze nutritional properties without repeated parsing.
The final cleaned dataset contains one row per recipe, includes the average user rating, and separates nutritional values into meaningful numeric columns. A preview of the cleaned dataset is shown below.

| name                                 |     id |   minutes | submitted   |   avg_rating |   calories |   sugar |
|:-------------------------------------|-------:|----------:|:------------|-------------:|-----------:|--------:|
| 1 brownies in the world    best ever | 333281 |        40 | 2008-10-27  |            4 |      138.4 |      50 |
| 1 in canada chocolate chip cookies   | 453467 |        45 | 2011-04-11  |            5 |      595.1 |     211 |
| 412 broccoli casserole               | 306168 |        40 | 2008-05-30  |            5 |      194.8 |       6 |
| millionaire pound cake               | 286009 |       120 | 2008-02-12  |            5 |      878.3 |     326 |
| 2000 meatloaf                        | 475785 |        90 | 2012-03-06  |            5 |      267   |      12 |

### Univariate Analysis
Most recipes contain under 500 calories, but the distribution is heavily right-skewed, indicating that a small number of recipes contain extremely high calorie counts.

<iframe src="assets/calories_dist.html" width="800" height="600" frameborder="0"></iframe>

### Bivariate Analysis
The scatter plot below shows the relationship between total fat and calories in recipes. There is a strong positive association, indicating that recipes with higher fat content 
tend to have substantially higher calorie counts. This relationship is expected given that fat is a calorie-dense macronutrient, and the trend is consistent across most recipes.

<iframe src="assets/calories_vs_fat.html" width="800" height="600" frameborder="0"></iframe>

### Interesting Aggregates
The table below compares average ratings between sugary and non-sugary recipes.

| is_sugary | average_rating |
|----------|----------------|
| False    | 4.52           |
| True     | 4.48           |

Sugary recipes receive slightly lower average ratings, though the difference is small.

## Assessment of Missingness
### NMAR Analysis

One column that may plausibly be NMAR is `avg_rating`. Recipes with very low ratings may be less likely to receive user feedback, as users who dislike a recipe may be less motivated to submit a rating. Because this missingness depends on the unobserved value itself (the rating), it cannot be fully explained using the observed data alone. Additional information about user behavior or submission patterns would be required to determine whether the missingness could instead be considered MAR.

### Missingness Dependency

We investigated whether the missingness of `avg_rating` depends on the number of steps (`n_steps`) using a permutation test. The test statistic was defined as the difference in the mean number of steps between recipes with missing ratings and those with observed ratings.

<iframe src="assets/avg_rating_missing_perm.html" width="800" height="600" frameborder="0"></iframe>

The observed statistic was compared against a null distribution generated by permuting the missingness indicator. The resulting p-value suggests that the missingness of `avg_rating` is not completely random and may depend on recipe complexity.

## Hypothesis Testing
Question: Do dessert recipes have a different average calorie content than main dish recipes?

Null Hypothesis (H₀): The average calories of dessert recipes is equal to the average calories of main dish recipes.

Alternative Hypothesis (H₁): The average calories of dessert recipes is different from the average calories of main dish recipes.

Test Statistic

The difference in mean calories between dessert recipes and main dish recipes (dessert − main dish).

Significance Level

α = 0.05

Method

We performed a permutation test by repeatedly shuffling recipe labels (dessert vs main dish) and recomputing the difference in mean calories to generate an empirical distribution under the null hypothesis.

Visualization

The plot below shows the empirical distribution of the test statistic under the null hypothesis.
The red vertical line indicates the observed difference in mean calories.

<iframe src="assets/permutation_calories_dessert_main.html" width="800" height="600" frameborder="0"></iframe>

Results

- Observed difference in mean calories: approximately 8.17

- p-value: 0.13

#### Conclusion

Since the p-value (0.13) is greater than the significance level of 0.05, we fail to reject the null hypothesis.
There is insufficient evidence to conclude that dessert recipes have a different average calorie content than main dish recipes.

## Framing a Prediction Problem
The goal of this project is to predict the average user rating of a recipe using information that is available at the time the recipe is posted. Recipe ratings summarize overall user satisfaction and provide a meaningful measure of recipe quality, making them a natural target for prediction.

This task is framed as a regression problem because the response variable, average rating, is continuous and takes values on a numerical scale rather than discrete categories.

The response variable used in this prediction problem is avg_rating, which represents the average rating (on a 1–5 scale) assigned by users to a recipe. This variable was chosen because it directly reflects how well a recipe is received by users and aligns with the goal of predicting recipe quality.

The features used for prediction are restricted to information that would be known prior to any user interaction, such as nutritional information (including calories, sugar, fat, and protein), cooking-related metadata (such as the number of steps and cooking time), and features derived from ingredients and recipe tags. User ratings, review counts, or any other post-publication information are excluded to prevent data leakage.

Model performance is evaluated using Root Mean Squared Error (RMSE). RMSE is appropriate for this regression task because it measures prediction error in the same units as the response variable and penalizes larger errors more heavily, which is important when predicting user ratings.

## Baseline Model
For our baseline model, we aim to establish a simple yet reasonable predictor for a recipe’s average rating (avg_rating) using features that are readily available at the time a recipe is submitted. We frame this as a regression problem, since the response variable is continuous.

The baseline model uses five features: minutes, n_steps, n_ingredients, and calories as quantitative variables, and first_tag as a categorical variable representing the primary recipe category. These features are directly related to recipe complexity, preparation effort, and nutritional content, which plausibly influence how users perceive and rate recipes.

To preprocess the data, we apply median imputation to numerical features to handle missing values and most-frequent imputation followed by one-hot encoding for the categorical feature. All preprocessing and modeling steps are implemented in a single sklearn pipeline to prevent data leakage.

We use Linear Regression as our baseline model due to its simplicity and interpretability. Model performance is evaluated using Root Mean Squared Error (RMSE) on a held-out test set (80/20 split). This baseline provides a reference point against which we can assess whether more complex models and additional features meaningfully improve predictive performance.

## Final Model
To improve upon the baseline model, we introduce additional feature engineering and use a more flexible modeling approach capable of capturing nonlinear relationships. The response variable remains avg_rating, and we continue to evaluate performance using RMSE on the same held-out test set to ensure a fair comparison.

In addition to the original numerical and categorical features, we apply transformations designed to better reflect the data-generating process. Cooking time (minutes) and calorie counts are right-skewed, so we apply a log transformation to reduce the influence of extreme values. We also engineer ratio-based features such as steps per ingredient and minutes per step, which serve as proxies for recipe complexity and preparation intensity. These features capture structure that raw counts alone cannot, and are more closely aligned with how users may experience a recipe.

For the final model, we use a Random Forest Regressor, which can naturally model nonlinear effects and interactions between features without requiring manual specification. To control model complexity and improve generalization, we tune key hyperparameters including the number of trees (n_estimators), maximum tree depth (max_depth), and minimum samples per leaf (min_samples_leaf).

Hyperparameter selection is performed using GridSearchCV with 5-fold cross-validation, optimizing for negative RMSE. The best-performing configuration uses 400 trees, a maximum depth of 10, and a minimum of 5 samples per leaf.

Compared to the baseline linear regression model, the final model achieves a lower RMSE, indicating improved predictive accuracy. This improvement suggests that nonlinear relationships and interactions between recipe characteristics play an important role in predicting average ratings, and that the engineered features better capture meaningful aspects of recipe complexity and user perception.

## Fairness Analysis
To evaluate the fairness of our final predictive model, we analyze whether its performance differs systematically across two meaningful subgroups of recipes. Specifically, we define Group X as dessert recipes and Group Y as non-dessert recipes, using the binary feature is_dessert. This grouping is appropriate because desserts often differ from other recipes in terms of sugar content, calories, and preparation style, which could plausibly affect how well the model predicts ratings for each group.

We evaluate fairness using Root Mean Squared Error (RMSE), the same metric used to assess overall model performance. RMSE is appropriate because our prediction task is regression-based and penalizes larger prediction errors more heavily, which aligns with our goal of detecting meaningful differences in model accuracy between groups.

Null Hypothesis (H₀): The model is equally fair across groups; the RMSE for dessert recipes is the same as the RMSE for non-dessert recipes.

Alternative Hypothesis (H₁): The model is not equally fair across groups; the RMSE differs between dessert and non-dessert recipes.

Our test statistic is the absolute difference in RMSE between the two groups. A larger value indicates greater disparity in predictive performance. We conduct a permutation test by repeatedly shuffling group labels and recomputing the RMSE difference to generate an empirical null distribution. We use a significance level of 0.05.

### Results and Conclusion

The resulting p-value was 0.912. Because 0.912 is much greater than 0.05, we fail to reject the null hypothesis. This means we do not have statistically significant evidence that our model performs differently for dessert versus non-dessert recipes under RMSE, so the model appears to be performing comparably across these groups based on our chosen fairness metric.
