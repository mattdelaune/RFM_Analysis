# Power BI RFM Analysis Dashboard

## Overview

This repository contains a comprehensive Power BI project that demonstrates advanced data visualization and analytical insights using Recency, Frequency, and Monetary (RFM) analysis. The project aims to segment customers based on their purchasing behavior to drive strategic business decisions.

## Table of Contents

- [Project Description](#project-description)
- [Features](#features)
- [Data Source](#data-source)
- [Usage](#usage)
- [Screenshots](#screenshots)
- [Insights and Analysis](#insights-and-analysis)
- [Future Work](#future-work)
- [Contact](#contact)

## Project Description

The RFM Analysis Dashboard is designed to help businesses understand their customer base by analyzing their purchasing behavior. This project leverages Power BI's advanced data visualization capabilities to segment customers and provide actionable insights.

## Features

- **Interactive Visualizations**: Detailed and interactive charts and graphs to explore customer segments.
- **RFM Segmentation**: Breakdown of customers based on Recency, Frequency, and Monetary values.
- **Tooltips and Annotations**: Detailed explanations and insights available through tooltips on each page.
- **Dynamic Filters**: Slicers for filtering data based on R, F, and M scores.
- **Comprehensive Analysis**: Pages dedicated to each aspect of RFM, including detailed segment analysis.

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


## Usage

1. Open Power BI Desktop and load the .pbix file.
2. Explore the interactive dashboard to analyze customer segments.
3. Use the slicers to filter data based on different R, F, and M scores.
4. Hover over the tooltips to gain deeper insights into the data.


## Understanding RFM Analysis
### RFM Analysis Overview:
RFM (Recency, Frequency, Monetary) analysis is a quantitative technique utilized to segment customers based on their purchasing patterns. This method helps businesses pinpoint their most valuable customers and customize marketing strategies to maximize engagement and profitability.

### Purpose of RFM Analysis:
The primary aim of RFM analysis is to enhance customer retention, boost sales, and improve overall business performance. By gaining insights into customer behavior, businesses can make informed decisions to foster stronger customer relationships.

### Benefits of RFM Analysis:
- Identification of High-Value Customers: Recognizes the most profitable customers who contribute significantly to revenue.
- Optimization of Marketing Efforts: Helps in tailoring marketing campaigns to suit different customer segments, leading to more effective resource utilization.
- Improvement in Customer Retention and Loyalty: By understanding customer behavior, businesses can implement strategies to keep customers engaged and loyal.
- Revenue Increase Through Targeted Strategies: Employing focused marketing strategies on segmented customer groups can drive higher sales and revenue growth.

### RFM Analysis Process:
- **Recency:** Measures how recently a customer has made a purchase. Customers who have purchased recently are more likely to buy again compared to those who haven't made a purchase in a long time.
- **Frequency:** Tracks how often a customer makes a purchase. Frequent purchasers are typically more loyal and valuable.
- **Monetary:** Evaluates how much money a customer has spent. Higher spending customers are generally more valuable to the business.

### How to Generate RFM Scores
To generate RFM scores, follow these steps using four key parameters:
- Identify the Customer: Use a unique identifier such as a customer name or ID.
- Recency: Determine the number of days since the customer's last purchase.
- Frequency: Calculate the total number of transactions made by the customer.
- Monetary: Compute the total amount spent by the customer (also known as 'ticket size').
Each customer is then assigned an RFM score on a scale from 1 to 5 for each of these parameters, where 1 indicates the lowest value and 5 indicates the highest. This scoring helps in segmenting customers based on their purchasing behavior and value to the business.

![RFN Scores Description](images/rfm_explanation.png)
*Description of Segmentation*

![RFM Scores](images/rfm_scores.png)
*RFM Scores*

DAX (Data Analysis Expression) for RFM
- To calculate the ‘R value’, we need to know the latest date of the transaction. Therefore, we’ll create a new measure called ‘last transaction date’

```
-- To create a new measure called 'last transaction date'
last transaction date =
MAXX(FILTER('scanner_data','scanner_data'[Customer_ID]='scanner_data'[Customer_ID]),'scanner_data'[Date])
```
- Create a new measure for R-value and F-value (number of the ticket)

```
-- To create a new measure called 'R value'
R value = DATEDIFF('scanner_data'[last transaction date],TODAY(),DAY)
-- To create a new measure called 'F value'
F value = DISTINCTCOUNT('scanner_data'[Transaction_ID])
```

- Create a new measure for M-value:

```
-- To create a new measure called 'M value'
M value =
var TotalSales = SUM('scanner_data'[Sales_Amount])
var TotalQuantity = sum(scanner_data[Quantity])
Return 
DIVIDE (TotalSales,TotalQuantity,0)
```

- Next, Generate the new table called ‘RFM table’; To create the ‘RFM table’ use dax as below

```
-- To create a new table called 'RFM table'
RFM table = SUMMARIZE(
   'scanner_data','scanner_data'[Customer_ID],
   "R Value",[R Value],
   "F Value",[F Value],
   "M Value",[M Value])
```

- After we generate the ‘RFM table’, we get the table as shown below

![RFM Table Example](images/rfm_table_example.png)
*RFM Table Example*

- Next, we create a new column ‘R Score’, ‘F Score, and ‘M Score’. We separate group each data by percentile.

```
-- To create a new column called 'R Score'
R Score = 
SWITCH (
  TRUE (),
   [R value] <= PERCENTILE.INC ( 'RFM table'[R Value], 0.20 ), "5",    
   [R value] <= PERCENTILE.INC ( 'RFM table'[R Value], 0.40 ), "4", 
   [R Value] <= PERCENTILE.INC ( 'RFM table'[R Value], 0.60 ), "3", 
   [R value] <= PERCENTILE.INC ( 'RFM table'[R Value], 0.80 ), "2",
   "1"
       )
-- To create a new column called 'F Score'
F Score =
SWITCH (
  TRUE (),
   [F value] <= PERCENTILE.INC ( 'RFM table'[F Value], 0.20 ), "1",    
   [F value] <= PERCENTILE.INC ( 'RFM table'[F Value], 0.40 ), "2", 
   [F Value] <= PERCENTILE.INC ( 'RFM table'[F Value], 0.60 ), "3", 
   [F value] <= PERCENTILE.INC ( 'RFM table'[F Value], 0.80 ), "4",
   "5"
       )
-- To create a new column called 'M Score'
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

- Create the new column called ‘RFM’ by concatenating columns ‘R Score’, ‘F Score’, and ‘M Score’

```
-- To create a new column called 'RFM'
RFM = 'RFM table'[R Score]& 'RFM table'[F Score]&'RFM table'[M Score]
```

- To create a relationship between the ‘RFM table’ and the ‘Segmnt-Scores Table’ by using the columns ‘RFM’ in the RFM table and ‘Scores’ in the Segmnt-Scores table.

![RFM Relationship](images/rfm_relationship.png)
*Relationship between RFM table and Segment-Scores table*



## Screenshots

![Dashboard Overview](images/dashboard_overview.png)
*Dashboard Overview Page*

![RFM Dashboard](_.png)
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
The majority of customers have made recent purchases, indicating high engagement.
A significant drop-off in customer count as recency values increase suggests a need for retention strategies.
### Frequency Insights
High-frequency customers are loyal and frequently return to make purchases.
Targeting low-frequency customers with marketing campaigns can increase their engagement.
### Monetary Insights
High-value segments like "Champions" and "Promising Customers" contribute significantly to revenue.
Segments at risk of churning need targeted campaigns to maintain their engagement.
### Future Work
Data Refresh: Implementing automatic data refresh to keep the dashboard updated.
Advanced Analytics: Adding predictive analytics to forecast customer behavior.
Publication: Exploring ways to publish the dashboard for broader access.

## Contact
For more information, please contact:

**Name:** Matt Delaune

**Email:** matt.delaune@gmail.com
