# Project 1: SQL E-commerce Data Exploring
## 1. Project overview
The eCommerce dataset is stored in a public Google BigQuery dataset. This dataset contains information about user sessions on a website collected from Google Analytics in 2017.

The project was conducted to assist Sales and Marketing Manager to have an outlook on the business situation and marketing efficiency by analysing performance of the website, in terms of Revenue, Bounce rate, The patterns of user behaviour, and a variety of other indicators. 

To extract necessary data from the dataset, Google BigQuery was used in executing SQL queries. 

## 2. The detailed project objectives
- Overview of website activity
- Bounce rate analysis
- Revenue analysis
- Transactions analysis
- Products analysis
## 3. Data preparation
### 3.1. Import raw data
To access data from the data warehouse of Google BigQuery, follow these steps: 
- Create a new project in Google Cloud Platform
- Navigate to the BigQuery console and select your newly created project.
- Select "Add Data" in the navigation panel and then "Search a project".
- Enter the project ID "bigquery-public-data.google_analytics_sample.ga_sessions" and click "Enter".
- Click on the "ga_sessions_" table to open it.
### 3.2. Data dictionary
The dataset dictionary was present fully in this link:
https://support.google.com/analytics/answer/3437719?hl=en
| Field Name               | Data Type | Description                                                                                                                                                                                                                                                                                                                                                                                |
|--------------------------|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| clientId                 | STRING    | Unhashed version of the Client ID for a given user associated with any given visit/session.                                                                                                                                                                                                                                                                                                |
| fullVisitorId            | STRING    | The unique visitor ID.                                                                                                                                                                                                                                                                                                                                                                     |
| visitorId                | NULL      | This field is deprecated. Use "fullVisitorId" instead.                                                                                                                                                                                                                                                                                                                                     |
| userId                   | STRING    | Overridden User ID sent to Analytics.                                                                                                                                                                                                                                                                                                                                                      |
| visitNumber              | INTEGER   | The session number for this user. If this is the first session, then this is set to 1.                                                                                                                                                                                                                                                                                                     |
| visitId                  | INTEGER   | An identifier for this session. This is part of the value usually stored as the _utmb cookie. This is only unique to the user. For a completely unique ID, you should use a combination of fullVisitorId and visitId.                                                                                                                                                                      |
| visitStartTime           | INTEGER   | The timestamp (expressed as POSIX time).                                                                                                                                                                                                                                                                                                                                                   |
| date                     | STRING    | The date of the session in YYYYMMDD format.                                                                                                                                                                                                                                                                                                                                                |
| totals                   | RECORD    | This section contains aggregate values across the session.                                                                                                                                                                                                                                                                                                                                 |
| totals.bounces           | INTEGER   | Total bounces (for convenience). For a bounced session, the value is 1, otherwise it is null.                                                                                                                                                                                                                                                                                              |
| totals.hits              | INTEGER   | Total number of hits within the session.                                                                                                                                                                                                                                                                                                                                                   |
| totals.newVisits         | INTEGER   | Total number of new users in session (for convenience). If this is the first visit, this value is 1, otherwise it is null.                                                                                                                                                                                                                                                                 |
| totals.pageviews         | INTEGER   | Total number of pageviews within the session.                                                                                                                                                                                                                                                                                                                                              |
| totals.screenviews       | INTEGER   | Total number of screenviews within the session.                                                                                                                                                                                                                                                                                                                                            |
| totals.sessionQualityDim | INTEGER   | An estimate of how close a particular session was to transacting, ranging from 1 to 100, calculated for each session. A value closer to 1 indicates a low session quality, or far from transacting, while a value closer to 100 indicates a high session quality, or very close to transacting. A value of 0 indicates that Session Quality is not calculated for the selected time range. |
### 3.3. Data statistics explore
- The number of rows in July, 2017
```
SELECT COUNT(fullVisitorId) row_num,
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*
```
| row_num |
|---------|
| 71812   |
- The number of rows in 2017
```
SELECT COUNT(fullVisitorId) row_num,
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
```
| row_num |
|---------|
| 467260  |
- The number and percentage of visit by month
```
SELECT EXTRACT(MONTH FROM PARSE_DATE("%Y%m%d",date)) month
,COUNT(*) AS counts
,ROUND((COUNT(*)/(SELECT COUNT(*) 
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`))*100,1) pct
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
GROUP BY EXTRACT(MONTH FROM PARSE_DATE("%Y%m%d",date))
```
| month | counts | pct  |
|-------|--------|------|
| 6     | 63578  | 13.6 |
| 3     | 69931  | 15.0 |
| 8     | 2556   | 0.5  |
| 2     | 62192  | 13.3 |
| 4     | 67126  | 14.4 |
| 1     | 64694  | 13.8 |
| 7     | 71812  | 15.4 |
| 5     | 65371  | 14.0 |
- UNNEST hits and products
```
SELECT date, 
fullVisitorId,
eCommerceAction.action_type,
product.v2ProductName,
product.productRevenue,
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
UNNEST(hits) AS hits,
UNNEST(hits.product) as product
```
| date     | fullVisitorId       | action_type | v2ProductName                         | productRevenue |
|----------|---------------------|-------------|---------------------------------------|----------------|
| 20170712 | 4080810487624198636 | 1           | YouTube Custom Decals                 |                |
| 20170712 | 4080810487624198636 | 2           | YouTube Custom Decals                 |                |
| 20170712 | 7291695423333449793 | 1           | Keyboard DOT Sticker                  |                |
| 20170712 | 7291695423333449793 | 2           | Keyboard DOT Sticker                  |                |
| 20170712 | 3153380067864919818 | 2           | Google Baby Essentials Set            |                |
| 20170712 | 3153380067864919818 | 1           | Google Baby Essentials Set            |                |
| 20170712 | 5615263059272956391 | 0           | Android Lunch Kit                     |                |
| 20170712 | 5615263059272956391 | 0           | Android Rise 14 oz Mug                |                |
| 20170712 | 5615263059272956391 | 0           | Android Sticker Sheet Ultra Removable |                |
| 20170712 | 5615263059272956391 | 0           | Windup Android                        |                |
## 4. Data analysing
### 4.1. Calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)
```
SELECT 
    FORMAT_DATE("%Y%m",PARSE_DATE("%Y%m%d",date)) month_extract
    ,SUM(totals.visits) visits
    ,SUM(totals.pageviews) pageviews
    ,SUM(totals.transactions) transactions
    ,ROUND(SUM(totals.totalTransactionRevenue)/POW(10,6),2) revenue
   FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
   WHERE _table_suffix BETWEEN '0101' AND '0331'
   GROUP BY month_extract
```
| month  | visits | pageviews | transactions | revenue   |
|--------|--------|-----------|--------------|-----------|
| 201701 | 64694  | 257708    | 713          | 106248.15 |
| 201702 | 62192  | 233373    | 733          | 116111.6  |
| 201703 | 69931  | 259522    | 993          | 150224.7  |

The table provides the overview of the website performance across three first months in 2017, in terms of visits, pageviews, transactions, and revenue. Some insights that might be obtained would be:

- **Traffic and User engagement:** The number of visits and pageviews increases over time. In regard of visit, 3 months witnesses a rise from 64694 (January) to 69931 (March), indicating the growing website interest. Simultaneously, the same patterns can be seen in the number of pageviews suggests that users tended to explore multiple pages in their visits, potentially, being interested in website's contents.
  
- **Conversion and Revenue:** The number of transactions and total revenue also increased over 3 months. Along with the rise in the number of visits and pageviews, those might demonstrate the growing in users engagement with the website, which resulted in their transactions, contributing to company's revenue. The efficiency in customers' conversion also could be seen in the rapidly jump in revenue from January (106248.15) to March (150224.7)
  
- **Seasonal or Marketing Influence:** The improvement of both traffic and revenue potentially indicated the impact of seasonal factors such as New Year Events, Marketing campaigns, or other optimisations strategies.
  
- **Future research:** The data that had been considered showed a very positive sign in the performance of website. However, it was only some aspects of a bigger picture. To have a better and more comprehensive perspective, it needed to incorporate more data such as users' journey or exit points. Furthermore, considering the current Marketing activities, Operation strategies, or any other changes that had been made during this perios would also assist in obtaining clearer insights. 

### 4.2. Bounce rate per traffic source (order by total_visit DESC)
```
select 
    trafficSource.source as source,
    sum(totals.visits) as total_visits, 
    sum(totals.bounces) as total_no_of_bounces, 
    sum(totals.bounces)/sum(totals.visits)*100 as bounce_rate

from `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
group by trafficSource.source
order by total_visits desc
```
| source                      | total_visits | total_no_of_bounces | bounce_rate |
|-----------------------------|--------------|---------------------|-------------|
| google                      | 38400        | 19798               | 51.56       |
| (direct)                    | 19891        | 8606                | 43.27       |
| youtube.com                 | 6351         | 4238                | 66.73       |
| analytics.google.com        | 1972         | 1064                | 53.96       |
| Partners                    | 1788         | 936                 | 52.35       |
| m.facebook.com              | 669          | 430                 | 64.28       |
| google.com                  | 368          | 183                 | 49.73       |
| dfa                         | 302          | 124                 | 41.06       |
| sites.google.com            | 230          | 97                  | 42.17       |
| facebook.com                | 191          | 102                 | 53.4        |
| reddit.com                  | 189          | 54                  | 28.57       |
| qiita.com                   | 146          | 72                  | 49.32       |
| baidu                       | 140          | 84                  | 60          |
| quora.com                   | 140          | 70                  | 50          |
| bing                        | 111          | 54                  | 48.65       |
| mail.google.com             | 101          | 25                  | 24.75       |
| yahoo                       | 100          | 41                  | 41          |
| blog.golang.org             | 65           | 19                  | 29.23       |
| l.facebook.com              | 51           | 45                  | 88.24       |
| groups.google.com           | 50           | 22                  | 44          |
| t.co                        | 38           | 27                  | 71.05       |
| google.co.jp                | 36           | 25                  | 69.44       |
| m.youtube.com               | 34           | 22                  | 64.71       |
| dealspotr.com               | 26           | 12                  | 46.15       |
| productforums.google.com    | 25           | 21                  | 84          |
| ask                         | 24           | 16                  | 66.67       |
| support.google.com          | 24           | 16                  | 66.67       |
| int.search.tb.ask.com       | 23           | 17                  | 73.91       |
| optimize.google.com         | 21           | 10                  | 47.62       |
| docs.google.com             | 20           | 8                   | 40          |
| lm.facebook.com             | 18           | 9                   | 50          |
| l.messenger.com             | 17           | 6                   | 35.29       |
| adwords.google.com          | 16           | 7                   | 43.75       |
| duckduckgo.com              | 16           | 14                  | 87.5        |
| google.co.uk                | 15           | 7                   | 46.67       |
| sashihara.jp                | 14           | 8                   | 57.14       |
| lunametrics.com             | 13           | 8                   | 61.54       |
| search.mysearch.com         | 12           | 11                  | 91.67       |
| tw.search.yahoo.com         | 10           | 8                   | 80          |
| outlook.live.com            | 10           | 7                   | 70          |
| phandroid.com               | 9            | 7                   | 77.78       |
| connect.googleforwork.com   | 8            | 5                   | 62.5        |
| plus.google.com             | 8            | 2                   | 25          |
| m.yz.sm.cn                  | 7            | 5                   | 71.43       |
| google.co.in                | 6            | 3                   | 50          |
| search.xfinity.com          | 6            | 6                   | 100         |
| google.ru                   | 5            | 1                   | 20          |
| online-metrics.com          | 5            | 2                   | 40          |
| hangouts.google.com         | 5            | 1                   | 20          |
| s0.2mdn.net                 | 5            | 3                   | 60          |
| m.sogou.com                 | 4            | 3                   | 75          |
| in.search.yahoo.com         | 4            | 2                   | 50          |
| googleads.g.doubleclick.net | 4            | 1                   | 25          |
| away.vk.com                 | 4            | 3                   | 75          |
| getpocket.com               | 3            |                     |             |
| m.baidu.com                 | 3            | 2                   | 66.67       |
| siliconvalley.about.com     | 3            | 2                   | 66.67       |
| msn.com                     | 2            | 1                   | 50          |
| google.it                   | 2            | 1                   | 50          |
| google.co.th                | 2            | 1                   | 50          |
| wap.sogou.com               | 2            | 2                   | 100         |
| calendar.google.com         | 2            | 1                   | 50          |
| github.com                  | 2            | 2                   | 100         |
| plus.url.google.com         | 2            |                     |             |
| myactivity.google.com       | 2            | 1                   | 50          |
| centrum.cz                  | 2            | 2                   | 100         |
| search.1and1.com            | 2            | 2                   | 100         |
| uk.search.yahoo.com         | 2            | 1                   | 50          |
| au.search.yahoo.com         | 2            | 2                   | 100         |
| m.sp.sm.cn                  | 2            | 2                   | 100         |
| google.cl                   | 2            | 1                   | 50          |
| moodle.aurora.edu           | 2            | 2                   | 100         |
| amp.reddit.com              | 2            | 1                   | 50          |
| newclasses.nyu.edu          | 1            |                     |             |
| google.es                   | 1            | 1                   | 100         |
| google.ca                   | 1            |                     |             |
| malaysia.search.yahoo.com   | 1            | 1                   | 100         |
| kidrex.org                  | 1            | 1                   | 100         |
| gophergala.com              | 1            | 1                   | 100         |
| aol                         | 1            |                     |             |
| google.nl                   | 1            |                     |             |
| kik.com                     | 1            | 1                   | 100         |
| earth.google.com            | 1            |                     |             |
| ph.search.yahoo.com         | 1            |                     |             |
| web.mail.comcast.net        | 1            | 1                   | 100         |
| google.bg                   | 1            | 1                   | 100         |
| news.ycombinator.com        | 1            | 1                   | 100         |
| es.search.yahoo.com         | 1            | 1                   | 100         |
| it.pinterest.com            | 1            | 1                   | 100         |
| mx.search.yahoo.com         | 1            | 1                   | 100         |
| images.google.com.au        | 1            | 1                   | 100         |
| search.tb.ask.com           | 1            |                     |             |
| arstechnica.com             | 1            |                     |             |
| web.facebook.com            | 1            | 1                   | 100         |
| online.fullsail.edu         | 1            | 1                   | 100         |
| google.com.br               | 1            |                     |             |
| suche.t-online.de           | 1            | 1                   | 100         |

Bounce rate is defined as the percentage of visitors that leave a webpage without taking an action, such as clicking on a link, filling out a form, or making a purchase.

This table showed the overview of website traffic, in regarding to different source, and the metrics that helped to analyse the level of engagement in the users as well as their behaviour. Those factors that were examined: source, total_visits, total_no_of_bounces, bounce_rate.

#### 4.2.1 Sources with high number of visits

Based on the table, sources that had the highest total number of visits are Google, (direct), Youtube, analytics.google.com, Partners, m.facebook.com. The same subsequency occured in terms of number of bounce. For those sources, the bounce rates were respectively 51.56%, 43.27%, 66.73%, 53.96%, 52.35%, 64.28%. These top sources commonly had bounce rate ranging from around 40-65%. 

#### 4.2.2 Sources with low number of visits

On the other hand, suche.t-online.de had the lowest number of visits (only 1) but the highest percentage of users who leave without any futher action (100%). The similar situation happened with online.fullsail.edu, web.facebook.com, images.google.com.au, mx.search.yahoo.com, it.pinterest.com, es.search.yahoo.com, news.ycombinator.com, google.bg, web.mail.comcast.net, kik.com, gophergala.com, kidrex.org, malaysia.search.yahoo.com, google.es. This might indicated that either these websites did not contain the content they were looking for or the landing pages were not appealing enough for them to stay and explore further. 

#### 4.2.3 Sources with low bounce rate

Reddit , Mail.google.com, Google.ru and Hangouts.google.com were 4 sources with the lowest bounce rate, ranging from 20-30%. However, besides Reddit and Mail.google.com, the other two sources only had a very modest number of visits (5 each). Thus, it might be biased if any conclusions would be drawn from these two sources. For Reddit and Mail.google.com, the low bounce rate might indicate that the content on these platforms were interesting to some extent to their users, or the marketing campaigns that were implemented via Gmail effectively targeted the potential customers. 

#### 4.2.4 Social media performace

Facebook.com and M.facebook.com were 2 link that directed to Facebook platform. With total visits of 191 and 669, respectively, their bounce rate were 53.4% and 64.28%, which suggested average engagement from Facebook users. In addition, more than 64% of mobile facebook (m.facebook.com) might showed the potential issues with experience of  mobile Facebook users or content relevancy.

#### 4.2.5 Conclusion

However, while bounce rate could tell some valuable insights of user behaviour and their engagement in the content on website, it was also necessary to assess thoroughly the context of those sources and user action, to draw a comprehensive conclusion about the meaning of bounce rate. 


### 4.3. Revenue by traffic source by week, by month in June 2017
### 4.4. Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017
### 4.5. Average number of transactions per user that made a purchase in July 2017
### 4.6. Average amount of money spent per session. Only include purchaser data in July 2017
### 4.7. Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017
### 4.8. Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017.

