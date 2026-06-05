by Anya Garg

# Introduction

Hawaii, in recent years, has seen a burgeoning tourism industry that has drawn a wedge between wealthier and less wealthy regions, leading to stark discrepancies in the types of goods and services offered across regions, who consumes them, and how consumers feel about the product. Naturally, a question was raised: do businesses in more affluent regions recieve better ratings or engagement? and which features on the whole lead to the most positive engagement?

To begin, I load in data from Google Maps containing reviews of various locations in Hawaii. It was originally scraped and used by the authors of these papers: https://aclanthology.org/2022.acl-long.426.pdf, https://arxiv.org/pdf/2207.00422. The full dataset is quite long, so I will only be using the 10-core one.

The first dataset, the `review` dataset, contains 1,504,347 rows and 8 columns as follows:

| Column | Description |
|----------|----------|
| `user_id` | ID of the reviewer |
| `name` | Name of the reviewer |
| `time` | Time of the review (Unix timestamp) |
| `rating` | Rating given by the reviewer |
| `text` | Text of the review |
| `pics` | Pictures attached to the review |
| `resp` | Business response, including response time and text |
| `gmap_id` | ID of the business |

The second `meta` dataset contains 21,507 rows and 15 columns:

| Column | Description |
|----------|----------|
| `name` | Name of the business |
| `address` | Address of the business |
| `gmap_id` | ID of the business |
| `description` | Description of the business |
| `latitude` | Latitude of the business |
| `longitude` | Longitude of the business |
| `category` | Business categories |
| `avg_rating` | Average rating of the business |
| `num_of_reviews` | Number of reviews |
| `price` | Price level of the business |
| `hours` | Business operating hours |
| `MISC` | Additional business information |
| `state` | Current business status (e.g., permanently closed) |
| `relative_results` | Related businesses recommended by Google |
| `url` | URL of the business |

# Step 2: Data Cleaning and Exploratory Data Analysis

In order to perform meaningful analysis, I had to merge several datasets together. This combining and cleaning proccess was conducted as follows:
1. Perform an outer merge on the `review` dataset and `meta` dataset together on the `gmap_id` column
2. Upload another dataset tracking which businesses are in wealthier and less wealthy regions, from the American Community Survey in Hawaii (https://data.census.gov/table/ACSDP5Y2018.DP03?g=040XX00US15&layer=VT_2018_140_00_PY_D1&hidePreview=false&cid=DP03_0001E&vintage=2018&tid=ACSDP5Y2018.DP03), and filter to only the neccessary columns (`GEOID`, `NAME`, `DP03_0062E` (representing median income))
3. To link them together, upload a census tract shapefile to merge on the `GEOID` column of the ACS dataset ("2020 Census Tracts").
4. Converted the merged set into a CRS (Coordinate Reference System) and join with first dataset with ```gpd.sjoin(hawaii_gpd, tracts_wealth, how="left", predicate="within")```
5. select only important columns and rename
6. Add a column called `wealth group`, a binary column stating `high` for businesses in regions where income is higher than overall median and `low` for businesses in regions where income is lower.

The first 5 rows of the cleaned dataframe are displayed below:

| gmap_id | rating | text | pics | resp | address | avg_rating | num_of_reviews | price | hours | relative_results | url | category | median_income | wealth_group | business_responds |
|----------|----------|----------|----------|----------|----------|----------:|----------:|----------|----------|----------|----------|----------|----------:|----------|----------|
| 0x0:0x9edcb14b0cf1ec04 | 5.0 | The whole diving experience was amazing, the service... | None | {'time': 1621844365748, 'text': 'Aloha Jesse, ...'} | Dive Oahu, 609 Keawe St, Honolulu, HI 96813 | 4.2 | 278 | None | [[Sunday, 8AM–6PM], ...] | [0x7c006e090607b10d:..., ...] | https://www.google.com/maps/place/... | [SCUBA instructor] | 79180.0 | Low | True |
| 0x0:0x9edcb14b0cf1ec04 | 5.0 | Alex and Morgan were phenomenal teachers. Took... | None | {'time': 1618905442032, 'text': 'Thanks for th...'} | Dive Oahu, 609 Keawe St, Honolulu, HI 96813 | 4.2 | 278 | None | [[Sunday, 8AM–6PM], ...] | [0x7c006e090607b10d:..., ...] | https://www.google.com/maps/place/... | [SCUBA instructor] | 79180.0 | Low | True |
| 0x0:0x9edcb14b0cf1ec04 | 5.0 | I did my scuba diving certification with Dive... | None | {'time': 1612794140450, 'text': 'Aloha Nigel!...'} | Dive Oahu, 609 Keawe St, Honolulu, HI 96813 | 4.2 | 278 | None | [[Sunday, 8AM–6PM], ...] | [0x7c006e090607b10d:..., ...] | https://www.google.com/maps/place/... | [SCUBA instructor] | 79180.0 | Low | True |
| 0x0:0x9edcb14b0cf1ec04 | 5.0 | Great experience great dive guides. There was... | [{'url': ['https://lh5.googleusercontent.com/...']}] | {'time': 1515689303717, 'text': 'Aloha, Amy!...'} | Dive Oahu, 609 Keawe St, Honolulu, HI 96813 | 4.2 | 278 | None | [[Sunday, 8AM–6PM], ...] | [0x7c006e090607b10d:..., ...] | https://www.google.com/maps/place/... | [SCUBA instructor] | 79180.0 | Low | True |
| 0x0:0x9edcb14b0cf1ec04 | 5.0 | Excellent instruction and instructors which led... | None | {'time': 1612796702509, 'text': 'Aloha Ismael!...'} | Dive Oahu, 609 Keawe St, Honolulu, HI 96813 | 4.2 | 278 | None | [[Sunday, 8AM–6PM], ...] | [0x7c006e090607b10d:..., ...] | https://www.google.com/maps/place/... | [SCUBA instructor] | 79180.0 | Low | True |

After cleaning, the dataframe has 1514800 rows and 16 columns.

## Univariate Plots

I aim to look at the distributions of relevant columns, namely ratings and income. This is the first step of our exploratory data analysis. The idea behind this is to get a sense of how exactly the data is stuctured, so I can move forward with our later analysis.

<iframe
  src="assets/uni_1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The histogram above shoes that average ratings are concentrated far to the right, meaning ratings are generally high. Additionally, it is worth noting that this analysis was done on the dataframe with individual rows per review, so business with more reviews are weighted more heavily in the results, which could be partially responsible for the severity of the skew (popular business are likely to be ones that are good, and these likely have high ratings).

<iframe
  src="assets/uni_2.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

An identical analysis into median income shows an opposite affect; a slight left skew, indicating that the bulk of restaurants lie in areas where residents make around 100k yearly.

## Bivariate Analysis

I continue exploring the nature of our dataset by investigating statistics for pairs of columns to identify possible associations. The variables in question here are the two explored above; income and ratings. I split income into the aforementioned wealtg groups, then plot boxplots of ratings for each.

<iframe
  src="assets/biv.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of business ratings is remarkably similar across income groups. Most businesses, regardless of the wealth of the surrounding region, receive ratings between 4.0 and 4.6 stars, with peaks near 4.4 stars. While businesses located in higher-income regions appear to have slightly higher ratings on average, the substantial overlap between distributions suggests that regional income is not a major determinant of customer ratings.

## Interesting Aggregates

I then create a pivot table displaying the average rating for businesses of a certain price level in neighborhoods with different median incomes, to investigate whether wealthier neighborhoods are less averse to paying high prices than less wealthy ones.

| wealth_group | $ | $$ | $$$ | $$$$ | ₩ | ₩₩ | ₩₩₩ | ₩₩₩₩ |
|--------------|------:|------:|------:|------:|------:|------:|------:|------:|
| Low | 4.180237 | 4.302138 | 4.310334 | 4.507408 | 4.085205 | 4.361706 | NaN | 4.500000 |
| High | 4.179796 | 4.298242 | 4.351764 | 4.524514 | 4.531418 | 4.305579 | 4.227101 | 4.313254 |

# Step 3: Assessment of Missingness

We now perform a missingness analysis. Missingness is classified into 4 types: Missing Completely at Random, Missing at Random, Missing Not at Random, and Missing by Design. Missing Completely at Random (MCAR) is a situation where the probability of data being missing is completely independent of both observed and unobserved data. It is purely accidental. Missing at Random (MAR) is where the missingness is systematically related to observed data but not the unobserved data. Not Missing at Random (NMAR) is when the missingness depends on the unobserved data itself. Missing by Design (MD) is data that is missing due to a logical design in the dataset, rather than being lost. In our dataset, there are likely 2 columns that are NMAR: `price` could be NMAR, as businesses often only report price information when applicable. Missingness may depend on the underlying price category itself. `hours` may also be NMAR, as Businesses without fixed operating hours are more likely to have missing hours. Missingness depends on the actual hours information. To analyse some of the remaining missingnes, we construct a pivot table first showing the proportion of reviews with missing ratings across review count groups.

## NMAR Analysis

| review_count_group | False | True |
|-------------------|--------:|-------:|
| 0-49 | 0.885060 | 0.114940 |
| 50-99 | 0.997264 | 0.002736 |
| 100-149 | 0.999187 | 0.000813 |
| 150-199 | 0.999619 | 0.000381 |
| 200-249 | 0.999789 | 0.000211 |
| 250-299 | 0.999931 | 0.000069 |
| 300-349 | 0.999899 | 0.000101 |
| 350-399 | 1.000000 | 0.000000 |
| 400-449 | 0.999953 | 0.000047 |
| 450-499 | 0.999978 | 0.000022 |
| 500-549 | 1.000000 | 0.000000 |
| 550-599 | 1.000000 | 0.000000 |
| 600-649 | 1.000000 | 0.000000 |
| 650-699 | 1.000000 | 0.000000 |
| 700-749 | 0.999967 | 0.000033 |
| 750-799 | 1.000000 | 0.000000 |
| 800-849 | 1.000000 | 0.000000 |
| 850-899 | 1.000000 | 0.000000 |
| 900-949 | 1.000000 | 0.000000 |

Here we see that the proportion of missing ratings varies substantially across review-count groups. Businesses with fewer than 50 reviews have an 11.5% missing-rate, while businesses with more than 100 reviews have missing-rates below 0.1%. Because the probability of missingness depends on the observed variable num_of_reviews, the missingness mechanism is unlikely to be Missing Completely at Random (MCAR). Instead, the evidence is consistent with Missing At Random (MAR), suggesting that analyses involving ratings should account for review count to mitigate potential bias.

Therefore, to investigate whether missing ratings are associated with the number of reviews a business has received, I conducted a permutation test.

Null Hypothesis (H_0): The missingness of a business's rating is independent of its number of reviews. Any observed difference in review counts between businesses with missing ratings and those with observed ratings is due to random chance.

Alternative Hypothesis (H_1): The missingness of a business's rating depends on its number of reviews.

The test statistic used was the difference in mean number of reviews between businesses with missing ratings and businesses with observed ratings:

\text{Mean Reviews}_{\text{Observed}}
]

To generate the null distribution, the missingness indicator was randomly shuffled 5,000 times while keeping the review counts fixed. For each shuffle, the difference in mean review counts was recomputed. The p-value was calculated as the proportion of simulated statistics whose absolute value was at least as extreme as the observed statistic.

<iframe
  src="assets/missingness_perm.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The resulting permutation distribution was centered near zero, while the observed statistic fell far in the tail of the distribution. The resulting p-value was effectively zero (or extremely small), providing strong evidence against the null hypothesis.

These results suggest that rating missingness is related to the number of reviews a business has received. In particular, businesses with relatively few reviews are substantially more likely to have missing ratings than businesses with many reviews. Therefore, the missingness mechanism is unlikely to be Missing Completely At Random (MCAR) and is more consistent with Missing At Random (MAR), since missingness appears to depend on an observed variable (`num_of_reviews`).

We constructed an alternative visualization to further demonstrate our results:

<iframe
  src="assets/missingness_line.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This pattern looks odd on the surface, but shows much higher proportions of missingness for businesses that have fewer reviews than businesses that have many. This could indicate that the pattern of missing individual ratings is systematically tied to the business (ie. every business, regardless of how many revues it has, has 1 missing rating). Therefore, to investigate this, we check exactly how many businesses have 0 missing ratings, 1 missing ratings, 2 missing, 3 missing, etc. displayed in the following table:

| rating | count |
|--------|------:|
| 0 | 11,686 |
| 1 | 9,681 |
| 2 | 54 |

Here, we see almost every business has at most 1 or 2 missing ratings. This could indicate that the missingness was actually introduced during the merging proccess, since some businesses may not have had any ratings at all. We verified this by checking whether or not the number of missing rows is the same as the number of businesses not present in `review_df` but present in `meta_df`, which evaluated to 9,735 businesses. This is exactly equivalent to the total number of businesses above that experienced missingness, 9,681 + 54 = 9,735, confirming our suspicion that businesses with missing ratings correspond to businesses that have metadata records but no matching review records. As a result, we regard this as a structural missingness, or Missing by Design (MD).

# Step 4: Hypothesis Testing

As mentioned in the introduction, I wish to explore if a business' location in a wealthier location leads to different ratings. It could be the case that businesses in more affluent neighborhoods are treated differently, as consumers living near them might be more or less inclined to see spending there as a worthwhile and appropriate use of their money, and thus respond differently. As such, we test the following hypotheses:

Null hypothesis: There is no difference in ratings in regions with high levels of affluence and low levels.

Alternative hypothesis: There is a difference in ratings in regions with high levels of affluence and low levels.

Test statistic: Signed difference in means.

Significance level: 0.05.

We shuffled the `wealth_group` column (which we created in the data cleaning process) 500 times and computed signed difference in means. I have visualized this distribution below:

<iframe
  src="assets/perm_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

`p value: 0.108`

As shown above, the p value is not very significant, indicating that we fail to reject the null hypothesis here. Additionally, the observed difference of 0.0220 stars is not immense, indicating at the total group level that there is no such relationship present.

# Step 5: Framing a Prediction Problem

So far, our analysis shows wealthier regions aren't really associated with higher ratings. however, we do seem to see that number of reviews is highly variable, and could have some relationship with wealth groups. Furthermore, in accrodance with our line of thinking, it is a good metric for customer engagement and could service our question of whether or not better customer engagement is recieved by businesses in wealthier neighborhoods. We therefore run an additional hypothesis test:

Null hypothesis: There is no difference in number of reviews in regions with high levels of affluence and low levels.

Alternative hypothesis: There is a difference in number of reviews in regions with high levels of affluence and low levels.

Test statistic: Signed difference in means.

Significance level: 0.05.

<iframe
  src="assets/perm2.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

p value: 0.004

As shown, these results are much more significant and more promising. We therefore approach a refined goal to model: Can we predict how much customer engagement a business receives based on its characteristics and the socioeconomic characteristics of its surrounding region?

# Step 6: Baseline Model

As a baseline, I fit a linear regression model to predict the number of reviews a business receives using two simple features: the median income of the surrounding area and the business's price category. Since price is categorical, it was one-hot encoded before fitting the model.

The model achieved an R² of 0.0083 on the training set, indicating that less than 1% of the variation in review counts can be explained by these predictors. This suggests that neither local income levels nor price category are strong determinants of the number of reviews a business receives.

To evaluate predictive performance, I computed the root mean squared error (RMSE). The training RMSE was 1876.99 reviews, while the testing RMSE was 1883.28 reviews. Because the training and testing errors are highly similar, there is little evidence of overfitting. Instead, the model appears to underfit the data, failing to capture important factors that influence customer engagement.

Overall, this baseline demonstrates that simple demographic and pricing information alone is insufficient for accurately predicting review counts. More informative business characteristics and textual features are likely needed to improve predictive performance.

# Step 7: Final Model

## Final Model

To improve upon the baseline model, I engineered several additional features that capture characteristics of businesses and customer engagement that are likely related to review volume.

### Feature Engineering

#### Review Text Length

I first transformed the raw review text into a quantitative variable by computing the number of characters in each review:

```python
analysis_df["text_length"]
```

Longer reviews may indicate greater customer engagement and could be associated with businesses that receive more attention.

Because businesses often have many reviews, I also created an aggregated feature:

```python
analysis_df["avg_text_length"]
```

which measures the average review length for each business. This captures the overall depth of customer feedback rather than the content of a single review.

#### Presence of Images

I created a binary indicator variable:

```python
analysis_df["has_img"]
```

that equals `True` when a review contains an image and `False` otherwise.

Including images generally requires greater customer effort and may serve as a proxy for customer engagement.

#### Business Categories

The original `category` column contained lists of business categories. To make this information usable in a machine learning model, I extracted the ten most common categories and created binary indicator variables for each one.

For example:

```python
category_Restaurant
category_Coffee Shop
category_Tourist Attraction
```

takes value 1 if a business belongs to that category and 0 otherwise.

This allows the model to capture systematic differences in review volume across different types of businesses.

---

### Final Features

The final model uses the following predictors:

#### Numerical Features

- Rating
- Median income
- Average review text length
- Presence of images

#### Categorical Features

- Price category
- Top 10 business category indicators

---

### Data Preprocessing

Before fitting the model, I constructed a preprocessing pipeline.

#### Numerical Variables

For numerical features:

1. Missing values were replaced using the median.
2. Features were standardized using z-scores.

```python
SimpleImputer(strategy="median")
StandardScaler()
```

Standardization prevents variables measured on different scales from disproportionately influencing the model.

#### Categorical Variables

For categorical features:

1. Missing values were replaced with `"Missing"`.
2. Variables were one-hot encoded.

```python
SimpleImputer(strategy="constant", fill_value="Missing")
OneHotEncoder(handle_unknown="ignore")
```

One-hot encoding converts categorical variables into binary indicators that can be used by machine learning algorithms.

---

## Linear Regression Final Model

I first fit a multiple linear regression model using all engineered features.

### Results

| Metric | Value |
|----------|----------:|
| Training R² | 0.143 |
| Test RMSE | 1744.48 |

Compared with the baseline model:

| Model | R² | Test RMSE |
|---------|---------:|---------:|
| Baseline | 0.008 | 1883.28 |
| Final Linear Regression | 0.143 | 1744.48 |

### Interpretation

The additional features substantially improved predictive performance.

The model now explains approximately **14.3% of the variation** in review counts, compared with less than 1% for the baseline model.

Similarly, RMSE decreased from roughly **1880 reviews to 1750 reviews**, indicating more accurate predictions.

Although the model still leaves a large amount of unexplained variation, the improvement suggests that review characteristics and business type contain useful information about customer engagement.

---

## Lasso Regression

Because the final model contains many engineered variables and category indicators, I also fit a Lasso regression model.

Lasso adds an L1 penalty to the regression objective:

\[
RSS + \lambda \sum |\beta_j|
\]

which shrinks unimportant coefficients toward zero and performs automatic feature selection.

### Hyperparameter Selection

The optimal penalty parameter was selected using cross-validation.

**Best alpha:** 0.4112

### Results

| Metric | Value |
|----------|----------:|
| Training R² | 0.143 |
| Training RMSE | 1747.16 |
| Test RMSE | 1744.54 |

### Interpretation

Lasso performed almost identically to ordinary linear regression.

The nearly identical R² and RMSE values suggest that most engineered features contribute useful information and that excessive overfitting is not occurring.

Because Lasso produces a more parsimonious model while maintaining similar predictive accuracy, it provides evidence that the selected features are reasonably robust.

---

## Random Forest Model

Finally, I trained a Random Forest Regressor.

Unlike linear regression, random forests can capture:

- Nonlinear relationships
- Feature interactions
- Threshold effects
- Complex decision boundaries

without requiring them to be specified manually.

### Hyperparameter Tuning

Due to technological constraints, I was not able to test very many parameters due to high runtime. I only tested 4 separate parameters here, with 100 trees. I used GridSearchCV with 5-fold cross-validation to tune:

```python
n_estimators = [100]
max_depth = [10, 15]
min_samples_split = [2, 5]
```

The optimal hyperparameters were:

```python
max_depth = 15
min_samples_split = 2
n_estimators = 100
```

### Results

| Metric | Value |
|----------|----------:|
| Training R² | 0.940 |
| Training RMSE | 463.78 |

### Interpretation

The random forest achieves dramatically better fit on the training data than either linear regression model.

A training R² of 0.94 indicates that the model explains approximately 94% of the variation in review counts within the training set.

Similarly, RMSE decreases from roughly 1750 reviews to only 463 reviews.

This large improvement suggests that relationships between business characteristics and review counts are highly nonlinear and involve interactions that linear models cannot capture.

However, the training metrics alone do not indicate whether the model generalizes well to unseen data. To evaluate potential overfitting, the test-set R² and RMSE should be compared with the training results. If test performance remains similarly strong, the random forest would be the preferred model. If test performance deteriorates substantially, the model may be overfitting the training data.

---

## Model Comparison

| Model | Training R² | Test RMSE |
|---------|---------:|---------:|
| Baseline Linear Regression | 0.008 | 1883.28 |
| Final Linear Regression | 0.142 | 1747.09 |
| Lasso Regression | 0.143 | 1747.16 |
| Random Forest | 0.940 | 461.98 |

The progression from the baseline model to the final models demonstrates that review characteristics, image presence, ratings, and business categories provide substantially more predictive power than neighborhood income and price alone. 

The Random Forest model achieved the strongest predictive performance of all models considered. After hyperparameter tuning using 5-fold cross-validation, the optimal model used 100 trees, a maximum depth of 15, and a minimum split size of 2 observations.

The model achieved a training R² of 0.940 and a test R² of 0.940, indicating that approximately 94% of the variation in review counts can be explained by the predictors included in the model. Additionally, the model achieved a training RMSE of 461.34 and a test RMSE of 463.93.

Importantly, the near-identical training and testing performance suggests that the model generalizes well to unseen data and does not exhibit meaningful overfitting. The Random Forest substantially outperformed both the baseline and linear regression models, suggesting that review volume is driven by complex nonlinear relationships and interactions among business characteristics, customer ratings, images, review text characteristics, and business categories.

Given its superior predictive accuracy and strong out-of-sample performance, the Random Forest was selected as the final model.

# Step 8: Fairness Analysis

Finally, I conducted a fairness analysis to determine whether the final Random Forest Regressor performs differently for businesses located in high-income versus low-income regions. Businesses were assigned to income groups using the wealth_group variable derived from median household income.

Because the target variable (num_of_reviews) varies substantially across businesses, comparing raw RMSE can be misleading; businesses with larger review counts naturally tend to have larger prediction errors. To account for this, I measured model performance using relative error, defined as the absolute prediction error divided by the true number of reviews. This allows for a comparison of proportional prediction accuracy across groups.

To assess fairness, I performed a permutation test. First, I computed the observed difference in mean relative error between low-income and high-income businesses. I then repeatedly shuffled the wealth_group labels across the test set, recomputed the difference in mean relative error for each permutation, and used the resulting distribution as an empirical null distribution. The p-value was calculated as the proportion of permuted differences that were at least as large as the observed difference.

Null Hypothesis: The model is fair; any difference in mean relative error between low-income and high-income businesses is due to random chance.

Alternative Hypothesis: The model is unfair; businesses in low-income regions experience higher mean relative prediction error than businesses in high-income regions.

Test Statistic: Difference in mean relative error (Low Income − High Income)

Significance Level: α = 0.05

<iframe
  src="assets/fairness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The observed difference in mean relative error was approximately 0.54, meaning that predictions for businesses in low-income regions were, on average, associated with substantially larger proportional errors than predictions for businesses in high-income regions. As shown above, the observed statistic lies far to the right of the permutation distribution, and none of the 1,000 permutations produced a difference as extreme as the observed value, resulting in a p-value less than 0.001.

Therefore, I reject the null hypothesis. The results provide strong evidence that the model's predictive performance differs across income groups, with substantially higher relative prediction error for businesses located in low-income regions. This suggests that the model does not perform equally well across wealth groups and may systematically disadvantage businesses in lower-income areas.

While the permutation test provides strong evidence that the model's prediction errors differ across wealth groups, it does not identify the underlying cause of this disparity. Future work could investigate whether the difference arises from unequal distributions of business categories, review counts, pricing levels, or other characteristics that vary across income regions. Additional fairness-aware modeling approaches or feature engineering techniques could also be explored to reduce this performance gap.

## Citations

“2020 Census Tracts.” Hawaii.Gov, 2020, https://geoportal.hawaii.gov/datasets/HiStateGIS::2020-census-tracts/explore?location=31.072143%2C54.064055%2C3. Accessed 5 June 2026.
‌
“Explore Census Data.” Census.Gov, 2026, https://data.census.gov/table/ACSDP5Y2018.DP03?g=040XX00US15&layer=VT_2018_140_00_PY_D1&hidePreview=false&cid=DP03_0001E&vintage=2018&tid=ACSDP5Y2018.DP03. Accessed 5 June 2026.
‌
