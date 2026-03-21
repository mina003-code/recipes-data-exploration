# Exploration of Health Trends in Recipes
By Mina K Wu

## Overview
This data science project aims to explore health trends in a recipe dataset taken from [food.com]([https://www.example.com](https://www.food.com/?ref=nav)). Specifically, This project is set to tudy whether people prefer healthier recipes and whether recipe health characteristics relate to ratings.

## Introduction
This project analyzes recipe and interaction data from Food.com to explore health-related patterns in recipes and ratings. The central question is whether people prefer “healthy” recipes over regular recipes, with a particular focus on dessert recipes and nutritional features. The dataset includes recipe-level information such as cooking time, tags, submission date, number of steps, ingredients, and nutrition values, along with user interaction data such as ratings and reviews. Relevant columns for this analysis include `tags`, `rating`, `review`, `avg_rating`, `submitted`, `minutes`, `n_steps`, and nutritional variables such as calories, sugar, protein, and saturated fat. The dataset has 234429 rows.

## Data Cleaning and Exploratory Data Analysis

The raw analysis merged the Food.com recipes dataset with the interactions dataset using recipe ID, producing a combined table with **234,429 rows**. Ratings of `0.0` were treated as missing because the notebook notes that the lowest valid rating is `0.5`, so those rows likely reflect users who left no rating. An `avg_rating` column was created by grouping by recipe ID and averaging non-missing ratings. The `submitted` and `date` columns were converted to timestamps, and the `tags`, `steps`, and `ingredients` columns were converted from string representations into Python lists. The nested `nutrition` column was also unpacked into separate numeric columns: calories, total fat, sugar, sodium, protein, saturated fat, and carbohydrates. Finally, normalized features such as `sugar / calories`, `protein / calories`, and `saturated fat / calories` were added to compare nutrition levels across recipes more fairly.

## Univariate Analysis

The first univariate plot shows the distribution of recipe submissions over time. The histogram indicates that recipe submissions are heavily concentrated in the earlier years of the dataset, especially around 2008, and then decline over time. The second univariate plot shows the distribution of ratings. Most observed ratings are concentrated near 5, which suggests a strong positive skew in user evaluations. These patterns matter because they show that both time and ratings are highly imbalanced, which affects how trends should be interpreted.

## Bivariate Analysis

One bivariate plot compares average rating against submission time. The notebook’s scatter plot suggests that ratings stay mostly high across the full time span, with most points between roughly 4 and 5, so there is not an obvious strong time trend in average rating. Another scatter plot compares cooking time with average rating. That plot shows a dense concentration of recipes at shorter cooking times, while ratings again remain mostly high across the range. Together, these plots suggest that neither submission year nor cooking time alone strongly explains average rating.

## Interesting Aggregates

While plotting data from the recipes' nutrition facts (such as sugar, protein, and calories), I noticed that there are certain outliers with very high values. This could be explained by the fact that recipes on the website do not all have the same serving size, so certain recipes that are meant to serve many would have higher-than-usual values for their nutrition facts. This discrepancy is compensated by normalizing the macronutrient values by the calories of the recipe. Scatter plot of `sugar / calories` against `avg_rating`. The visual patterns of the sugar-versus-rating plot shows that high ratings occur across a broad range of sugar content. This is useful because it hints that recipe popularity may not depend only on sugar level.

