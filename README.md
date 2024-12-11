# Invetigation on Cooking Minutes
Author: Mingrui Zhuang
## Introduction
Cooking is a universal activity that connects people across cultures, lifestyles, and skill levels, yet the time it takes to prepare meals can often be a challenge. However, **what features in recipes affect the cooking times?** Understanding this question can help both beginner and experienced cooks better plan their meals, save time, and make informed decisions in the kitchen. By analyzing recipe elements such as the number of steps, ingredients, and tags like "baking" or "slow-cooked," we can uncover patterns that influence cooking durations. This insight not only allows readers to select recipes that fit their schedules but also deepens their knowledge of culinary techniques and make the experience more enjoyable.

We choose two dataset consisting of [recipes and ratings posted since 2008](ttps://food.com/) to analyze this question. 

The first dataset, Recipes Dataset, contains 83782 rows, where each row is a unique recipes, and 10 columns including the following information:

| Column          | Description                       |
|-----------------|-----------------------------------|
| `name`          | Recipe name                       |
| `id`            | Recipe ID                         |
| `minutes`       | Minutes to prepare recipe         |
| `contributor_id`| User ID who submitted this recipe |
| `submitted`     | Date recipe was submitted         |
| `tags`          | Food.com tags for recipe          |
| `nutrition`     | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for "percentage of daily value" |
| `n_steps`       | Number of steps in recipe         |
| `steps`         | Text for recipe steps, in order   |
| `description`   | User-provided description         |

---

The second dataset, Ratings Dataset, contains 731927 rows, where each row is a review from the user, and 5 columns including the following information:

| Column          | Description                   |
|-----------------|-------------------------------|
| `user_id`       | User ID                       |
| `recipe_id`     | Recipe ID                     |
| `date`          | Date of interaction           |
| `rating`        | Rating given                  |
| `review`        | Review text                   |


## Data Cleaning and Exploratory Data Analysis
### Data Cleaning
To prepare the dataset for meaningful and accurate analysis, we take several steps to address inconsistencies, missing values, and outliers.

1. Performed a **left merge** between the recipes and interactions datasets. In this way, we preserved all recipes while including interaction data where available, enabling analysis on recipes with ratings.
2. Replaced all ratings of 0 with `np.nan`. Since the rating is on a scale from 1 to 5, 0 ratings likely stem from the absence of user reviews. Replacing the 0 values will prevent skewing of averages due to artificial zero values, ensuring accurate computation of the avg_rating column.
3. Created a new column, `'avg_rating'`, by calculating the mean rating for each recipe using the name column.
We can assumes recipes with higher ratings are more positively received, which enabled comparisons between recipes based on user feedback.
4. Converted the `'submitted'` and `'date'` columns to datetime format, which are more meaningful for time-based analyses.
5. Converted the `'tags'` and `'ingredients'` columns from string to lists, which enabled further analysis.
6. Filtered out extreme values based on the following thresholds:
	- `'minutes'` ≤ 720 (12 hours)
	- `'n_ingredients'` ≤ 20
	- `'n_steps'` ≤ 30
	Recipes with excessively high cooking times, steps, or ingredients likely represent data entry errors or rare anomalies not reflective of typical recipes.

Our cleaned dataFrame contains 227834 rows and 18 columns. Here are the first 5 rows of recipes of our cleaned dataframe. Scroll right to view all the columns.

| name             |     id |   minutes |   contributor_id | submitted            | tags                                     | nutrition           |   n_steps | steps                       | description                               | ingredients                                      |   n_ingredients |   user_id |   recipe_id | date    |   rating | review                                  |   avg_rating |
|:---------------------------------|-------:|-----:|--------:|:-----------------|:---------------------------------------------|:-------------------|-----:|:------------------------------|:-----------------------------------|:---------------------------------------------------------|----:|-------:|------------:|:--------------------|--------:|:--------------------------------------|------:|
| 1 brownies in the world best ever  | 333281 | 40 | 985201 | 2008-10-27 00:00:00 | ['60-minutes-or-less', 'time-to-make',...] | [138.4, 10.0,...]   | 10 | ['heat the oven to 350f ...']    | these are the most; chocolatey, moist,...   | ['bittersweet chocolate', 'unsalted butter',...] | 9 | 386585  |      333281 | 2008-11-19 00:00:00 |    4 | These were pretty good, but took... |    4 |
| 1 in canada chocolate chip cookies | 453467 | 45 | 1848091 | 2011-04-11 00:00:00 | ['60-minutes-or-less', 'time-to-make',...] | [595.1, 46.0,...] |  12 | ['pre-heat oven the 350 degrees ...'] | this is the recipe that we...  | ['white sugar', 'brown sugar',...]                    |  11 | 424680  |      453467 | 2012-01-26 00:00:00 |    5 | Originally I was gonna cut the ...     |     5 |
| 412 broccoli casserole     | 306168 |  40 |     50969 | 2008-05-30 00:00:00 | ['60-minutes-or-less', 'time-to-make',...] | [194.8, 20.0,...] |   6 | ['preheat oven to 350 degrees, ...']   | since there are already 411 recipes ... | ['frozen broccoli cuts', 'cream of chicken soup',...] | 9 |  29782   |   306168 | 2008-12-31 00:00:00 |    5 | This was one of the best broccoli...  |      5 |                                        |              |
| 412 broccoli casserole     | 306168 |  40 |     50969 | 2008-05-30 00:00:00 | ['60-minutes-or-less', 'time-to-make',...] | [194.8, 20.0,...] |   6 | ['preheat oven to 350 degrees, ...']   | since there are already 411 recipes ... | ['frozen broccoli cuts', 'cream of chicken soup',...] | 9 | 1.19628e+06 | 306168 | 2009-04-13 00:00:00 |   5 | I made this for my son's first birthday...  |      5 |
| 412 broccoli casserole     | 306168 |  40 |     50969 | 2008-05-30 00:00:00 | ['60-minutes-or-less', 'time-to-make',...] | [194.8, 20.0,...] |   6 | ['preheat oven to 350 degrees, ...']   | since there are already 411 recipes ... | ['frozen broccoli cuts', 'cream of chicken soup',...] | 9 | 768828     |  306168 | 2013-08-02 00:00:00 |    5 | Loved this.  Be sure to completely...  |      5 |

### Univariate Analysis
We first explored the distribution of Number of Steps (`'n_steps'`) by histogram:

<iframe
  src="assets/Univariate_1.html"
  width="650"
  height="450"
  frameborder="0"
></iframe>

   The majority of recipes have around 4–10 steps, suggesting that most recipes are designed to be relatively straightforward.

Then we explored the distribution of of Minutes (`'minutes'`) by Box Plot :

<iframe
  src="assets/Univariate_2.html"
  width="650"
  height="450"
  frameborder="0"
></iframe>

   Most recipes have cooking times clustered within a short range (between 0–125 minutes). There are significant outliers extending beyond 200 minutes, which represent recipes requiring much longer cooking times.

### Bivariate Analysis
We further investigated the two variables that are most relevant to cooking minutes: number of steps (`'n_steps'`), and numbers of ingredients (`'n_ingredients'`).

<iframe
  src="assets/Bivariate_1.html"
  width="650"
  height="450"
  frameborder="0"
></iframe>

   This scatter plot shows the relationship between the square root of the number of steps (`'test_n_steps'`) and the median time (`'minutes'`). By applying the square root to the number of steps, we can compress the exponential-like growthsee to a more linear trend. This trend indicates a **positive correlation**, where more steps generally correspond to longer median times, which suggests that recipes with more steps tend to take longer.

<iframe
  src="assets/Bivariate_2.html"
  width="650"
  height="450"
  frameborder="0"
></iframe>

   This bar chart illustrates the average time required (`'minutes'`) for recipes with varying numbers of ingredients (`'n_ingredients'`), grouped by whether the recipe has a rating of 5. Recipes with higher numbers of ingredients tend to take longer, and those rated 5 (`'true'`) often require more time compared to lower-rated recipes. This highlights that more complex recipes (higher ingredients) often lead to higher ratings but demand more time.

### Interesting Aggregates
We explored the trends of minutes over time by comparing total cooking minutes across publish_year and rating_year.

|   publish_year |    2008 |    2009 |    2010 |    2011 |    2012 |    2013 |    2014 |    2015 |    2016 |    2017 |    2018 |
|---------------:|--------:|--------:|--------:|--------:|--------:|--------:|--------:|--------:|--------:|--------:|--------:|
|           2008 | 1708.54 | 1367.14 | 687.695 | 498.815 | 329.959 | 286.945 | 189.589 | 150.988 | 125.52  | 184.136 | 128.45  |
|           2009 |    0    | 1278.34 | 734.573 | 416.251 | 262.391 | 208.034 | 133.694 | 111.566 |  82.412 | 123.528 | 106.863 |
|           2010 |    0    |    0    | 586.221 | 350.009 | 183.447 | 146.589 |  85.504 |  57.539 |  54.209 |  72.995 |  57.079 |
|           2011 |    0    |    0    |   0     | 335.204 | 205.27  | 125.583 |  79.444 |  52.19  |  38.494 |  49.173 |  41.344 |
|           2012 |    0    |    0    |   0     |   0     | 296.768 | 154.669 |  75.685 |  51.648 |  44.519 |  57.831 |  36.053 |
|           2013 |    0    |    0    |   0     |   0     |   0     | 280.107 | 101.323 |  54.089 |  31.84  |  45.612 |  34.229 |
|           2014 |    0    |    0    |   0     |   0     |   0     |   0     | 102.368 |  20.136 |   9.334 |  16.898 |  12.918 |
|           2015 |    0    |    0    |   0     |   0     |   0     |   0     |   0     |  19.484 |   4     |   5.179 |   4.121 |
|           2016 |    0    |    0    |   0     |   0     |   0     |   0     |   0     |   0     |   6.893 |   4.885 |   3.753 |
|           2017 |    0    |    0    |   0     |   0     |   0     |   0     |   0     |   0     |   0     |  17.603 |  13.669 |
|           2018 |    0    |    0    |   0     |   0     |   0     |   0     |   0     |   0     |   0     |   0     |  18.601 |

Form the dataframe, we find two obvious patterns.
- Content published in 2008 has very high engagement (1708.54k total minutes in 2008), with a gradual decline in subsequent years. This suggests that content was highly engaging in the year of release but experiences decreasing attention over time.
- Content from 2015–2018 sees relatively lower overall engagement, but earlier contents have noticeable engagement continues into much later years, which suggest a saturation or a shift in user behavior toward older, rediscovered content.

## Assessment of Missingness
### NMAR Analysis
We believe that the missingness of the `'review'` column can be NMAR, which means that the missing review value itself is a key factor influencing whether it is missing. Users who were **dissatisfied or indifferent** specifically chose not to leave a review, because they felt unmotivated to review or had some other personal reason (e.g., they didn’t want to type letters at all). This would suggest that the missingness is dependent on the unobserved missing value (the review itself). Additional data can be collected on user engagement data, where information on how often users interact with the platform could help determine if certain users are more likely to leave reviews. For example, user's frequency of interactions, recipe views, and time spent on the platform may give us some missingness pattern about review.

### Missingness Dependency
Now we focus on the missingness of `description` in the dataframe by testing the dependency of its missingness on `'minutes'` and `'n_ingredients'`.

**H<sub>0</sub>**: The distribution of preparation time (`'minutes'`) is independent of whether the `'description'` is missing.

**H<sub>a</sub>**: The distribution of preparation time (`'minutes'`) is dependent on whether the `'description'` is missing.

Test Statistic:  Kolmogorov-Smirnov statistic between group of minutes with and without missing description

Significance Level: 0.05

We looked at the distributions of `minutes` separately for missing or not, and check to see if they are similar.

<iframe
  src="assets/Missing_1.html"
  width="650"
  height="450"
  frameborder="0"
></iframe>

The observed KS statistic is 0.0796, with p-value 0.626. This is a high p-value, indicating that the observed difference is not statistically significant, so we **Fail to Reject** the null hypothesis. There is no evidence to suggest that the distributions of minutes differ significantly between the two groups.

<iframe
  src="assets/Missing_2.html"
  width="820"
  height="620"
  frameborder="0"
></iframe>

Then we took a KS Permutation Test with n_ingredients:

**H<sub>0</sub>**: The distribution of number of ingredients (`'n_ingredients'`) is independent of whether the `'description'` is missing.

**H<sub>a</sub>**: The distribution of number of ingredients (`'n_ingredients'`) is dependent on whether the `'description'` is missing.

**Test Statistic**:  Kolmogorov-Smirnov statistic between group of n_ingredients with and without missing description

**Significance Level**: 0.05

Here is a distribution of `n_ingredients` conditional on missingness of `description`:

<iframe
  src="assets/Missing_3.html"
  width="500"
  height="300"
  frameborder="0"
></iframe>

The observed KS statistic is 0.1641, with p-value  0.012. This is a low p-value below the common significance threshold (0.05), meaning that only 1.2% of the KS statistics from the permuted datasets are greater than or equal to the observed KS statistic. We **reject** the null hypothesis, and conclude that the missingness in the description column is not random with respect to n_ingredients. There maybe a possible systematic relationship between the number of ingredients in a recipe and whether its description is missing.

<iframe
  src="assets/Missing_4.html"
  width="820"
  height="620"
  frameborder="0"
></iframe>

## Hypothesis Testing
Since our project is centered around what features affect the cooking time, our test question focuses on whether a linear relationship exists between n_steps and minutes. 

**H<sub>0</sub>**: There is no significant linear relationship between the number of steps (`'n_steps'`) and cooking time (`'minutes'`).

**H<sub>a</sub>**: There is a significant linear relationship between the number of steps (`'n_steps'`) and cooking time (`'minutes'`).

**Test Statistic**:  Pearson’s Correlation Coefficient (r)

**Significance Level**: 0.01

We choose a permutation test because it's non-parametric and doesn't make assumptions about the distribution of the data; We choose a smaller significance level (α=0.01) to decrease the probability of making a Type I error, setting a stricter threshold for rejecting the null; We choose the Pearson's correlation because it measures the strength and direction of the linear relationship between two continuous variables. 

After a permutation test with 10,000 simulations, the observed correlation is 0.1496, with a p_value of 0.0. The correlation indicates a **weak positive linear relationship** between the number of steps in a recipe and its cooking time. We conclude that there is a statistically significant relationship between n_steps and minutes. Recipes with more steps tend to have longer cooking times, even though the relationship is weak.

<iframe
  src="assets/Hypothesis_1.html"
  width="820"
  height="620"
  frameborder="0"
></iframe>

## Framing a Prediction Problem
Based on our central problem, we are going to **predict the cooking times** in dataframe. This prediction problem is a regression problem since we are predicting a continuous numeric variable. For evaluating the model, we use two common evaluation metrics: Root Mean Square Error (RMSE) and R-squared (R²). RMSE measures the average magnitude of the errors in a set of predictions, and R-squared measures the proportion of variance in the dependent variable (`'minutes'`).

At the time of prediction, we would know all the information in dataframe except the tags that explicitly describe minutes. For instance, if we know that a recipe is tagged as `'60-minutes-or-less'` or `'4-hours-or-less'`, we already know the cooking time before starts processing other features. To ensure our model learns meaningful patterns and avoids data leakage, we need to exclude this kind of tags.

## Baseline Model
Our baseline model is a linear regression model used two continuous features (`'n_steps'` and `'n_ingredients'`) to predict minutes. These two features are likely to have a relationship with cooking time. We scaled the features using Standardization to ensure equal contribution to the model, since features with different scales (e.g., n_steps range from 1 to 10, while n_ingredients range from 1 to 30) can lead to some features dominating the model's learning process.

The model's intercept is 57.87, weight for n_steps is 7.97, and weight for n_ingredients is 9.64, suggests that minutes is positively associated with both the number of steps (`'n_steps'`) and the number of ingredients (`'n_ingredients'`).
The RMSE on Training Data is 79.25, which means model's predictions deviate from the actual values by 79.25 minutes. Given that cooking times range from a few minutes to 12 hours, this could be considered a fairly large error. The RMSE of Test Data is 80.04, which is quite similar to the training data, suggesting that the model is not overfitting. 
The R-squared (**R<sup>2</sup>**) Value is 0.03, which means that the model explains only 3% of the variance in the cooking time (`'minutes'`). This is extremely low and suggests that the model is not capturing much of the variability in cooking time. It indicates that number of steps and ingredients do not explain the cooking time very well.

## Final Model
For our final model, we used `'n_steps'`,`'n_ingredients'`, `'tags'`, and `'ingredients'` as features and performed several feature engineering.
1. We one-hot encode the tags and ingredients column. Tags and ingredients such as `"vegetables"`, `"main-ingredient"`, `"equipment"`, etc., are one-hot encoded to represent the presence of each label in the recipe because they can influence the preparation time. For instance, a `"beginner-cook"` tag might indicate simpler ingredients and fewer steps, and a `"stewing pork"` can takes longer to cook than vegetables, which both influence the cooking time. In this step we extract the elements with frequency of occurence lower than 0.9 to avoid redundancy in variables.
2. We grouped some of the less common categories together under a general label to reduce the number of sparse categories to improve model performance. For example, we grouped `'tag_low-sodium'`, `'tag_low-cholesterol'`, `'tag_low-protein'`, and `'tag_low-saturated-fat'` into a single label called `'low-in-something'`.
3. We transformed the `'n_steps'` Feature into `'sqrt_n_steps'` using square root. Since we found that the relationship between the number of steps and cooking time is not strictly linear, we used square root transformation to help the model capture this relationship.
4. We removed some highly correlated features after one-hot encoding. In the heatmap, we removed one feature in any single pair of features with r greater than 0.85. The correlated features can cause multicollinearity, which can affect the stability of our model and make the coefficients hard to interpret.

We ran three different models on the training set: Polynomial Regression, Random Forest Regression and finally back to Linear Regression, and we choose linear regression as our final model. 

| Model                | RMSE (Train) | RMSE (Test) | R²        |
|----------------------|--------------|-------------|-----------|
| Baseline (Linear)    | 79.25        | 80.04       | 0.0304    |
| Polynomial           | 79.34        | 77.99       | 0.0422    |
| Random Forest        | 78.91        | 78.72       | 0.0408    |
| Final (Linear)       | 72.10        | 72.69       | 0.1796    |

The final model showed a significant improvement over the baseline model. Unlike the baseline linear regression model, which struggled to explain the variance in minutes with an R² of just 0.03, the final model achieved a much better performance, reducing the RMSE from 80.04 to 72.69 and increasing the R² to 0.18. This improvement was largely driven by the well-engineered features. The final model incorporated features such as the square root of the number of steps (sqrt_n_steps) to capture non-linear relationships, as well as one-hot encoded tags and ingredients, which  allowed the model to better account for recipe complexity and specific attributes.

In other two models, we use MAE as our matric to penalizes all errors equally, and we used GridSearchCV with 5-fold cross-validation to find the best hyperparameters. The best hyperparameter for Polynomial Regression is `'poly_features__degree': 6`, showing that introducing non-linearity did not lead to substantial performance gains. The best hyperparameter for Random Forest Regression is `'model__max_depth': 100`, `'model__min_samples_split': 10`, `'model__n_estimators': 50`, which provided similar results to polynomial regression, indicating that the additional complexity didn't significantly outperform the baseline model. One potential reason is that relationship between minutes and predictors (`'n_steps'`, `'n_ingredients'`) is  linear, adding unnecessary complexity leads to worse performance.


## Fairness Analysis
We want to check if the model perform the same for recipes submitted before 2013 and recipes submitted after 2013.

**H<sub>0</sub>**: The model is fair. The RMSE for recipes submitted before 2013 and after 2013 is roughly the same, and any observed differences are due to random chance.

**H<sub>a</sub>**: The model is unfair. The RMSE for recipes submitted before 2013 is significantly different from the RMSE for recipes submitted after 2013.

**Test Statistic**: absolute difference in RMSE between the two groups (recipes submitted before 2013 and after 2013)

**Significance Level**: 0.05

We perform a permutation test by shuffling the labels (whether a recipe was submitted before or after 2013) 1000 times and calculating the RMSE difference for each permutation. The resulting observed RMSE difference is 21.089, with p-value 0.0. Since it's far below the significance level of 0.05, we **reject the null hypothesis**, and conclude that the model is unfair. The model performs different for recipes submitted before 2013 and after 2013.

<iframe
  src="assets/Fairness_1.html"
  width="820"
  height="620"
  frameborder="0"
></iframe>
