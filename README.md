# Strategic Customer Segmentation: RFM Analysis with Power BI

## Overview

Welcome to the Customer RFM (Recency, Frequency, Monetary) Analysis Project! This initiative provides comprehensive insights into customer behavior, engagement, and value, showcasing the power of RFM analysis. This project demonstrates how to drive targeted marketing strategies, enhance customer retention, and maximize business profitability by understanding and segmenting customers. Key features include the use of DAX expressions for precise calculations, interactive visualizations with dynamic filters and tooltips for an enhanced user experience, and a thorough data analysis with actionable recommendations. Additionally, a fictionalized timeline and roadmap outline the proposed steps for implementation.

## Table of Contents

- [Features](#features)
- [Data Source](#data-source)
- [Usage](#usage)
- [Screenshots](#screenshots)
- [Insights and Analysis](#insights-and-analysis)
- [Future Work](#future-work)
- [Contact](#contact)

## Features

- **Interactive Visualizations**: Engaging and interactive charts and graphs to deeply explore various customer segments.
- **RFM Segmentation**: Detailed breakdown of customers based on Recency, Frequency, and Monetary values.
- **Tooltips and Annotations**: Comprehensive explanations and insights are available through interactive tooltips on each page.
- **Dynamic Filters**: Slicers enabling data filtering based on R, F, and M scores for tailored analysis.
- **Comprehensive Analysis**: Dedicated pages for each aspect of RFM, including a thorough segment analysis, conclusions, and business recommendations.

## Data Source

Our data comes from retail store sales transactions available on Kaggle. [The anonymized dataset](https://www.kaggle.com/datasets/marian447/retail-store-sales-transactions?resource=download) includes 64.682 transactions of 5.242 SKU's sold to 22.625 customers during one year.

| Dataset Columns                       |
|---------------------------------------|
| Date of Sales Transaction             |
| Customer ID                           |
| Transaction ID                        |
| SKU Category ID                       |
| SKU ID                                |
| Quantity Sold                         |
| Sales Amount (Unit price times quantity. For unit price, please divide Sales Amount by Quantity.) |

We are also using an additional "Segment_Scores Table" that has the following columns:

| Dataset Columns        |
|------------------------|
| Segment                |
| Scores                 |
| Index                  |



## Usage

1. Open Power BI Desktop and load the .pbix file.
2. Explore the interactive dashboard to analyze customer segments.
3. Use the slicers to filter data based on different R, F, and M scores.
4. Hover over the tooltips to gain deeper insights into the data.


## Understanding RFM Analysis
RFM (Recency, Frequency, Monetary) analysis is a quantitative technique utilized to segment customers based on purchasing patterns. This method helps businesses pinpoint their most valuable customers and customize marketing strategies to maximize engagement and profitability.

The primary aim of RFM analysis is to enhance customer retention, boost sales, and improve overall business performance. By gaining insights into customer behavior, businesses can make informed decisions to foster stronger customer relationships. Read more on RFM analysis by [clicking here](https://www.putler.com/rfm-analysis/).

### Benefits of RFM Analysis:
- Identification of High-Value Customers: Recognizes the most profitable customers who contribute significantly to revenue.
- Optimization of Marketing Efforts: Helps in tailoring marketing campaigns to suit different customer segments, leading to more effective resource utilization.
- Improvement in Customer Retention and Loyalty: By understanding customer behavior, businesses can implement strategies to keep customers engaged and loyal.
- Revenue Increase Through Targeted Strategies: Employing focused marketing strategies on segmented customer groups can drive higher sales and revenue growth.

## RFM Analysis Process:
- **Recency:** Measures how recently a customer has made a purchase. Customers who have purchased recently are more likely to buy again compared to those who haven't purchased in a long time.
- **Frequency:** Tracks how often a customer makes a purchase. Frequent purchasers are typically more loyal and valuable.
- **Monetary:** Evaluates how much money a customer has spent. Higher-spending customers are generally more valuable to the business.

### How to Generate RFM Scores
To generate RFM scores, we need these four key parameters:
- **Unique Customer Identifier:** Use a unique identifier such as a customer name or ID.
- **Recency:** Determine the number of days since the customer's last purchase.
- **Frequency:** Calculate the total number of transactions made by the customer.
- **Monetary:** Compute the total amount spent by the customer, also known as the CLV (Customer Lifetime Value).
Each customer is then assigned an RFM score on a scale from 1 to 5 for each of these parameters, where for F and M scores, 1 indicates the lowest value and 5 indicates the highest, while the opposite is true for R scores. This scoring helps to segment customers based on their purchasing behavior and value to the business.


*Description of Segmentation*

![RFN Scores Description](images/rfm_explanation.webp)

*RFM Scores*

![RFM Scores](images/rfm_scores.webp)


### DAX (Data Analysis Expression) for RFM
1. We begin the analysis by calculating the 'R Value', in order to gauge customer's recency. To determine the ‘R value’, we need the most recent transaction date, so we’ll introduce a new measure named ‘last transaction date’.


```
-- Creating a new measure called 'last transaction date'
last transaction date =
MAXX(FILTER('scanner_data','scanner_data'[Customer_ID]='scanner_data'[Customer_ID]),'scanner_data'[Date])
```

2. We'll now establish new measures for the R-value, F-value, and M-value.


```
-- Creating a new measure called 'R value'
R value = DATEDIFF('scanner_data'[last transaction date],TODAY(),DAY)
-- Creating a new measure called 'F value'
F value = DISTINCTCOUNT('scanner_data'[Transaction_ID])
-- Creating a new measure called 'M value'
M value =
var TotalSales = SUM('scanner_data'[Sales_Amount])
var TotalQuantity = sum(scanner_data[Quantity])
Return 
DIVIDE (TotalSales,TotalQuantity,0)
```

3. Here we'll proceed by generating a new table named ‘RFM table’, using the following DAX formula to create it:


```
-- Creating a new table called 'RFM table'
RFM table = SUMMARIZE(
   'scanner_data','scanner_data'[Customer_ID],
   "R Value",[R Value],
   "F Value",[F Value],
   "M Value",[M Value])
```


- Once the ‘RFM table’ is generated, it appears as shown below.

*RFM Table Example*

![RFM Table Example](images/rfm_table_example.webp)



4. We will then create new columns ‘R Score’, ‘F Score’, and ‘M Score’, grouping each dataset by percentile.


```
-- Creating a new column called 'R Score'
R Score = 
SWITCH (
  TRUE (),
   [R value] <= PERCENTILE.INC ( 'RFM table'[R Value], 0.20 ), "5",    
   [R value] <= PERCENTILE.INC ( 'RFM table'[R Value], 0.40 ), "4", 
   [R Value] <= PERCENTILE.INC ( 'RFM table'[R Value], 0.60 ), "3", 
   [R value] <= PERCENTILE.INC ( 'RFM table'[R Value], 0.80 ), "2",
   "1"
       )
-- Creating a new column called 'F Score'
F Score =
SWITCH (
  TRUE (),
   [F value] <= PERCENTILE.INC ( 'RFM table'[F Value], 0.20 ), "1",    
   [F value] <= PERCENTILE.INC ( 'RFM table'[F Value], 0.40 ), "2", 
   [F Value] <= PERCENTILE.INC ( 'RFM table'[F Value], 0.60 ), "3", 
   [F value] <= PERCENTILE.INC ( 'RFM table'[F Value], 0.80 ), "4",
   "5"
       )
-- Creating a new column called 'M Score'
M Score =
SWITCH (
  TRUE (),
   [F value] <= PERCENTILE.INC ( 'RFM table'[F Value], 0.20 ), "1",    
   [F value] <= PERCENTILE.INC ( 'RFM table'[F Value], 0.40 ), "2", 
   [F Value] <= PERCENTILE.INC ( 'RFM table'[F Value], 0.60 ), "3", 
   [F value] <= PERCENTILE.INC ( 'RFM table'[F Value], 0.80 ), "4",
   "5"
       )
```


5. Next, form the new column ‘RFM’ by merging the columns ‘R Score’, ‘F Score’, and ‘M Score’.


```
-- To create a new column called 'RFM'
RFM = 'RFM table'[R Score]& 'RFM table'[F Score]&'RFM table'[M Score]
```


6. Lastly, establish a relationship between the ‘RFM table’ and the ‘Segment-Scores Table’, using the ‘RFM’ column in the RFM table and the ‘Scores’ column in the Segment-Scores table.

*Relationship between RFM table and Segment-Scores table*

![RFM Relationship](images/rfm_relationship.webp)


With the columns in place, you can now proceed to dashboard design! The following are screenshots of the various pages of the report:


## Screenshots

![Dashboard Overview](images/dashboard_overview.png)
*Dashboard Overview Page*

![RFM Dashboard](images/rfm_dashboard.png)
*RFM Dashboard*

![Customer Segmentation](images/customer_segmentation.png)
*Customer Segmentation*

![Recency Analysis](images/customer_recency_analysis.png)
*Recency Analysis Page*

![Frequency Analysis](images/customer_purchase_frequency_analysis.png)
*Frequency Analysis Page*

![Monetary Analysis](images/customer_value_analysis.png)
*Monetary Analysis Page*

![Conclusion and Recommendations](images/conclusion_and_recommendations.png)
*Conclusion and Recommendations*

![Next Steps](images/next_steps.png)
*Next Steps*

![Project Timeline and Roadmap](images/project_timeline_and_roadmap.png)
*Project Timeline and Roadmap*

## Insights and Analysis

### Recency Insights
- The majority of customers have made recent purchases, indicating high engagement.
- While only compromising a minority, the amount of customers with higher R scores suggests a need for retention strategies.
### Frequency Insights
- Most customers score highly in frequency.
- Targeting low-frequency customers with marketing campaigns can increase their engagement.
### Monetary Insights
- High-value segments like "Champions" and "Promising Customers" contribute significantly to revenue.
- Segments at risk of churning need targeted campaigns to maintain their engagement.


## Recommendations

### Immediate Actions: Quick Wins
**Engagement Programs:** 
- Initiate programs specifically aimed at recent customers to sustain their engagement.
- Consistently track the distribution of recent interactions to spot shifts in engagement patterns.

**Reactivation Campaigns:**
- Craft tailored campaigns to re-engage customers with high recency scores through personalized offers and incentives.
- Target low-frequency customers (F1, F2) with special promotions and reminders to encourage more frequent purchases.

**Focus on High-Value Customers:** 
- Launch loyalty initiatives for high-value segments such as Promising Customers, Cannot Lose Them, and Champions to sustain and enhance their value.

### Long-Term Strategies

**Customer Behavior Insights:** 
- Continuously study the behaviors and preferences of high-frequency customers to apply successful strategies to other segments.
- Develop and implement referral programs that leverage loyal customers to attract new ones.

**Segment-Specific Approaches:** 
- Re-engage segments like At Risk and About To Sleep with tailored incentives, discounts, and personalized communications.
- Encourage new customers to make repeat purchases, transitioning them into higher value segments.
- Devise strategies to win back Lost Customers and bolster retention efforts.

**Monitoring and Adaptation:**
- Regularly assess the contributions of different segments to the overall monetary value and adjust strategies based on evolving customer behaviors.
- Utilize insights from the waterfall chart to dynamically guide marketing and retention strategies.


## Future Work
- **Customer Lifetime Value (CLV) Evaluation:** Conduct an in-depth analysis of customer lifetime value to pinpoint long-term revenue potential and optimize the allocation of resources accordingly.
- **Churn Prediction:** Develop predictive models to identify customers at risk of churning and implement proactive retention strategies to mitigate this risk.
- **Enhanced Segmentation:** Investigate additional segmentation criteria beyond RFM, including customer demographics and behavior patterns, to refine and enhance marketing strategies.

## Contact
Thank you for your interest in this project! For more information, please contact:

**Name:** Matt Delaune

**Email:** matt.delaune@gmail.com
