# recipesratings
# Predicting Recipe Ratings

**Name:** Obert Javier Sitiabudi  
**Course:** DSC 80  

## Introduction
In this project, I analyze a dataset of online recipes to understand what factors influence recipe ratings. 
The goal is to build a model that predicts the average rating of a recipe using features such as cooking time,
number of ingredients, and recipe tags.

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

