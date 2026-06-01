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
