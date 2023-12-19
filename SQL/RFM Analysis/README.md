# :mag_right: RFM Analysis

This project demonstrates the application of RFM analysis using SQL on a retail dataset in BigQuery. It focuses on segmenting customers into Bronze, Silver, and Gold categories based on their purchasing patterns (Recency, Frequency, Monetary Value), enabling targeted marketing strategies and enhanced customer engagement and retention. The process involves data cleaning, calculation of RFM metrics, and categorization of customers into segments to inform actionable marketing insights.




The dataset used can be found [here](https://archive.ics.uci.edu/dataset/352/online+retail)

DB Used: BigQuery 

RFM analysis is a marketing technique used to quantitatively rank and group customers based on their purchasing patterns. The acronym RFM stands for Recency, Frequency, and Monetary value, each of which is a key customer behavior metric. By segmenting customers in this manner, businesses can create more personalized, effective marketing strategies and improve customer engagement and retention.


***

## 📚 Table of Contents

  - [SQL Query](#sql-query)
  - [Result](#result)
  - [Analysis](#analysis)
  
***
## SQL Query

## Full Code
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
),

pre_RFM AS (
    SELECT 
        CustomerID, 
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
),

rfm2 AS (
    SELECT
        CustomerID,
        Recency,
        Frequency,
        MonetaryValue,
        PERCENTILE_DISC(Recency, 0.25) OVER () AS Recency_1_Quin,
        PERCENTILE_DISC(Recency, 0.5) OVER () AS Recency_2_Quin,
        PERCENTILE_DISC(Recency, 0.75) OVER () AS Recency_3_Quin,
        PERCENTILE_DISC(Recency, 1) OVER () AS Recency_4_Quin,
        PERCENTILE_DISC(MonetaryValue, 0.25) OVER () AS MonetaryValue_1_Quin,
        PERCENTILE_DISC(MonetaryValue, 0.5) OVER () AS MonetaryValue_2_Quin,
        PERCENTILE_DISC(MonetaryValue, 0.75) OVER () AS MonetaryValue_3_Quin,
        PERCENTILE_DISC(MonetaryValue, 1) OVER () AS MonetaryValue_4_Quin,
        PERCENTILE_DISC(Frequency, 0.25) OVER () AS Frequency_1_Quin,
        PERCENTILE_DISC(Frequency, 0.5) OVER () AS Frequency_2_Quin,
        PERCENTILE_DISC(Frequency, 0.75) OVER () AS Frequency_3_Quin,
        PERCENTILE_DISC(Frequency, 1) OVER () AS Frequency_4_Quin
    FROM 
        pre_RFM
),

rfm_quantile AS (
    SELECT
        CustomerID,
        Recency,
        Frequency,
        MonetaryValue,
        (CASE 
            WHEN Recency <= Recency_1_Quin THEN 4
            WHEN Recency > Recency_1_Quin AND Recency <= Recency_2_Quin THEN 3
            WHEN Recency > Recency_2_Quin AND Recency <= Recency_3_Quin THEN 2
            WHEN Recency > Recency_3_Quin AND Recency <= Recency_4_Quin THEN 1
            ELSE 5 
        END) AS Recency_Quantile,
        (CASE 
            WHEN Frequency <= Frequency_1_Quin THEN 1
            WHEN Frequency > Frequency_1_Quin AND Frequency <= Frequency_2_Quin THEN 2
            WHEN Frequency > Frequency_2_Quin AND Frequency <= Frequency_3_Quin THEN 3
            WHEN Frequency > Frequency_3_Quin AND Frequency <= Frequency_4_Quin THEN 4
            ELSE 5 
        END) AS Frequency_Quantile,
        (CASE 
            WHEN MonetaryValue <= MonetaryValue_1_Quin THEN 1
            WHEN MonetaryValue > MonetaryValue_1_Quin AND MonetaryValue <= MonetaryValue_2_Quin THEN 2
            WHEN MonetaryValue > MonetaryValue_2_Quin AND MonetaryValue <= MonetaryValue_3_Quin THEN 3
            WHEN MonetaryValue > MonetaryValue_3_Quin AND MonetaryValue <= MonetaryValue_4_Quin THEN 4
            ELSE 5 
        END) AS Monetary_Value_Quantile
    FROM 
        rfm2
),

rfm AS (
    SELECT 
        *,
        CONCAT(CAST(Recency_Quantile AS STRING), CAST(Frequency_Quantile AS STRING), CAST(Monetary_Value_Quantile AS STRING)) AS RFM_Segment, 
        (Recency_Quantile + Frequency_Quantile + Monetary_Value_Quantile) AS RFM_Score
    FROM 
        rfm_quantile
),

rfm_segments AS (
    SELECT
        *,
        CASE 
            WHEN RFM_Score >= 9 THEN 'Gold'
            WHEN RFM_Score BETWEEN 5 AND 9 THEN 'Silver'
            ELSE 'Bronze' 
        END AS Segments
    FROM 
        rfm
)

SELECT 
    Segments,
    AVG(Recency) AS AVG_Recency,
    AVG(Frequency) AS AVG_Frequency,
    AVG(MonetaryValue) AS AVG_MonetaryValue,
    COUNT(*) AS Count
FROM 
    rfm_segments
GROUP BY 
    Segments
ORDER BY 
    Segments
```
***

## Broken Down Code
Step 1: Clean the data where CustomerID is not empty and Quantity and Price is greater than 0. And create TotalSum column by multiplying  Quantity and Unitprice.
```sql
    SELECT 
        *, 
        Quantity * UnitPrice AS TotalSum
    FROM 
        `trans-filament-408113.onlineretaildb.retail` 
    WHERE 
        CustomerID IS NOT NULL 
         AND ((Quantity > 0) AND (UnitPrice > 0))
```

Step 2: Create Recency Column by finding the difference in date between maximum date of the dataset + 1 (since this is a historic dataset) subtracting the latest invoicedate from each customer.Create Frequency Column by couting the amount of times customer has purchased from store. Create MonetaryValue buy summing up the amount of money customer has spent.
```sql
    SELECT 
        CustomerID, 
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
Step3: Create 4 bins for Recency,Frequency,and MonetaryValue. 
```sql
    SELECT
        CustomerID,
        Recency,
        Frequency,
        MonetaryValue,
        PERCENTILE_DISC(Recency, 0.25) OVER () AS Recency_1_Quin,
        PERCENTILE_DISC(Recency, 0.5) OVER () AS Recency_2_Quin,
        PERCENTILE_DISC(Recency, 0.75) OVER () AS Recency_3_Quin,
        PERCENTILE_DISC(Recency, 1) OVER () AS Recency_4_Quin,
        PERCENTILE_DISC(MonetaryValue, 0.25) OVER () AS MonetaryValue_1_Quin,
        PERCENTILE_DISC(MonetaryValue, 0.5) OVER () AS MonetaryValue_2_Quin,
        PERCENTILE_DISC(MonetaryValue, 0.75) OVER () AS MonetaryValue_3_Quin,
        PERCENTILE_DISC(MonetaryValue, 1) OVER () AS MonetaryValue_4_Quin,
        PERCENTILE_DISC(Frequency, 0.25) OVER () AS Frequency_1_Quin,
        PERCENTILE_DISC(Frequency, 0.5) OVER () AS Frequency_2_Quin,
        PERCENTILE_DISC(Frequency, 0.75) OVER () AS Frequency_3_Quin,
        PERCENTILE_DISC(Frequency, 1) OVER () AS Frequency_4_Quin
    FROM 
        pre_RFM
```
Step 4: Place each customer in each bin by using CASE

```sql
    SELECT
        CustomerID,
        Recency,
        Frequency,
        MonetaryValue,
        (CASE 
            WHEN Recency <= Recency_1_Quin THEN 4
            WHEN Recency > Recency_1_Quin AND Recency <= Recency_2_Quin THEN 3
            WHEN Recency > Recency_2_Quin AND Recency <= Recency_3_Quin THEN 2
            WHEN Recency > Recency_3_Quin AND Recency <= Recency_4_Quin THEN 1
            ELSE 5 
        END) AS Recency_Quantile,
        (CASE 
            WHEN Frequency <= Frequency_1_Quin THEN 1
            WHEN Frequency > Frequency_1_Quin AND Frequency <= Frequency_2_Quin THEN 2
            WHEN Frequency > Frequency_2_Quin AND Frequency <= Frequency_3_Quin THEN 3
            WHEN Frequency > Frequency_3_Quin AND Frequency <= Frequency_4_Quin THEN 4
            ELSE 5 
        END) AS Frequency_Quantile,
        (CASE 
            WHEN MonetaryValue <= MonetaryValue_1_Quin THEN 1
            WHEN MonetaryValue > MonetaryValue_1_Quin AND MonetaryValue <= MonetaryValue_2_Quin THEN 2
            WHEN MonetaryValue > MonetaryValue_2_Quin AND MonetaryValue <= MonetaryValue_3_Quin THEN 3
            WHEN MonetaryValue > MonetaryValue_3_Quin AND MonetaryValue <= MonetaryValue_4_Quin THEN 4
            ELSE 5 
        END) AS Monetary_Value_Quantile
    FROM 
        rfm2
```

Step 5: Create RFM Score by concatenating R,F,M values together and summing the scores together.
```sql
    SELECT 
        *,
        CONCAT(CAST(Recency_Quantile AS STRING), CAST(Frequency_Quantile AS STRING), CAST(Monetary_Value_Quantile AS STRING)) AS RFM_Segment, 
        (Recency_Quantile + Frequency_Quantile + Monetary_Value_Quantile) AS RFM_Score
    FROM 
        rfm_quantile
),
```

Step 6: Distinguishing each customer into different teirs by RFM_Score
```sql
    SELECT
        *,
        CASE 
            WHEN RFM_Score >= 9 THEN 'Gold'
            WHEN RFM_Score BETWEEN 5 AND 9 THEN 'Silver'
            ELSE 'Bronze' 
        END AS Segments
    FROM 
        rfm
```
Step 7: Calculate average values for each tier so we can distinguish the characteristics of each tier.
```sql
SELECT 
    Segments,
    AVG(Recency) AS AVG_Recency,
    AVG(Frequency) AS AVG_Frequency,
    AVG(MonetaryValue) AS AVG_MonetaryValue,
    COUNT(*) AS Count
FROM 
    rfm_segments
GROUP BY 
    Segments
ORDER BY 
    Segments
```
***
## Result:
| Segments | AVG_Recency         | AVG_Frequency      | AVG_MonetaryValue | Count |
|----------|---------------------|--------------------|-------------------|-------|
| Bronze   | 218.55541069100397  | 10.940026075619299 | 198.78607561929593| 767   |
| Gold     | 26.86575178997613   | 191.9797136038186  | 4406.4589510739852| 1676  |
| Silver   | 100.80897097625329  | 35.74406332453826  | 724.91492453825879| 1895  |
***
## Analysis:
|Segment|Analysis|Action|
|------|----------|----|
|Bronze|Customers in the Bronze segment have the longest average recency, indicating they haven't made a purchase recently compared to other segments. Their frequency and monetary value are the lowest among the three groups. This suggests they are either occasional shoppers or new customers who haven't yet established a pattern of regular purchasing.|Aim to increase engagement through welcome offers, re-engagement campaigns, or introducing loyalty programs.|
|Gold|Gold segment customers are the most valuable to the business. They have purchased recently, do so frequently, and spend significantly more than other segments. This group likely includes loyal customers or those with a higher propensity to purchase. Focusing marketing efforts and customer retention strategies on this segment could yield high returns.|Focus on retention strategies, personalized offers, and exclusive services to maintain their loyalty and high spending.|
|Silver|Customers in the Silver segment are in the middle in terms of recent activity, frequency of purchases, and monetary value. They are less active and spend less than Gold customers but are more engaged than Bronze customers. This segment might represent occasional or seasonal shoppers, and there might be potential to move some of these customers into the Gold segment with targeted engagement strategies.|Identify potential customers who could be moved to the Gold segment with the right incentives, such as targeted marketing campaigns or personalized recommendations.|
