
## Analyzing Seasonal Recipe Trends in Complexity 🍽️
##### Cecilia Marie Aban

### Introduction
The data sets subject to analyses are a subset of recipes available at "Food.com" and a corresponding sample of reviews left by users. For a site like Food.com it may be useful to investigate the waves of review traffic that occur per year. Temporal or seasonal trends can help inform decisions on what kind of recipes perform well and when. 

Accordingly, understanding the kinds of recipes that are reviewed per season is appropriate for investigation. **Are there seasonal trends regarding the complexity?** For example, are people during the holidays less likely to do recipes that are very difficult? 

**Note**: To make these sorts of conclusions about general traffic rather than review traffic, we would need to assume that the reviews that are submitted in the data set are a representative population of the titla traffic that is brought on to the site. This assumption does not consider volunteer bias. 

The first data set we have is `recipes`. It has *83782* rows and *12* columns (described below).

|Column | Description | Data Type |
|----------|----------|----------|
| name  | Recipe Name   | str  |
| id   | Recipe ID   | int64   |
|  minutes | The number of minutes required to prepare a recipe  | int64 |
| contributor_id   | User ID who submitted this recipe   | int64 |
| submitted  | Date the recipe was submitted  | str  |
| tags  | Food.com tags for recipe  | str   |
| nutrition  | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value”   | str   |
| n_steps   | 	Number of steps in recipe   | int64  |
| steps | 	Text for recipe steps, in order   | str   |
| description  | User-provided description   | str   |
| ingredients   | Text for recie ingredinets   | str   |
| n_ingredients | The number of required ingredients  | int64   |

The most important columns for temporal anlaysis of complexity will be `submitted`, `tags`, and `n_steps`. 

The second data set we have is `ratings`. It has *731927* rows and *5* columns (described below). 

|Column | Description | Data Type |
|----------|----------|----------|
| user_id  | User ID of the user who left the review   | int64 |
| recipe_id   | Recipe ID   | in64   |
|  date | date of interaction  | str |
| rating   | Rating of the recipe given  | int64 |
| review  | Text of the review given | str  |

The most imortant columns for temporal analysis of complexity will be `date`. 

## Exploratory Data Analysis 

### Data Cleaning

In order to do proper investigation, I will clean the data set accordingly. 
1. Left merge the ratings and the recipes together on `recipe_id` and `id`. 
- The most important data points to keep are the recipes that have user data. 

    **The merged data set will now be referred to as `recipes_ratings`.** 

2. Fill the `rating` values with `np.nan` if the are `0`.
- Looking at the layout of Food.com, all star ratings are out of 1-5. If the user does not leavea a rating, a rating is not dispalyed. In addition if we look through some of the `review` values that correspond to missing `rating` values. Some of them are positive in sentiment and so assigning them to `0` may artificially lower aggregate data. 
3. Find the average rating per recipe and merge back into the `recipes_ratings`. 

    **The following steps are additional cleaning / transformation steps required for the specific analyses.**

4. Convert the `date` and `submitted` columns to `pd.DateTime` objects.
- This will allow us to do transformations on dates.
- extract the month and year for every both columns into a seperate column
- I also transformed both the month the review was left and the month the recipe was first submitted into corresponding seasons. 
`{winter: 1, spring: 2, summer: 3, fall: 4}`
5. Convert the tags into a list of strings.
- Also parse the tags for every review for whether or not they contain specific tags. This allows us to analyze the *kind* of recipe that the review is being left on. 
    - If the tags included `beginner` or `easy`, I indicate recipe difficulty, `is_easy == True`.
    - If the tags include `fall, winter, spring, summer, or seasonal`, I indicate the recipe as `is_seasonal == True`.
    - If the tags include `main-dish`, I indicate the recipe as `is_main == True`


The additional or transformed columns are now as described below.  

|Column | Description | Data Type |
|----------|----------|----------|
|avg recipe rating | the average rating of a recipe | float64|
|date| the date the user left the review | dateTime|
|review_season| the season the review was submitted | int32|
|recipe_submitted_season| the season the recipe was submited | int32|
|submitted| the date the recipe was submitted |dateTime|
|month| the month the user left the review |int32|
|year| the year the user left the review | int32|
|year_submitted| the year the recipe was submitted| int32|
|tags| Food.com tags| list|
|is_easy| difficulty of the recipe according to tags | Boolean|
|is_seasonal| if the recipe is seasonal according to tags | Boolean| 
|is_main| if the recipe is for a main dish according to tags | Boolean| 

The most important columns of the final DataFrame for analyses are displayed below.

|   recipe_id | date                |   month |   year |   review_season |   rating | submitted           |   year_submitted |   recipe_submitted_season | is_easy   | is_main   | is_seasonal   |   n_steps |   rating |
|------------:|:--------------------|--------:|-------:|----------------:|---------:|:--------------------|-----------------:|--------------------------:|:----------|:----------|:--------------|----------:|---------:|
|      306785 | 2008-07-15 00:00:00 |       7 |   2008 |               3 |        5 | 2008-06-02 00:00:00 |             2008 |                         3 | True      | False     | True          |         4 |        5 |
|      310237 | 2010-05-07 00:00:00 |       5 |   2010 |               2 |        5 | 2008-06-19 00:00:00 |             2008 |                         3 | False     | False     | False         |         9 |        5 |
|      321038 | 2009-06-22 00:00:00 |       6 |   2009 |               3 |        5 | 2008-08-24 00:00:00 |             2008 |                         3 | False     | False     | False         |        14 |        5 |


`n_steps` will be used in complexity investigations. `recipes_ratings` has *234426* rows and *28* columns. 

### Univariate Analysis
The first thing that would be useful to determine is if there is a month that brings in a lot of review traffic. To do this we plot the distribution of review counts per month. This way we can gear temporal trends around the most popular month. It looks like all traffic is even per month. 

<iframe
  src="assets\review_month_distr.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

To determine if users generally leave reviews on more recent recipes. We can look at the distribution of the year difference between when the user reviewed the recipe and when the review was first submitted. This would give us an idea of general review traffic in terms of recency.

<iframe
  src="assets\time_from_submitted.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

It looks like most reviews are left within two years of when the recipe was first submitted. This may hint that review traffic is most likely on new or current recipes. 

### Bivariate Analysis 
The next thing I want to investigate is whether or not people generally review the recipe in the same or adjacent season as the recipe was initially posted. If we condition this statistic on whether or not the recipe is considered seasonal, we can see if people are reviewing seasonal recipes in different seasons similarly to non seasonal recipes. To do this, I have to considered that there is a cyclic relationship between seasons; winter is close to fall even though I have labeled the `1` and `4`. I used cyclic difference to do this. 

<iframe
  src="assets\seasonal_diff.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

What this shows us is that for both seasonal and non seasonal recipes reviews are left mostly within 1 season away from when the review was posted. Though the distribution may differ between the two groups;proportions vary between the two groups. The proportion of reviews that were left within the same season is higher for the seasonal recipes than the non-seasonal recipes. 

I want to investigate whether there is any other representative statistics of recipe complexity that I can use in my analyses. Logically, the number of steps a recipe may require may be alligned with recipe complexity. To do this I investigated the distribution of `n_steps` when conditioned on `is_easy`.

<iframe
  src="assets\steps_by_diff.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The modes and averages are different whether the group is considered easy or if it is not. We can investigate this further with aggregate data. 

### Interesting Aggregates

| is_easy   |   ('mean', 'minutes') |   ('mean', 'n_ingredients') |   ('mean', 'n_steps') |   ('median', 'minutes') |   ('median', 'n_ingredients') |   ('median', 'n_steps') |
|:----------|----------------------:|----------------------------:|----------------------:|------------------------:|------------------------------:|------------------------:|
| False     |               94.4941 |                    10.7953  |              12.6056  |                      45 |                            10 |                      11 |
| True      |              100.213  |                     7.83355 |               8.15922 |                      30 |                             7 |                       7 |

Continuing with the analysis of whether there are other representative statistics of recipe complexity, both the mean and median number of ingredients, `n_ingredients` and number of required steps, `n_steps` is lower for easy recipes. The mean number of minutes, `minutes`, required is higher for easy recipes, but the median number of minutes is lower for easy recipes. This may indicate an outlier number of minutes in the easy category.

## Assessment of Missingness
### NMAR Missingness
In the final DataFrame, `recipes_ratings`, there are *57* `review` values missing. The missingness mechanism of this column is likely **NMAR**. A user is probably more likely to leave a review if they feel strongly about a recipe, whether the love it or they hate. A user that is less interested or indifferent about the recipe would likely not be compelled to leave a review. 

## MAR Missingnes
There are *15035* missing `rating` values missing. As previously mentioned, investigating the `review` column of the corresponding missing `rating` rows, the `review` may very positive, but the `rating` is missing. This indicates that the missingness is significant. 

Is `rating` MAR dependent on `n_steps`?

**Null Hypothesis**: The missing and non-missing groups of `rating` have the same distribution of `n_steps`. \
**Alternative Hypothesis**: The missing and non-missing groups of `rating` have a different distribution of `n_steps`. 

Use a permutation test using the *absolute difference in group means*. `n_steps` is a numerical variable.

<iframe
  src="assets\steps_MAR.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

I shuffled the `rating` column a *1000* times. Each time, I calculated the test statistic to compare the null and non-null data. After the simulation I compared the simulated test statistics with the observed test statistic by calculating **p-value = 0**. In the above graph, the observed value is represented by the red line ; its far from the rest of the simulated data.  

The p-value is 0; for any significance level,  we reject the null hypothesis. The missingness of `ratings` *is* MAR dependent on `n_steps`.

Is `rating` MAR dependent on `is_main`, where `is_main` is a boolean corresponding to whether the recipe is a main dish?

**Null Hypothesis**: The missing and non-missing groups of `rating` have the same distribution of `is_main` counts. \
**Alternative Hypothesis**: The missing and non-missing groups of `rating` have a different distribution of `is_main` counts. 

Use a permutation test using the *total variation distance*. `is_main` is a categorical variable. 

<iframe
  src="assets\main_MAR.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Using a similar testing procedure as explained above. I calculated the **p-value = 0.845**. For a significance level, alpha = 0.05, p-value > alpha. We fail to reject the null hypothesis. The misingness of `rating` is *not* MAR dependent on `is_main`.


## Hypothesis Testing
In order to further investigate if there are seasonal trends in recipe complexity, I want to investigate if the monthly distribution of recipes depends on whether the recipe is easy or not. 

**Null Hypothesis**: Recipes that are considered easy have the same month distribution than those that are not. \
**Alternative Hypothesis**: Recpies that are considered easy have a different month distribution than those that are not.

We will use the *total variation distance* as the test statistic and a permutation test to investigate the signifance of the `is_easy` group label. `month` is a categorical variable for this analysis.

<iframe
  src="assets\season_difficulty.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

For alpha = 0.05, **p-value < alpha** , we reject the null hypothesis. The monthly distribution of recipes depends on whether the dish is easy or not. This hints at there possible temporal patterns in recipe complexity.

## Framing A Prediction Problem

I want to predict the season that one review will be left. 

This is a **multi-class classification problem** with the **season** that the review being submitted is the *response variable*. I choose this because the distribution of recipes left per season is even and there is real world implications of being able to predict this. Primarily, a company can use a model that can predict any one review's season to simulate a year reviews for one particular recipe. This could guide the site to posting certain kinds of recipes to manipulate review traffic during a particular season. In addition, my hypothesis test has provide some evidence towards temporal / seasonal differences in the complexity of the recipe. 

For the problem I will be using **accuracy** as metric. This is reasonable since the distribution of recipes across seasons is fairly even; I have generally balanced classes. I will also interpret **F1 Scores** per class, to see if there are some features that overlap during some seasons leading to ambiguity. For predictors, I only have access to features that are available at the time of prediction. Specifically, I will only be using information about a recipe, not the review left. 

I will split the data into training and test sets with my test sets being 25% of the entire data set. For reproducibility, I fixed the splitting of the data's randomness. 

In general predicitng time-based response variables is difficult because we are relying on users' consistent trends. A model may not be able to decipher and appriopriately interpret ambiguity in the features that come with usage trends. 

## Baseline Model
Since we have some evidence towards temporal / seasonal difference in the complexity of the recipe from hypothesis testing, I will first train a model based on representative statistics of  recipe complexity: `n_steps` and `is_easy`. 

I pass `n_steps` without any transformations into a ColumnTransformer ; it is a numerical variable. I one-hot encoded `is_easy`, a binary, nominal categorical variable. I train a `sklearn` RandomForestClassifier on the training sets with default parameters.

**Training Set Accuracy**: **0.2838 **\
**Test Set Accuracy**: **0.2788**

The training set accuracy and test set accuracy are very close, so whether the model is overfitting or underfitting is not clear. In general, the accuracy is low 🫢. I would not use this to simulate a year of reviews for a single recipe. 

**F1 Score Per Class**: **[0.08, 0.39, 0.3 , 0.08]**

The F1 score for winter and fall are very low compared to the rest of the season even though the data is balanced per class. This likely indicates feature imbalance for these classes that should be improved upon. 


## Final Model

One very obvious feature that we can add to improve the model is whether the recipe was tagged with a specific season. If there is a recipe that is seasonally winter in nature, it is logically to say that a someone would likely not try that recipe in the middle of summer. To add this I pass the `tags` column into a FunctionTransform which, for every recipe's tags, outputs an array of one-hot encoded vectors that corresponding to whether `summer, winter, fall, or spring` was included into the tags. 

From previous exploritory data anlysis, we know that a lot of reviews tend to be posted close to the season that the recipe was posted. I one-hot encoded the `recipe_submitted_season` column to add as a feature. 

In addition, I preformed 3-fold CrossValidation using `GridSearchCV` on a hyperparameter grid. Even though it is unclear whether the model is overfittingor underfitting because the training and test set errors of the baseline model are similar, I will try to reduce model complexity for performance reasons. For example, if I can use least trees per forest and still get similar performance, that may be perferred. I grid searched through 16 models with varying *number of tress in a forest*, `n_estimators`, and the maximum depth of a single tree, `max_depth`. 

The grid search yeilded the best hyperparameters: `max_depth = 10` and `n_estimators = 75`. `sklearn`'s default parameters are `max_depth = None` and `n_estimators = 100`. 

**Training Set Accuracy**: **0.3598** \
**Test Set Accuracy**: **0.3537**

Test set accuracy increased by approximately 27%. Dispite this, the accuracy is still quite low. I would not use this to simulate a year of reviews for a single recipe. 🫢 

**F1 Score Per Class**: **[0.32, 0.36, 0.41, 0.31]**

The F1 scores are more balanced per class with the new model. 

I can further attribute low model accuracy to, in general, weak temporal trends. This can further be attributed to the assumptions we are making. Food.com is used all over the word where seasonal trends in food may not be well aligned with months in the year. Per region analysis would likely lead to higher accuracy if the data were available. 

## Fairness Analysis

Does my model have the same accuracy for recipes of main dishes as compared to recieps that are not main dishes? 
- Group X: recipes `is_main == True` ; recipes for main dishes
- Group Y: recipes `is_main == False` ; recipes for not main dishes

**Null Hypothesis**: The model is fair. Its classification accuracy for main dishes is the same as recipes that are not main dishes.  \
**Alternative Hypothesis**: The model is unfair. Its classification accuracy for main dishes is different as recipes that are not main dishes. 

We will use the *absolute difference in group means* as the test statistic with a *permutation test*
<iframe
  src="assets\fairnes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


With significance level, alpha = 0.05, the p-value = 0.982 > alpha, we fail to reject the null hypothesis. The model achieves accuracy parity between group *X* and *Y*.