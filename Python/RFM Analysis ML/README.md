# :tv:	RFM Analysis Machine Learning

Project can be viewed [here](https://github.com/forgek153/Projects/blob/main/Python/RFM%20Analysis%20ML/RFM%20Clustering%20ML.ipynb).

This project employs RFM (Recency, Frequency, Monetary Value) analysis to segment customers based on their purchasing behavior, using advanced data analytics techniques including K-Means clustering and silhouette scoring. The goal is to categorize customers into distinct groups - Occasional Engagers, Loyal High Spenders, and Consistent Customers - for targeted marketing strategies and improved customer engagement.


## Prerequisite

The data used for this project was queried from [UCI Online Dataset]( https://archive.ics.uci.edu/dataset/352/online+retail) through the following steps using Google's BigQuery.

```sql
WITH 
filtered_df AS (
    SELECT 
        *, 
        Quantity * UnitPrice AS TotalSum
    FROM 
        `trans-filament-408113.onlineretaildb.retail` 
    WHERE 
        CustomerID IS NOT NULL 
         AND ((Quantity > 0) AND (UnitPrice > 0))
)


    SELECT  
        DATE_DIFF(
            DATE_ADD((SELECT MAX(InvoiceDate) FROM filtered_df), INTERVAL 1 DAY),
            MAX(InvoiceDate),
            DAY
        ) AS Recency,  
        COUNT(InvoiceNo) AS Frequency,
        SUM(TotalSum) AS MonetaryValue
    FROM 
        filtered_df
    GROUP BY 
        CustomerID




```
The queried data can be downloaded [here](https://github.com/forgek153/Projects/files/13715506/rfm.csv).




