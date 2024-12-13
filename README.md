# Ad Campaign Analysis with SQL and Looker Studio

## Introduction
This repository contains SQL scripts and Google Looker Studio dashboards designed to analyze the performance of advertising campaigns on both Google Ads and Facebook Ads. The goal of this project is to provide insights into campaign effectiveness, identify areas for improvement, and inform future marketing decisions.

## Data Sources
The data for this project was extracted from a PostgreSQL database containing four primary tables:
* **Facebook Ads:** `facebook_ads_basic_daily`, `facebook_adset`, `facebook_campaign`
* **Google Ads:** `google_ads_basic_daily`
  
![Facebook_and_Google_ads](Images/1.png)

## Tools
DBeaver, Looker Studio

## SQL Queries

### Query 1. Facebook Campaign Metrics Daily

~~~SQL
SELECT
	f.ad_date,
	f.campaign_id,
	sum (f.spend) AS spend,
	sum (f.impressions) AS impressions,
	sum (f.clicks) AS clicks,
	sum (f.value) AS value,
	sum (f.spend) / sum (f.clicks) AS cpc,
	round((sum(f.spend) ::NUMERIC / sum(f.impressions)) * 1000,
	2) AS cpm,
	round((sum(f.clicks) ::NUMERIC / sum(f.impressions)) * 100,
	2) AS ctr,
	round((((sum(f.value) - sum(f.spend)) ::NUMERIC / sum(f.spend))) * 100,
	2) AS romi
FROM
	facebook_ads_basic_daily f
WHERE 
	f.campaign_id IS NOT NULL
	AND f.clicks > 0
	AND f.impressions > 0
	AND f.spend > 0
	AND f.impressions > 0
GROUP BY 
	f.ad_date,
	f.campaign_id
ORDER BY
	f.ad_date DESC;
 ~~~

This SQL query analyzes Facebook ad performance data from the `facebook_ads_basic_daily` table. It calculates key metrics for each day and campaign:

1. **Total Spend:** The total amount spent on ads.
2. **Total Impressions:** The total number of times ads were shown.
3. **Total Clicks:** The total number of clicks on ads.
4. **Total Value:** The total value generated by the ads.
5. **CPC (Cost Per Click):** The average cost per click.
6. **CPM (Cost Per Mille):** The cost per 1000 impressions.
7. **CTR (Click-Through Rate):** The percentage of impressions that resulted in clicks.
8. **ROMI (Return on Marketing Investment):** The return on investment for the ads, expressed as a percentage.

The query filters out records with zero clicks, impressions, or spend to avoid division by zero errors. It then groups the results by date and campaign ID and sorts them by date in descending order.

### Query 2. Daily Ad Performance by Platform 

~~~SQL
WITH combined_ads AS (
SELECT
        ad_date,
	'Facebook Ads' AS media_source,
	spend,
	impressions,
	reach,
	clicks,
	leads,
	value
FROM
	facebook_ads_basic_daily f
WHERE
        ad_date IS NOT NULL
UNION ALL
SELECT
	ad_date,
	'Google Ads' AS media_source,
	spend,
	impressions,
	reach,
	clicks,
	leads,
	value
FROM
	google_ads_basic_daily g
WHERE
	ad_date IS NOT NULL
)
SELECT
	ad_date,
	media_source,
	sum (spend) / sum (clicks) AS CPC,
	round((sum(spend)::NUMERIC / sum(impressions)) * 1000,
	2) AS CPM,
	round((sum(clicks)::NUMERIC / sum(impressions)) * 100,
	2) AS CTR,
	round((((sum(value) - sum(spend))::NUMERIC / sum(spend))) * 100,
	2) AS ROMI
FROM
	combined_ads
WHERE
	reach > 0
	AND leads > 0
	AND clicks > 0
	AND impressions > 0
	AND spend > 0
	AND impressions > 0
GROUP BY
	ad_date,
	media_source
ORDER BY
	ad_date,
	media_source DESC;
~~~

This SQL query combines data from Facebook Ads and Google Ads into a single dataset, calculates key performance metrics, and filters out irrelevant data.

**Here's a breakdown:**

1. **Data Combination:**
   - It merges data from `facebook_ads_basic_daily` and `google_ads_basic_daily` tables into a single Common Table Expression (CTE) named `combined_ads`.
   - It adds a `media_source` column to differentiate between the two platforms.

2. **Metric Calculation:**
   - It calculates the following metrics for each date and media source:
     - **CPC (Cost Per Click):** Average cost per click.
     - **CPM (Cost Per Mille):** Cost per 1000 impressions.
     - **CTR (Click-Through Rate):** Percentage of impressions that resulted in clicks.
     - **ROMI (Return on Marketing Investment):** Percentage return on investment.

3. **Data Filtering:**
   - It filters the data to exclude records with zero values for `reach`, `leads`, `clicks`, `impressions`, and `spend` to avoid division by zero errors.

4. **Grouping and Sorting:**
   - It groups the results by `ad_date` and `media_source`.
   - It sorts the results by `ad_date` in descending order and then by `media_source` in descending order.
  
### Query 3. Combined Ad Set Daily Performance

~~~SQL
WITH combi_data AS (
SELECT
	fbd.ad_date,
	'Facebook Ads' AS media_source,
	fc.campaign_name,
	fa.adset_name,
	fbd.spend,
	fbd.impressions,
	fbd.reach,
	fbd.clicks,
	fbd.leads,
	fbd.value
FROM
	facebook_ads_basic_daily fbd
LEFT JOIN facebook_adset fa ON
	fbd.adset_id = fa.adset_id
LEFT JOIN facebook_campaign fc ON
	fbd.campaign_id = fc.campaign_id
WHERE
        fbd.ad_date IS NOT NULL
UNION ALL
SELECT
	g.ad_date,
	'Google Ads' AS media_source,
	g.campaign_name,
	g.adset_name,
	g.spend,
	g.impressions,
	g.reach,
	g.clicks,
	g.leads,
	g.value
FROM
	google_ads_basic_daily g
WHERE
	g.ad_date IS NOT NULL	
)
SELECT
	ad_date,
	media_source,
	campaign_name,
	adset_name,
	sum (spend) AS total_spend,
	sum (impressions) AS total_impressions,
	sum (clicks) AS total_clicks,
	sum (value) AS total_value
FROM
	combi_data
GROUP BY
	ad_date,
	media_source,
	campaign_name,
	adset_name
ORDER BY 
	ad_date DESC;
~~~

This SQL query combines data from Facebook Ads and Google Ads into a single dataset, providing a detailed view of campaign and ad set performance.

**Here's a breakdown:**

1. **Data Combination:**
   - It merges data from `facebook_ads_basic_daily`, `facebook_adset`, and `facebook_campaign` tables for Facebook Ads.
   - It merges data from the `google_ads_basic_daily` table for Google Ads.
   - The resulting dataset includes information about ad date, media source, campaign name, ad set name, spend, impressions, reach, clicks, leads, and value.

2. **Data Aggregation:**
   - It groups the combined data by `ad_date`, `media_source`, `campaign_name`, and `adset_name`.
   - It calculates the total spend, impressions, clicks, and value for each group.

3. **Result Ordering:**
   - It sorts the results by `ad_date` in descending order.

**In essence, this query provides a consolidated view of advertising performance across both platforms, allowing for detailed analysis and comparison of campaigns and ad sets.**

### Query 4. For Google Looker Studio

~~~SQL
WITH combined_ads AS (
SELECT
	ad_date,
	'Facebook Ads' AS media_source,
	spend,
	impressions,
	reach,
	clicks,
	leads,
	value
FROM
	facebook_ads_basic_daily f
WHERE
        ad_date IS NOT NULL
UNION ALL
SELECT
	ad_date,
	'Google Ads' AS media_source,
	spend,
	impressions,
	reach,
	clicks,
	leads,
	value
FROM
	google_ads_basic_daily g
WHERE
	ad_date IS NOT NULL
)
SELECT
	ad_date,
	media_source,
	sum (spend) AS spend,
	sum (impressions) AS impressions,
	sum (clicks) AS clicks,
	sum (value) AS value
FROM
	combined_ads
WHERE
	reach > 0
	AND leads > 0
	AND clicks > 0
	AND impressions > 0
	AND spend > 0
	AND impressions > 0
GROUP BY
	ad_date,
	media_source
ORDER BY
	ad_date,
	media_source DESC;
~~~

This SQL query combines data from Facebook Ads and Google Ads into a single dataset and calculates aggregated metrics for each day and media source. 

**Here's a breakdown:**

1. **Data Combination:**
   - Merges data from `facebook_ads_basic_daily` and `google_ads_basic_daily` tables.
   - Adds a `media_source` column to distinguish between the two platforms.

2. **Data Filtering:**
   - Filters out records with zero values for `reach`, `leads`, `clicks`, `impressions`, and `spend` to avoid division by zero errors.

3. **Metric Calculation:**
   - Calculates the total `spend`, `impressions`, `clicks`, and `value` for each day and media source.

4. **Result Ordering:**
   - Sorts the results by `ad_date` in descending order.

In essence, this query provides a consolidated view of advertising performance across both platforms, allowing for comparison and analysis of overall campaign performance.

Additional calculated fields have been created in Google Looker Studio:
1. Total Ad Spend or Ad Spend
2. CPC (Cost Per Click)
3. CPM (Cost Per Mille or Cost Per Thousand Impressions)
4. CTR (Click-Through Rate)
5. ROMI (Return on Marketing Investment)

Based on the query and calculated fields, three graphs were constructed.

1. **Total Ad Spend and ROMI Over Time**  
   Description: This chart shows the monthly advertising spend and the Return on Marketing Investment (ROMI) from November 2020 to November 2022.

   ![ad_spend](5.png)

2. **Count of Active Campaigns by Month**  
   Description: This chart tracks the number of active advertising campaigns for each month over the same period.

   ![active_cmp](6.png)

4. **Breakdown of Total Ad Spend by Campaign Type**  
   Description: This chart categorizes ad spend by campaign types such as "Expansion," "Lookalike," and others, along with key metrics like CPC, CPM, CTR, and ROMI.

   ![total](7.png)

### Query 5. Comprehensive Ad Campaign Performance Report with UTM Tracking

~~~SQL
WITH cte_facebook_ads AS (
SELECT
	fad.ad_date,
	fad.url_parameters,
	COALESCE(fad.spend,
	0) AS spend,
	COALESCE(fad.impressions,
	0) AS impressions,
	COALESCE(fad.reach,
	0) AS reach,
	COALESCE(fad.clicks,
	0) AS clicks,
	COALESCE(fad.leads,
	0) AS leads,
	COALESCE(fad.value,
	0) AS value
FROM
	facebook_ads_basic_daily AS fad
JOIN facebook_adset AS fas ON
	fad.adset_id = fas.adset_id
JOIN facebook_campaign AS fc ON
	fad.campaign_id = fc.campaign_id
),
cte_google_ads AS (
SELECT
	gad.ad_date,
	gad.url_parameters,
	COALESCE(gad.spend,
	0) AS spend,
	COALESCE(gad.impressions,
	0) AS impressions,
	COALESCE(gad.reach,
	0) AS reach,
	COALESCE(gad.clicks,
	0) AS clicks,
	COALESCE(gad.leads,
	0) AS leads,
	COALESCE(gad.value,
	0) AS value
FROM
	google_ads_basic_daily AS gad
),
cte_combined_ads AS (
SELECT
	ad_date,
	url_parameters,
	spend,
	impressions,
	reach,
	clicks,
	leads,
	value
FROM
	cte_facebook_ads
UNION ALL
SELECT
	ad_date,
	url_parameters,
	spend,
	impressions,
	reach,
	clicks,
	leads,
	value
FROM
	cte_google_ads
),
cte_parsed_data AS (
SELECT
	ad_date,
	spend,
	impressions,
	clicks,
	value,
	LOWER(NULLIF(SUBSTRING(url_parameters FROM 'utm_campaign=([^&]+)'), 'nan')) AS utm_campaign
FROM
	cte_combined_ads
)
SELECT
	ad_date,
	utm_campaign,
	SUM(spend) AS total_spend,
	SUM(impressions) AS total_impressions,
	SUM(clicks) AS total_clicks,
	SUM(value) AS total_value,
	CASE
		WHEN SUM(impressions) = 0 THEN 0
		ELSE ROUND((SUM(clicks) * 100.0 / SUM(impressions)),
		2)
	END AS ctr,
	CASE
		WHEN SUM(clicks) = 0 THEN 0
		ELSE ROUND((SUM(spend)::NUMERIC / SUM(clicks)),
		2)
	END AS cpc,
	CASE
		WHEN SUM(impressions) = 0 THEN 0
		ELSE ROUND((SUM(spend) * 1000.0 / SUM(impressions)),
		2)
	END AS cpm,
	CASE
		WHEN SUM(spend) = 0 THEN 0
		ELSE ROUND(((SUM(value) - SUM(spend))::NUMERIC / SUM(spend)) * 100.00,
		2)
	END AS romi
FROM
	cte_parsed_data
GROUP BY
	ad_date,
	utm_campaign
ORDER BY
	ad_date;
~~~

This SQL query combines data from Facebook Ads and Google Ads, extracts UTM campaign information, and calculates key performance metrics.

**Here's a breakdown:**

1. **Data Combination:**
   - Merges data from `facebook_ads_basic_daily` and `google_ads_basic_daily` tables.
   - Adds a `media_source` column to distinguish between the two platforms.

2. **Data Cleaning and Preparation:**
   - Uses `COALESCE` to replace null values with 0 for numerical columns.
   - Extracts the `utm_campaign` parameter from the `url_parameters` column.

3. **Metric Calculation:**
   - Calculates the following metrics for each `ad_date` and `utm_campaign`:
     - **Total Spend:** Total amount spent.
     - **Total Impressions:** Total number of impressions.
     - **Total Clicks:** Total number of clicks.
     - **Total Value:** Total value generated.
     - **CTR (Click-Through Rate):** Percentage of impressions that resulted in clicks.
     - **CPC (Cost Per Click):** Average cost per click.
     - **CPM (Cost Per Mille):** Cost per 1000 impressions.
     - **ROMI (Return on Marketing Investment):** Percentage return on investment.

4. **Data Grouping and Sorting:**
   - Groups the results by `ad_date` and `utm_campaign`.
   - Sorts the results by `ad_date`.

**In essence, this query provides a consolidated view of advertising performance across both platforms, allowing for detailed analysis of campaign performance based on UTM parameters.**

### Query 6. Month-over-Month Ad Metric Comparison

~~~SQL
WITH cte_facebook_ads AS (
SELECT
	fad.ad_date,
	fad.url_parameters,
	COALESCE(fad.spend,
	0) AS spend,
	COALESCE(fad.impressions,
	0) AS impressions,
	COALESCE(fad.reach,
	0) AS reach,
	COALESCE(fad.clicks,
	0) AS clicks,
	COALESCE(fad.leads,
	0) AS leads,
	COALESCE(fad.value,
	0) AS value
FROM
	facebook_ads_basic_daily AS fad
JOIN facebook_adset AS fas ON
	fad.adset_id = fas.adset_id
JOIN facebook_campaign AS fc ON
	fad.campaign_id = fc.campaign_id
),
cte_google_ads AS (
SELECT
	gad.ad_date,
	gad.url_parameters,
	COALESCE(gad.spend,
	0) AS spend,
	COALESCE(gad.impressions,
	0) AS impressions,
	COALESCE(gad.reach,
	0) AS reach,
	COALESCE(gad.clicks,
	0) AS clicks,
	COALESCE(gad.leads,
	0) AS leads,
	COALESCE(gad.value,
	0) AS value
FROM
	google_ads_basic_daily AS gad
),
cte_combined_ads AS (
SELECT
	ad_date,
	url_parameters,
	spend,
	impressions,
	reach,
	clicks,
	leads,
	value
FROM
	cte_facebook_ads
UNION ALL
SELECT
	ad_date,
	url_parameters,
	spend,
	impressions,
	reach,
	clicks,
	leads,
	value
FROM
	cte_google_ads
),
cte_parsed_data AS (
SELECT
	ad_date,
	spend,
	impressions,
	clicks,
	value,
	LOWER(NULLIF(SUBSTRING(url_parameters FROM 'utm_campaign=([^&]+)'), 'nan')) AS utm_campaign
FROM
	cte_combined_ads
),
cte_monthly_data AS (
SELECT
	DATE_TRUNC('month',
	ad_date) AS ad_month,
	utm_campaign,
	SUM(spend) AS total_spend,
	SUM(impressions) AS total_impressions,
	SUM(clicks) AS total_clicks,
	SUM(value) AS total_value,
	CASE
		WHEN SUM(impressions) = 0 THEN 0
		ELSE ROUND((SUM(clicks) * 100.0 / SUM(impressions)),
		2)
	END AS ctr,
	CASE
		WHEN SUM(clicks) = 0 THEN 0
		ELSE ROUND((SUM(spend)::NUMERIC / SUM(clicks)),
		2)
	END AS cpc,
	CASE
		WHEN SUM(impressions) = 0 THEN 0
		ELSE ROUND((SUM(spend) * 1000.0 / SUM(impressions)),
		2)
	END AS cpm,
	CASE
		WHEN SUM(spend) = 0 THEN 0
		ELSE ROUND(((SUM(value) - SUM(spend))::NUMERIC / SUM(spend)) * 100.00,
		2)
	END AS romi
FROM
	cte_parsed_data
GROUP BY
	ad_month,
	utm_campaign
),
cte_with_differences AS (
SELECT
	ad_month,
	utm_campaign,
	total_spend,
	total_impressions,
	total_clicks,
	total_value,
	ctr,
	cpc,
	cpm,
	romi,
	LAG(cpm) OVER (PARTITION BY utm_campaign
ORDER BY
	ad_month) AS prev_cpm,
	LAG(ctr) OVER (PARTITION BY utm_campaign
ORDER BY
	ad_month) AS prev_ctr,
	LAG(romi) OVER (PARTITION BY utm_campaign
ORDER BY
	ad_month) AS prev_romi
FROM
	cte_monthly_data
)
SELECT
	ad_month,
	utm_campaign,
	total_spend,
	total_impressions,
	total_clicks,
	total_value,
	ctr,
	cpc,
	cpm,
	romi,
	ROUND((cpm - prev_cpm) * 100.0 / prev_cpm,
	2) AS cpm_difference,
	ROUND((ctr - prev_ctr) * 100.0 / prev_ctr,
	2) AS ctr_difference,
	ROUND((romi - prev_romi) * 100.0 / prev_romi,
	2) AS romi_difference
FROM
	cte_with_differences
ORDER BY
	ad_month,
	utm_campaign;
~~~

This SQL query combines data from Facebook Ads and Google Ads, extracts UTM campaign information, calculates key performance metrics, and analyzes month-over-month changes in these metrics.

**Here's a breakdown:**

1. **Data Combination and Cleaning:**
   - Merges data from both platforms.
   - Extracts UTM campaign information from URL parameters.
   - Handles potential null values.

2. **Metric Calculation:**
   - Calculates key metrics like `total_spend`, `total_impressions`, `total_clicks`, `total_value`, `CTR`, `CPC`, `CPM`, and `ROMI` for each `ad_month` and `utm_campaign`.

3. **Month-over-Month Comparison:**
   - Uses the `LAG` window function to access previous month's values for each `utm_campaign`.
   - Calculates the percentage change in `CPM`, `CTR`, and `ROMI` compared to the previous month.

4. **Result Output:**
   - Presents the results, including the calculated metrics and month-over-month percentage changes, sorted by `ad_month` and `utm_campaign`.

**In essence, this query provides a detailed analysis of campaign performance over time, allowing you to identify trends and make data-driven decisions.**

## Ad Campaign Analysis: Key Findings and Optimization Recommendations

### Key Findings:

* **Growth in spending and ROMI in early 2021:** There was a significant increase in both overall advertising spending and ROMI in early 2021, indicating high campaign effectiveness during this period.
* **Stabilization and decline:** Following the peak in early 2021, there was a trend towards stabilization and even a decline in both spending and ROMI. This could be attributed to various factors such as seasonality, changing market conditions, or campaign optimizations.
* **Sharp decline in late 2022:** A sharp decrease in both spending and ROMI was observed in late 2022, possibly indicating significant changes in advertising strategy or external factors impacting campaign effectiveness.
* **CPC variation:** Cost per click (CPC) varied significantly, with lower values for "Discounts" and "Crazy discounts" campaigns and higher values for "Expansion" and "Brand" campaigns. This could be due to differing levels of competition in respective niches or targeting specifics.
* **CPM variation:** Cost per thousand impressions (CPM) also showed significant differences. Higher CPM values were typical for campaigns with a wide audience or premium placements.
* **CTR variation:** Click-through rate (CTR) varied depending on the campaign type and creatives. "Discounts" and "New items" campaigns had the highest CTRs, suggesting their attractiveness to users.
* **ROMI variation:** Return on marketing investment (ROMI) also varied significantly across different campaign types. "Promos" and "Trendy" campaigns demonstrated the highest ROMI values, indicating their high effectiveness.

### Recommendations for Optimization:
* **Analyze the sharp decline in late 2022:** Conduct a detailed analysis to identify the causes of this significant drop.
* **Compare across channels:** Compare the effectiveness of advertising campaigns across different channels (e.g., social media, search engines) to identify the most effective ones.
* **Test new strategies:** Continuously test new advertising strategies, audiences, and creatives to find optimal solutions.
* **Monitor competitors:** Keep an eye on competitors' actions and adjust your advertising strategy accordingly.
* **Utilize analytics tools:** Use analytics tools to collect and analyze data on campaign effectiveness.
* **Optimize spending:** Conduct a deeper analysis of low-ROMI campaigns to identify the reasons for low efficiency and take measures to optimize them (e.g., adjust bids, change creatives, refine target audience).
* **Focus on effective channels:** Pay more attention to high-ROMI campaigns like "Promos" and "Trendy," considering increasing their budgets.
* **Test new formats:** To improve campaign effectiveness, test new advertising formats, creatives, and target audiences.
* **Analyze seasonality:** Consider seasonal demand fluctuations and adjust advertising budgets accordingly.
* **Use analytics tools:** Use analytics tools to monitor campaign performance and identify trends for informed decision-making.

**In essence, the analysis suggests that while the company has a diverse range of advertising campaigns, there's room for optimization. By focusing on high-performing campaigns, testing new strategies, and continuously monitoring performance, the company can improve the overall effectiveness of its advertising efforts.**




   








 
