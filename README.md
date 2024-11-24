# Ad Campaign Analysis with SQL and Looker Studio

## Introduction
This repository contains SQL scripts and Google Looker Studio dashboards designed to analyze the performance of advertising campaigns on both Google Ads and Facebook Ads. The goal of this project is to provide insights into campaign effectiveness, identify areas for improvement, and inform future marketing decisions.

## Data Sources
The data for this project was extracted from a PostgreSQL database containing four primary tables:
* **Facebook Ads:** `facebook_ads_basic_daily`, `facebook_adset`, `facebook_campaign`
* **Google Ads:** `google_ads_basic_daily`
  
![Facebook_and_Google_ads](1.png)

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

 
