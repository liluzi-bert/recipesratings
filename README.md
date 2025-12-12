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
The dataset was cleaned by removing invalid entries and standardizing numeric fields. 
Exploratory analysis showed that recipe ratings are generally high, while features like cooking time and calories
are right-skewed.

## Assessment of Missingness
Missingness was assessed for key variables such as calories and cooking time. 
Most missing values appeared to be non-systematic, so median imputation was used in later modeling steps.

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

