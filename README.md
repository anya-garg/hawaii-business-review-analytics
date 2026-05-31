# Mo' Money Mo' Problems: Do Businesses in Wealthier Regions Recieve Worse Reviews?

by Anya Garg

# Introduction

Hawaii, in recent years, has seen a burgeoning tourism industry that has drawn a wedge between wealthier and less wealthy regions, leading to stark discrepancies in the types of goods and services offered across regions, who consumes them, and how consumers feel about the product. Naturally, a question was raised: do businesses in more affluent regions recieve better ratings or engagement? and which features on the whole lead to the most positive engagement?

To begin, we load in data from Google Maps containing reviews of various locations in Hawaii. It was originally scraped and used by the authors of these papers: https://aclanthology.org/2022.acl-long.426.pdf, https://arxiv.org/pdf/2207.00422. The full dataset is quite long, so we will only be using the 10-core one.

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

## Step 2: Data Cleaning and Exploratory Data Analysis

In order to perform meaningful analysis, I had to merge several datasets together. This combining and cleaning proccess was conducted as follows:
1. Perform an outer merge on the `review` dataset and `meta` dataset together on the `gmap_id` column
2. Upload another dataset tracking which businesses are in wealthier and less wealthy regions, from the American Community Survey in Hawaii (https://data.census.gov/table/ACSDP5Y2018.DP03?g=040XX00US15&layer=VT_2018_140_00_PY_D1&hidePreview=false&cid=DP03_0001E&vintage=2018&tid=ACSDP5Y2018.DP03), and filter to only the neccessary columns (`GEOID`, `NAME`, `DP03_0062E` (representing median income))
3. To link them together, upload a census tract shapefile to merge on the `GEOID` column of the ACS dataset (https://geoportal.hawaii.gov/datasets/HiStateGIS::2020-census-tracts/explore?location=31.072143%2C54.064055%2C3).
4. Converted the merged set into a CRS (Coordinate Reference System) and join with first dataset with ```gpd.sjoin(hawaii_gpd, tracts_wealth, how="left", predicate="within")```
5. select only important columns and rename

