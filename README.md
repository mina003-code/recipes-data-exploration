# Exploration of Health Trends in Recipes
By Mina K Wu

## Overview
This data science project aims to explore health trends in a recipe dataset taken from [food.com]([https://www.example.com](https://www.food.com/?ref=nav)). Specifically, This project is set to study whether people prefer healthier recipes and whether recipe health characteristics relate to ratings.

## Introduction
This project analyzes recipe and interaction data from Food.com to explore health-related patterns in recipes and ratings. The central question is whether people prefer “healthy” recipes over regular recipes, with a particular focus on dessert recipes and nutritional features. The dataset includes recipe-level information such as cooking time, tags, submission date, number of steps, ingredients, and nutrition values, along with user interaction data such as ratings and reviews. Relevant columns for this analysis include `tags`, `rating`, `review`, `avg_rating`, `submitted`, `minutes`, `n_steps`, and nutritional variables such as calories, sugar, protein, and saturated fat. The dataset has 234429 rows.

## Data Cleaning and Exploratory Data Analysis

The raw analysis merged the Food.com recipes dataset with the interactions dataset using recipe ID, producing a combined table with **234,429 rows**. Ratings of `0.0` were treated as missing because the lowest valid rating is `0.5`, so those rows likely reflect users who left no rating. An `avg_rating` column was created by grouping by recipe ID and averaging non-missing ratings. The `submitted` and `date` columns were converted to timestamps, and the `tags`, `steps`, and `ingredients` columns were converted from string representations into Python lists. The nested `nutrition` column was also unpacked into separate numeric columns: calories, total fat, sugar, sodium, protein, saturated fat, and carbohydrates. Finally, normalized features such as `sugar / calories`, `protein / calories`, and `saturated fat / calories` were added to compare nutrition levels across recipes more fairly.

## Univariate Analysis

The first univariate plot shows the distribution of recipe submissions over time. The histogram indicates that recipe submissions are heavily concentrated in the earlier years of the dataset, especially around 2008, and then decline over time. The second univariate plot shows the distribution of ratings. Most observed ratings are concentrated near 5, which suggests a strong positive skew in user evaluations. These patterns matter because they show that both time and ratings are highly imbalanced, which affects how trends should be interpreted.

## Bivariate Analysis

One bivariate plot compares average rating against submission time. The scatter plot suggests that ratings stay mostly high across the full time span, with most points between roughly 4 and 5, so there is not an obvious strong time trend in the average rating. Another scatter plot compares cooking time with average rating. That plot shows a dense concentration of recipes at shorter cooking times, while ratings again remain mostly high across the range. Together, these plots suggest that neither submission year nor cooking time alone strongly explains average rating.

## Interesting Aggregates

While plotting data from the recipes' nutrition facts (such as sugar, protein, and calories), I noticed that there are certain outliers with very high values. This could be explained by the fact that recipes on the website do not all have the same serving size, so certain recipes that are meant to serve many would have higher-than-usual values for their nutrition facts. This discrepancy is compensated by normalizing the macronutrient values by the calories of the recipe. Scatter plot of `sugar / calories` against `avg_rating`. The visual patterns of the sugar-versus-rating plot shows that high ratings occur across a broad range of sugar content. This is useful because it hints that recipe popularity may not depend only on sugar level.

## Assessment of Missingness

The dataset has several columns with non-trivial missing values, and the one chosen to be further investigated is the `rating` column. To study whether missing ratings depend on other observed variables, a permutation test on review length and cooking time is performed. 

First, for the permutation test on cooking time, the null hypothesis states that the average cooking time of the recipe is the same for rows with missing and non-missing ratings, while the alternative states that entries with missing ratings have different cooking times. The reported p-value is **0.126000**, which does not show strong evidence of dependence.

Second, for the permutation test on review length, the null hypothesis states that the average review word count is the same for rows with missing and non-missing ratings, while the alternative states that entries with missing ratings have lower review length. The reported p-value is effectively **0.000000**, providing strong evidence that rating missingness depends on review length. This allows the conclusion that the missingness in ratings is likely **MNAR**, becuase the users who do not feel strongly enough to write longer reviews may also be less likely to leave ratings.

## Hypothesis Testing

To gain a better understanding of the dataset, a hypothesis test is conducted to evaluate whether there is a true distinction between dessert recipes labeled as healthy and those that are not. The question asked is whether dessert recipes with health-related tags have lower sugar content per calorie than dessert recipes without those tags. To build this analysis, I first define a list of dessert-related tags and a separate list of health-related tags such as `healthy`, `low-fat`, `low-calorie`, `low-carb`, and `high-protein`. Recipes are filtered to dessert entries, then each dessert is classified as healthy or not based on whether its tag list contains one of the health-related tags. This produces a dessert-only dataset with **55,617 rows**.

A KDE plot shows that the two groups have somewhat different distribution shapes, I chose Kolmogorov–Smirnov test statistic instead of the diffference in group means. The null hypothesis is that healthy-tagged and non-healthy-tagged dessert recipes have the same sugar-per-calorie distribution. The reported p-value is **0.000000**, so we reject the null hypothesis and conclude that the sugar-content distributions differ between the two groups.

## Framing a Prediction Problem

The prediction problem is to **predict the average rating of a recipe**, which makes this a **regression** task. The response variable is `avg_rating`. This target is appropriate because average recipe rating is a direct measure of how positively users evaluate a recipe overall, and it lets the analysis connect recipe health characteristics to user preference in a quantitative way. The model sets to use features such as health-related tags, normalized sugar content, cooking time, number of steps, submission year, and calories. Model quality is evaluated primarily with **RMSE**, which is appropriate for continuous prediction because it measures typical prediction error in rating units. 

## Baseline Model

The baseline model uses two features: `tags` and `sugar / calories`. The `tags` feature is transformed into a Boolean indicator representing whether a recipe is classified as healthy, while `sugar / calories` is passed through as a numeric feature. A linear regression model is then fit through a preprocessing pipeline. On the full cleaned prediction dataset, the baseline model reports **R² = 0.00017079605964920308** and **RMSE = 0.4929099469859542**. In 5-fold cross-validation, the mean CV RMSE is **0.4928866410110781**. These results indicate that the baseline model does not have a strong explanatory power and only modest predictive ability.

## Final Model

The final model adds several features beyond the baseline: `minutes`, `n_steps`, `submitted`, and `calories (#)`, while still using `tags` and `sugar / calories`. The preprocessing pipeline applies a custom healthy-tag indicator to `tags`, polynomial and standardized transformations to numeric variables, and one-hot encoding to the submission year extracted from `submitted`. A linear regression model is again used, and `GridSearchCV` tunes the polynomial degree over values 1 through 4 using 5-fold cross-validation and RMSE as the scoring metric. The best degree is **1**, and the best cross-validated RMSE is **0.49261412906840674**. This is only a small improvement over the baseline model, but it still shows that a somewhat richer feature set performs slightly better.

## Fairness Analysis

The fairness analysis checks whether the final model has similar RMSE for recipes with higher sugar content and lower sugar content. A group indicator `high_sugar` is defined, which states whether `sugar / calories >= 0.5`. It then computes group-specific RMSE values using the tuned final model and takes the absolute difference between those RMSE values. The observed difference is **0.009373531776350474**. A permutation test with 500 repetitions gives a p-value of **0.238**, so the notebook fails to reject the null hypothesis. Based on this result, we conclude that the final model is fair with respect to this particular sugar-content grouping.
