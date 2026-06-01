Do Businesses in Wealthier Regions Recieve Worse Reviews?

by Anya Garg

# Introduction

Hawaii, in recent years, has seen a burgeoning tourism industry that has drawn a wedge between wealthier and less wealthy regions, leading to stark discrepancies in the types of goods and services offered across regions, who consumes them, and how consumers feel about the product. Naturally, a question was raised: do businesses in more affluent regions recieve better ratings or engagement? and which features on the whole lead to the most positive engagement?

To begin, I load in data from Google Maps containing reviews of various locations in Hawaii. It was originally scraped and used by the authors of these papers: https://aclanthology.org/2022.acl-long.426.pdf, https://arxiv.org/pdf/2207.00422. The full dataset is quite long, so I will only be using the 10-core one.

The first dataset, the `review` dataset, contains 1,504,347 rows and 8 columns as follows:
|  **Column**       |  **Description** |
| ---------         | ------------ |
|`user_id`	        |   ID of the reviewer    |
|`name`             |   name of the reviewer      |
|`time`             |   time of the review (unix time)  |
|`rating`           |	rating of the business   |
|`text`             |	text of the review   |
|`pics`	            |   pictures of the review   |
|`resp`             |	business response to the review including unix time and text of the response   |
|`gmap_id`          |	ID of the business   |

The second `meta` dataset contains 21,507 rows and 15 columns as follows:
|  **Column**       |  **Description** |
| ---------         | ------------ |
|`name`	            |   name of the business    |
|`address`          |   address of the business      |
|`gmap_id`          |   ID of the business  |
|`description`      |	description of the business   |
|`latitude`         |	latitude of the business   |
|`longitude`	      |   longitude of the business   |
|`category`         |	category of the business   |
|`avg_rating`       |	average rating of the business   |
|`num_of_reviews`	  |   number of reviews    |
|`price`            |   price of the business     |
|`hours`            |   open hours    |
|`MISC`             |	MISC information    |
|`state`            |	the current status of the business (e.g., permanently closed)   |
|`relative_results`	|   relative businesses recommended by Google   |
|`url`              |	URL of the business   |

# Step 2: Data Cleaning and Exploratory Data Analysis

In order to perform meaningful analysis, I had to merge several datasets together. This combining and cleaning proccess was conducted as follows:
1. Perform an outer merge on the `review` dataset and `meta` dataset together on the `gmap_id` column
2. Upload another dataset tracking which businesses are in wealthier and less wealthy regions, from the American Community Survey in Hawaii (https://data.census.gov/table/ACSDP5Y2018.DP03?g=040XX00US15&layer=VT_2018_140_00_PY_D1&hidePreview=false&cid=DP03_0001E&vintage=2018&tid=ACSDP5Y2018.DP03), and filter to only the neccessary columns (`GEOID`, `NAME`, `DP03_0062E` (representing median income))
3. To link them together, upload a census tract shapefile to merge on the `GEOID` column of the ACS dataset (https://geoportal.hawaii.gov/datasets/HiStateGIS::2020-census-tracts/explore?location=31.072143%2C54.064055%2C3).
4. Converted the merged set into a CRS (Coordinate Reference System) and join with first dataset with ```gpd.sjoin(hawaii_gpd, tracts_wealth, how="left", predicate="within")```
5. select only important columns and rename
6. Add a column called `wealth group`, a binary column stating `high` for businesses in regions where income is higher than overall median and `low` for businesses in regions where income is lower.

The first 5 rows of the cleaned dataframe are displayed below:

| gmap_id | rating | text | address | avg_rating | num_of_reviews | category | median_income | wealth_group | business_responds |
|----------|--------|------|----------|------------|----------------|----------|---------------|--------------|------------------|
| 0x0:0x9edcb14b0cf1ec04 | 5.0 | The whole diving experience was amazing... | Dive Oahu, 609 KeaI St, Honolulu, HI 96813 | 4.2 | 278 | SCUBA instructor | 79180.0 | Low | True |
| 0x0:0x9edcb14b0cf1ec04 | 5.0 | Alex and Morgan were phenomenal teachers... | Dive Oahu, 609 KeaI St, Honolulu, HI 96813 | 4.2 | 278 | SCUBA instructor | 79180.0 | Low | True |
| 0x0:0x9edcb14b0cf1ec04 | 5.0 | I did my scuba diving certification... | Dive Oahu, 609 KeaI St, Honolulu, HI 96813 | 4.2 | 278 | SCUBA instructor | 79180.0 | Low | True |
| 0x0:0x9edcb14b0cf1ec04 | 5.0 | Great experience great dive guides... | Dive Oahu, 609 KeaI St, Honolulu, HI 96813 | 4.2 | 278 | SCUBA instructor | 79180.0 | Low | True |
| 0x0:0x9edcb14b0cf1ec04 | 5.0 | Excellent instruction and instructors... | Dive Oahu, 609 KeaI St, Honolulu, HI 96813 | 4.2 | 278 | SCUBA instructor | 79180.0 | Low | True |

After cleaning, the dataframe has 1475175 rows and 30 columns.

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

The distribution of business ratings is remarkably similar across income groups. Most businesses, regardless of the wealth of the surrounding region, receive ratings between 4.0 and 4.8 stars, with peaks near 4.5 stars. While businesses located in higher-income regions appear to have slightly higher ratings on average, the substantial overlap between distributions suggests that regional income is not a major determinant of customer ratings.

## Interesting Aggregates

I then create a pivot table displaying the average rating for businesses of a certain price level in neighborhoods with different median incomes, to investigate whether wealthier neighborhoods are less averse to paying high prices than less wealthy ones.

| wealth_group | $ | $$ | $$$ | $$$$ |
|-------------|----|----|-----|------|
| Low | 4.21 | 4.35 | 4.42 | 4.51 |
| High | 4.18 | 4.29 | 4.39 | 4.47 |

# Step 3: Assessment of Missingness

We now perform a missingness analysis. Missingness is classified into 4 types: Missing Completely at Random, Missing at Random, Missing Not at Random, and Missing by Design. Missing Completely at Random (MCAR) is a situation where the probability of data being missing is completely independent of both observed and unobserved data. It is purely accidental. Missing at Random (MAR) is where the missingness is systematically related to observed data but not the unobserved data. Missing Not at Random (MNAR) is when the missingness depends on the unobserved data itself. Missing by Design is data that is missing due to a logical design in the dataset, rather than being lost. In our dataset, there are likely 3 columns missing by design: `pics`, `text`, and `resp` are NMAR as many reviews do not contain pictures, review text, or have no responses. To analyse the remaining missingnes, we construct a pivot table first showing the proportion of reviews with missing ratings across review count groups.

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
| 0 | 11,449 |
| 1 | 9,411 |
| 2 | 53 |

Here, we see almost every business has at most 1 missing rating, with a few missing 2. We now will investigate the characteristics of these missing rows. Sometimes, google review datasets have "summary rows", which have the rest of the columns showing NaN as well. we check if this could be the case visually by randomly sampling 10 rows, then empirically by calculating the porportion of rows for which this is true.

| Index | rating | text | resp |
|--------:|-------:|------|------|
| 1447457 | NaN | NaN | NaN |
| 277750 | NaN | NaN | NaN |
| 1196273 | NaN | NaN | NaN |
| 513087 | NaN | NaN | NaN |
| 218274 | NaN | NaN | NaN |
| 265980 | NaN | NaN | NaN |
| 1359875 | NaN | NaN | NaN |
| 129855 | NaN | NaN | NaN |
| 461941 | NaN | NaN | NaN |
| 290118 | NaN | NaN | NaN |

`proportion of missing rows where all other columns are also missing: 1.0`

This appears to be true. This pattern strongly suggests that the missing values arise from the structure of the dataset or the merge process rather than from users selectively omitting ratings. Consequently, the missingness appears to be structural (MD) rather than MCAR, MAR, or NMAR in the traditional sense.

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

`p value: 0.09`

As shown above, the p value is not very significant, indicating that we fail to reject the null hypothesis here. Additionally, the observed difference of 0.0220 stars is not immense, indicating at the total group level that there is no such relationship present.

# Step 5: Framing a Prediction Problem

So far, our analysis shows wealthier regions aren't really associated with higher ratings. however, we do seem to see that number of reviews is highly variable, and could have some relationship with wealth groups. Furthermore, in accrodance with our ;ine of thinking, it is a good metric for customer engagement and could service our question of whether or not better customer ngagement is recieved by businesses in wealthier neighborhoods. We therefore run an additional hypothesis test:

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

p value: 0.006

As shown, these results are much more significant and more promising. We therefore approach a refined goal to model: Can we predict how much customer engagement a business receives based on its characteristics and the socioeconomic characteristics of its surrounding region?

# Step 6: Baseline Model

As a baseline, I fit a linear regression model to predict the number of reviews a business receives using two simple features: the median income of the surrounding area and the business's price category. Since price is categorical, it was one-hot encoded before fitting the model.

The model achieved an R² of 0.0085 on the training set, indicating that less than 1% of the variation in review counts can be explained by these predictors. This suggests that neither local income levels nor price category are strong determinants of the number of reviews a business receives.

To evaluate predictive performance, I computed the root mean squared error (RMSE). The training RMSE was 1879.10 reviews, while the testing RMSE was 1874.85 reviews. Because the training and testing errors are nearly identical, there is little evidence of overfitting. Instead, the model appears to underfit the data, failing to capture important factors that influence customer engagement.

Overall, this baseline demonstrates that simple demographic and pricing information alone is insufficient for accurately predicting review counts. More informative business characteristics and textual features are likely needed to improve predictive performance.

# Step 7: Final Model
