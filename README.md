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
A hypothesis test was conducted to determine whether recipes with more ingredients receive lower ratings.
A permutation test was used, and the results suggested no statistically significant difference.

## Framing a Prediction Problem
This is a regression problem where the response variable is the average recipe rating.
Model performance is evaluated using RMSE.

## Baseline Model
A baseline regression model using numeric features was trained to establish a performance benchmark.

## Final Model
The final model uses a Random Forest regressor with engineered features and tuned hyperparameters.
This model achieved lower RMSE than the baseline model.

## Fairness Analysis
A fairness analysis compared model RMSE between recipes with many ingredients and those with fewer ingredients.
A permutation test found no evidence that the model performs worse for high-ingredient recipes.

