# 💡 Cohort Table Analysis
Welcome, this page will be about using SQL to create a cohort table for further analysis! 

A cohort table, often used in cohort analysis, is a tool that groups customers (or users) into cohorts based on shared characteristics or experiences within a defined time-span. These characteristics are often their first purchase date or the date they first signed up for a service. The cohort table then tracks and compares the behavior and performance of these groups over time.

The dataset used can be found here: https://archive.ics.uci.edu/dataset/352/online+retail
DB Used: BigQuery 
***

## 📚 Table of Contents

  - [SQL Query](#sql-query)
  - [Python](#python)
    - [Quick Analysis](#quick-analysis)


***
## SQL Query

## Full Code

```sql
WITH 
filtered_df AS (
    SELECT 
        * 
    FROM 
        `trans-filament-408113.onlineretaildb.retail` 
    WHERE 
        CustomerID IS NOT NULL 
        AND ((Quantity > 0) AND (UnitPrice > 0))
),

cohortdate AS (
    SELECT 
        CustomerID, 
        InvoiceDate,
        DATE_TRUNC(InvoiceDate, MONTH) AS InvoiceMonth, 
        MIN(DATE_TRUNC(InvoiceDate, MONTH)) OVER (PARTITION BY CustomerID) AS CohortMonth
    FROM 
        filtered_df
),

cohortdatediff AS (
    SELECT 
        *,
        EXTRACT(YEAR FROM InvoiceMonth) - EXTRACT(YEAR FROM CohortMonth) AS year_diff,
        EXTRACT(MONTH FROM InvoiceMonth) - EXTRACT(MONTH FROM CohortMonth) AS month_diff
    FROM 
        cohortdate
),

cohorttable AS (
    SELECT 
        CustomerID, InvoiceDate, InvoiceMonth, CohortMonth, 
        year_diff * 12 + month_diff + 1 AS CohortIndex
    FROM 
        cohortdatediff
),

cohort_counts AS (
    SELECT
        CohortMonth,
        CohortIndex,
        COUNT(DISTINCT CustomerID) AS CustomerCount
    FROM
        cohorttable
    GROUP BY
        CohortMonth, CohortIndex
),

cohort_sizes AS (
    SELECT 
        CohortMonth,
        CustomerCount AS CohortSize
    FROM 
        cohort_counts
    WHERE 
        CohortIndex = 1
),

cohort_index AS (
    SELECT
        c.CohortMonth,
            SUM(CASE WHEN CohortIndex = 1 THEN CustomerCount ELSE 0 END) AS Index_1,
            SUM(CASE WHEN CohortIndex = 2 THEN CustomerCount ELSE 0 END) AS Index_2,
            SUM(CASE WHEN CohortIndex = 3 THEN CustomerCount ELSE 0 END) AS Index_3,
            SUM(CASE WHEN CohortIndex = 4 THEN CustomerCount ELSE 0 END) AS Index_4,
            SUM(CASE WHEN CohortIndex = 5 THEN CustomerCount ELSE 0 END) AS Index_5,
            SUM(CASE WHEN CohortIndex = 6 THEN CustomerCount ELSE 0 END) AS Index_6,
            SUM(CASE WHEN CohortIndex = 7 THEN CustomerCount ELSE 0 END) AS Index_7,
            SUM(CASE WHEN CohortIndex = 8 THEN CustomerCount ELSE 0 END) AS Index_8,
            SUM(CASE WHEN CohortIndex = 9 THEN CustomerCount ELSE 0 END) AS Index_9,
            SUM(CASE WHEN CohortIndex = 10 THEN CustomerCount ELSE 0 END) AS Index_10,
            SUM(CASE WHEN CohortIndex = 11 THEN CustomerCount ELSE 0 END) AS Index_11,
            SUM(CASE WHEN CohortIndex = 12 THEN CustomerCount ELSE 0 END) AS Index_12,
            SUM(CASE WHEN CohortIndex = 13 THEN CustomerCount ELSE 0 END) AS Index_13   
        FROM
        cohort_counts c
    GROUP BY
        c.CohortMonth
)


    SELECT
        ci.CohortMonth,
        ci.Index_1 / cs.CohortSize *100 AS Retention_Index_1,
        ci.Index_2 / cs.CohortSize *100 AS Retention_Index_2,
        ci.Index_3 / cs.CohortSize *100 AS Retention_Index_3,
        ci.Index_4 / cs.CohortSize *100 AS Retention_Index_4,
        ci.Index_5 / cs.CohortSize *100 AS Retention_Index_5,
        ci.Index_6 / cs.CohortSize *100 AS Retention_Index_6,
        ci.Index_7 / cs.CohortSize *100 AS Retention_Index_7,
        ci.Index_8 / cs.CohortSize *100 AS Retention_Index_8,
        ci.Index_9 / cs.CohortSize *100 AS Retention_Index_9,
        ci.Index_10 / cs.CohortSize *100 AS Retention_Index_10,
        ci.Index_11 / cs.CohortSize *100 AS Retention_Index_11,
        ci.Index_12 / cs.CohortSize *100 AS Retention_Index_12,
        ci.Index_13 / cs.CohortSize *100 AS Retention_Index_13
    FROM 
        cohort_index ci
    JOIN 
        cohort_sizes cs ON ci.CohortMonth = cs.CohortMonth
    ORDER BY CohortMonth
```
## Broken Down Code




Step 1: Filter Data where CustomerID is not empty and Quantity and UnitPrice is greater than 0
```sql
    SELECT 
        * 
    FROM 
        `trans-filament-408113.onlineretaildb.retail` 
    WHERE 
        CustomerID IS NOT NULL 
        AND ((Quantity > 0) AND (UnitPrice > 0))
```
| InvoiceNo | StockCode | Description                         | Quantity | InvoiceDate | UnitPrice | CustomerID | Country |
|-----------|-----------|-------------------------------------|----------|-------------|-----------|------------|---------|
| 572215    | 23293     | SET OF 12 FAIRY CAKE BAKING CASES   | 32       | 2011-10-21  | 0.83      | 12646      | USA     |
| 572215    | 23296     | SET OF 6 TEA TIME BAKING CASES      | 32       | 2011-10-21  | 1.25      | 12646      | USA     |
| 580553    | 23366     | SET 12 COLOURING PENCILS DOILY      | 72       | 2011-12-05  | 0.65      | 12646      | USA     |
| 580553    | 20975     | 12 PENCILS SMALL TUBE RED RETROSPOT | 72       | 2011-12-05  | 0.65      | 12646      | USA     |
| 536540    | 22355     | CHARLOTTE BAG SUKI DESIGN           | 50       | 2010-12-01  | 0.85      | 14911      | EIRE    |
| 536890    | 17084R    | ASSORTED INCENSE PACK               | 1440     | 2010-12-03  | 0.16      | 14156      | EIRE    |
| 536890    | 17091J    | VANILLA INCENSE IN TIN              | 72       | 2010-12-03  | 0.85      | 14156      | EIRE    |
| 536975    | 22049     | WRAP CHRISTMAS SCREEN PRINT         | 50       | 2010-12-03  | 0.42      | 14911      | EIRE    |
| 536975    | 22112     | CHOCOLATE HOT WATER BOTTLE          | 9        | 2010-12-03  | 4.95      | 14911      | EIRE    |
| 537378    | 84945     | MULTI COLOUR SILVER T-LIGHT HOLDER  | 72       | 2010-12-06  | 0.85      | 14911      | EIRE    |

Step 2: Create InvoiceMonth by getting the first date of the InvoiceDate and create CohortMonth by getting the earliest first day of the month that customer purchased an item.
```sql
    SELECT 
        CustomerID, 
        InvoiceDate,
        DATE_TRUNC(InvoiceDate, MONTH) AS InvoiceMonth, 
        MIN(DATE_TRUNC(InvoiceDate, MONTH)) OVER (PARTITION BY CustomerID) AS CohortMonth
    FROM 
        filtered_df
```
| CustomerID | InvoiceDate | InvoiceMonth | CohortMonth |
|------------|-------------|--------------|-------------|
| 12431      | 2010-12-17  | 2010-12-01   | 2010-12-01  |
| 12431      | 2011-02-27  | 2011-02-01   | 2010-12-01  |
| 12431      | 2011-05-23  | 2011-05-01   | 2010-12-01  |
| 12431      | 2011-07-26  | 2011-07-01   | 2010-12-01  |
| 12431      | 2011-08-12  | 2011-08-01   | 2010-12-01  |
| 12431      | 2011-11-04  | 2011-11-01   | 2010-12-01  |
| 12431      | 2011-02-17  | 2011-02-01   | 2010-12-01  |
| 12431      | 2011-02-17  | 2011-02-01   | 2010-12-01  |
| 12431      | 2011-02-17  | 2011-02-01   | 2010-12-01  |
| 12431      | 2011-02-17  | 2011-02-01   | 2010-12-01  |

Step 3: Find difference betwen years and month between InvoiceMonth and CohortMonth
```sql
     SELECT 
         *,
         EXTRACT(YEAR FROM InvoiceMonth) - EXTRACT(YEAR FROM CohortMonth) AS year_diff,
         EXTRACT(MONTH FROM InvoiceMonth) - EXTRACT(MONTH FROM CohortMonth) AS month_diff
     FROM 
         cohortdate
```
| CustomerID | InvoiceDate | InvoiceMonth | CohortMonth | year_diff | month_diff |
|------------|-------------|--------------|-------------|-----------|------------|
| 14911      | 2010-12-01  | 2010-12-01   | 2010-12-01  | 0         | 0          |
| 14911      | 2010-12-03  | 2010-12-01   | 2010-12-01  | 0         | 0          |
| 14911      | 2010-12-03  | 2010-12-01   | 2010-12-01  | 0         | 0          |
| 14911      | 2010-12-06  | 2010-12-01   | 2010-12-01  | 0         | 0          |
| 12646      | 2011-10-21  | 2011-10-01   | 2011-10-01  | 0         | 0          |
| 12646      | 2011-10-21  | 2011-10-01   | 2011-10-01  | 0         | 0          |
| 12646      | 2011-12-05  | 2011-12-01   | 2011-10-01  | 0         | 2          |
| 12646      | 2011-12-05  | 2011-12-01   | 2011-10-01  | 0         | 2          |
| 14156      | 2010-12-03  | 2010-12-01   | 2010-12-01  | 0         | 0          |
| 14156      | 2010-12-03  | 2010-12-01   | 2010-12-01  | 0         | 0          |

Step 4: Create CohortIndex by  converting total difference to months. month_diff+1 so that first month is marked as 1 instead of 0 for easier interpretation.
```sql
    SELECT 
        CustomerID, InvoiceDate, InvoiceMonth, CohortMonth, 
        year_diff * 12 + month_diff + 1 AS CohortIndex
    FROM 
        cohortdatediff
```
| CustomerID | InvoiceDate | InvoiceMonth | CohortMonth | CohortIndex |
|------------|-------------|--------------|-------------|-------------|
| 12646      | 2011-10-21  | 2011-10-01   | 2011-10-01  | 1           |
| 12646      | 2011-10-21  | 2011-10-01   | 2011-10-01  | 1           |
| 12646      | 2011-12-05  | 2011-12-01   | 2011-10-01  | 3           |
| 12646      | 2011-12-05  | 2011-12-01   | 2011-10-01  | 3           |
| 14911      | 2010-12-01  | 2010-12-01   | 2010-12-01  | 1           |
| 14911      | 2010-12-03  | 2010-12-01   | 2010-12-01  | 1           |
| 14911      | 2010-12-03  | 2010-12-01   | 2010-12-01  | 1           |
| 14911      | 2010-12-06  | 2010-12-01   | 2010-12-01  | 1           |
| 14156      | 2010-12-03  | 2010-12-01   | 2010-12-01  | 1           |
| 14156      | 2010-12-03  | 2010-12-01   | 2010-12-01  | 1           |

Step 5: Count CustomerID per CohortMonth and per CohortIndex

```sql
    SELECT
        CohortMonth,
        CohortIndex,
        COUNT(DISTINCT CustomerID) AS CustomerCount
    FROM
        cohorttable
    GROUP BY
        CohortMonth, CohortIndex
```
| CohortMonth | CohortIndex | CustomerCount |
|-------------|-------------|---------------|
| 2011-03-01  | 10          | 39            |
| 2011-03-01  | 1           | 452           |
| 2011-03-01  | 8           | 104           |
| 2011-09-01  | 4           | 34            |
| 2011-08-01  | 1           | 169           |
| 2011-01-01  | 3           | 111           |
| 2010-12-01  | 7           | 321           |
| 2011-11-01  | 1           | 323           |
| 2011-05-01  | 4           | 49            |
| 2011-02-01  | 3           | 71            |

Step 6: Grab data for user's first month for further analysis down the pipeline.
``sql
    SELECT 
        CohortMonth,
        CustomerCount AS CohortSize
    FROM 
        cohort_counts
    WHERE 
        CohortIndex = 1
```
| CohortMonth   | CohortSize |
|---------------|------------|
| 2011-06-01    | 242        |
| 2011-11-01    | 323        |
| 2011-09-01    | 299        |
| 2011-03-01    | 452        |
| 2011-07-01    | 188        |
| 2011-08-01    | 169        |
| 2011-05-01    | 284        |
| 2011-04-01    | 300        |
| 2011-10-01    | 358        |
| 2011-01-01    | 417        |
| 2010-12-01    | 885        |
| 2011-02-01    | 380        |
| 2011-12-01    | 41         |

Step 7: Create a pivottable where we count customers based on their CohortIndex
```sql
    SELECT
        c.CohortMonth,
            SUM(CASE WHEN CohortIndex = 1 THEN CustomerCount ELSE 0 END) AS Index_1,
            SUM(CASE WHEN CohortIndex = 2 THEN CustomerCount ELSE 0 END) AS Index_2,
            SUM(CASE WHEN CohortIndex = 3 THEN CustomerCount ELSE 0 END) AS Index_3,
            SUM(CASE WHEN CohortIndex = 4 THEN CustomerCount ELSE 0 END) AS Index_4,
            SUM(CASE WHEN CohortIndex = 5 THEN CustomerCount ELSE 0 END) AS Index_5,
            SUM(CASE WHEN CohortIndex = 6 THEN CustomerCount ELSE 0 END) AS Index_6,
            SUM(CASE WHEN CohortIndex = 7 THEN CustomerCount ELSE 0 END) AS Index_7,
            SUM(CASE WHEN CohortIndex = 8 THEN CustomerCount ELSE 0 END) AS Index_8,
            SUM(CASE WHEN CohortIndex = 9 THEN CustomerCount ELSE 0 END) AS Index_9,
            SUM(CASE WHEN CohortIndex = 10 THEN CustomerCount ELSE 0 END) AS Index_10,
            SUM(CASE WHEN CohortIndex = 11 THEN CustomerCount ELSE 0 END) AS Index_11,
            SUM(CASE WHEN CohortIndex = 12 THEN CustomerCount ELSE 0 END) AS Index_12,
            SUM(CASE WHEN CohortIndex = 13 THEN CustomerCount ELSE 0 END) AS Index_13   
        FROM
        cohort_counts c
    GROUP BY
        c.CohortMonth
```
| CohortMonth   | Index_1 | Index_2 | Index_3 | Index_4 | Index_5 | Index_6 | Index_7 | Index_8 | Index_9 | Index_10 | Index_11 | Index_12 | Index_13 |
|---------------|---------|---------|---------|---------|---------|---------|---------|---------|---------|----------|----------|----------|----------|
| 2010-12-01    | 885     | 324     | 286     | 340     | 321     | 352     | 321     | 309     | 313     | 350      | 331      | 445      | 235      |
| 2011-10-01    | 358     | 86      | 41      | 0       | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        | 0        |
| 2011-08-01    | 169     | 35      | 42      | 41      | 21      | 0       | 0       | 0       | 0       | 0        | 0        | 0        | 0        |
| 2011-03-01    | 452     | 68      | 114     | 90      | 101     | 76      | 121     | 104     | 126     | 39       | 0        | 0        | 0        |
| 2011-01-01    | 417     | 92      | 111     | 96      | 134     | 120     | 103     | 101     | 125     | 136      | 152      | 49       | 0        |
| 2011-07-01    | 188     | 34      | 39      | 42      | 51      | 21      | 0       | 0       | 0       | 0        | 0        | 0        | 0        |
| 2011-06-01    | 242     | 42      | 38      | 64      | 56      | 81      | 23      | 0       | 0       | 0        | 0        | 0        | 0        |
| 2011-11-01    | 323     | 36      | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        | 0        |
| 2011-09-01    | 299     | 70      | 90      | 34      | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        | 0        |
| 2011-05-01    | 284     | 54      | 49      | 49      | 59      | 66      | 75      | 27      | 0       | 0        | 0        | 0        | 0        |
| 2011-02-01    | 380     | 71      | 71      | 108     | 103     | 94      | 96      | 106     | 94      | 116      | 26       | 0        | 0        |
| 2011-04-01    | 300     | 64      | 61      | 63      | 59      | 68      | 65      | 78      | 22      | 0        | 0        | 0        | 0        |
| 2011-12-01    | 41      | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        | 0        |

Step 8: Remember step 6? This is where we use the result to divide the size of the indexes by the cohortindex1 to obtain the ratio.
```sql
    SELECT
        ci.CohortMonth,
        ci.Index_1 / cs.CohortSize *100 AS Retention_Index_1,
        ci.Index_2 / cs.CohortSize *100 AS Retention_Index_2,
        ci.Index_3 / cs.CohortSize *100 AS Retention_Index_3,
        ci.Index_4 / cs.CohortSize *100 AS Retention_Index_4,
        ci.Index_5 / cs.CohortSize *100 AS Retention_Index_5,
        ci.Index_6 / cs.CohortSize *100 AS Retention_Index_6,
        ci.Index_7 / cs.CohortSize *100 AS Retention_Index_7,
        ci.Index_8 / cs.CohortSize *100 AS Retention_Index_8,
        ci.Index_9 / cs.CohortSize *100 AS Retention_Index_9,
        ci.Index_10 / cs.CohortSize *100 AS Retention_Index_10,
        ci.Index_11 / cs.CohortSize *100 AS Retention_Index_11,
        ci.Index_12 / cs.CohortSize *100 AS Retention_Index_12,
        ci.Index_13 / cs.CohortSize *100 AS Retention_Index_13
    FROM 
        cohort_index ci
    JOIN 
        cohort_sizes cs ON ci.CohortMonth = cs.CohortMonth
    ORDER BY CohortMonth
```
| CohortMonth   | Retention_Index_1 | Retention_Index_2 | Retention_Index_3 | Retention_Index_4 | Retention_Index_5 | Retention_Index_6 | Retention_Index_7 | Retention_Index_8 | Retention_Index_9 | Retention_Index_10 | Retention_Index_11 | Retention_Index_12 | Retention_Index_13 |
|---------------|--------------------|--------------------|--------------------|--------------------|--------------------|--------------------|--------------------|--------------------|--------------------|---------------------|---------------------|---------------------|---------------------|
| 2010-12-01    | 100.0              | 36.610             | 32.316             | 38.418             | 36.271             | 39.774             | 36.271             | 34.915             | 35.367             | 39.548              | 37.401              | 50.282              | 26.553              |
| 2011-01-01    | 100.0              | 22.062             | 26.618             | 23.021             | 32.134             | 28.776             | 24.700             | 24.220             | 29.976             | 32.613              | 36.450              | 11.750              | 0.0                |
| 2011-02-01    | 100.0              | 18.684             | 18.684             | 28.421             | 27.105             | 24.736             | 25.263             | 27.894             | 24.736             | 30.526              | 6.842               | 0.0                | 0.0                |
| 2011-03-01    | 100.0              | 15.044             | 25.221             | 19.911             | 22.345             | 16.814             | 26.769             | 23.008             | 27.876             | 8.628               | 0.0                 | 0.0                 | 0.0                 |
| 2011-04-01    | 100.0              | 21.333             | 20.333             | 21.0               | 19.666             | 22.666             | 21.666             | 26.0               | 7.333              | 0.0                 | 0.0                 | 0.0                 | 0.0                 |
| 2011-05-01    | 100.0              | 19.014             | 17.253             | 17.253             | 20.774             | 23.239             | 26.408             | 9.507              | 0.0                | 0.0                 | 0.0                 | 0.0                 | 0.0                 |
| 2011-06-01    | 100.0              | 17.355             | 15.702             | 26.446             | 23.140             | 33.471             | 9.504              | 0.0                | 0.0                | 0.0                 | 0.0                 | 0.0                 | 0.0                 |
| 2011-07-01    | 100.0              | 18.085             | 20.744             | 22.340             | 27.127             | 11.170             | 0.0                | 0.0                | 0.0                | 0.0                 | 0.0                 | 0.0                 | 0.0                 |
| 2011-08-01    | 100.0              | 20.710             | 24.852             | 24.260             | 12.426             | 0.0                | 0.0                | 0.0                | 0.0                | 0.0                 | 0.0                 | 0.0                 | 0.0                 |
| 2011-09-01    | 100.0              | 23.411             | 30.100             | 11.371             | 0.0                | 0.0                | 0.0                | 0.0                | 0.0                | 0.0                 | 0.0                 | 0.0                 | 0.0                 |
| 2011-10-01    | 100.0              | 24.022             | 11.452             | 0.0                | 0.0                | 0.0                | 0.0                | 0.0                | 0.0                | 0.0                 | 0.0                 | 0.0                 | 0.0                 |
| 2011-11-01    | 100.0              | 11.145             | 0.0                | 0.0                | 0.0                | 0.0                | 0.0                | 0.0                | 0.0                | 0.0                 | 0.0                 | 0.0                 | 0.0                 |
| 2011-12-01    | 100.0              | 0.0                | 0.0                | 0.0                | 0.0                | 0.0                | 0.0                | 0.0                | 0.0                | 0.0                 | 0.0                 | 0.0                 | 0.0                 |


***
## Python: 
Let's make this table more visually appealing and easier to understand.
Step 1: import pandas as numpy and create pandas dataframe

```python
import pandas as pd
import numpy as np
df = pd.read_csv('retention.csv',index_col=0)
```
| CohortMonth   | Retention_Index_1 | Retention_Index_2 | Retention_Index_3 | Retention_Index_4 | Retention_Index_5 | Retention_Index_6 | Retention_Index_7 | Retention_Index_8 | Retention_Index_9 | Retention_Index_10 | Retention_Index_11 | Retention_Index_12 | Retention_Index_13 |
|---------------|--------------------|--------------------|--------------------|--------------------|--------------------|--------------------|--------------------|--------------------|--------------------|---------------------|---------------------|---------------------|---------------------|
| 2010-12-01    | 100.0              | 36.610169          | 32.316384          | 38.418079          | 36.271186          | 39.774011          | 36.271186          | 34.915254          | 35.367232          | 39.548023           | 37.401130           | 50.282486           | 26.553672           |
| 2011-01-01    | 100.0              | 22.062350          | 26.618705          | 23.021583          | 32.134293          | 28.776978          | 24.700240          | 24.220624          | 29.976019          | 32.613909           | 36.450839           | 11.750600           | 0.000000            |
| 2011-02-01    | 100.0              | 18.684211          | 18.684211          | 28.421053          | 27.105263          | 24.736842          | 25.263158          | 27.894737          | 24.736842          | 30.526316           | 6.842105            | 0.000000            | 0.000000            |
| 2011-03-01    | 100.0              | 15.044248          | 25.221239          | 19.911504          | 22.345133          | 16.814159          | 26.769912          | 23.008850          | 27.876106          | 8.628319            | 0.000000            | 0.000000            | 0.000000            |
| 2011-04-01    | 100.0              | 21.333333          | 20.333333          | 21.000000          | 19.666667          | 22.666667          | 21.666667          | 26.000000          | 7.333333           | 0.000000            | 0.000000            | 0.000000            | 0.000000            |
| 2011-05-01    | 100.0              | 19.014085          | 17.253521          | 17.253521          | 20.774648          | 23.239437          | 26.408451          | 9.507042           | 0.000000           | 0.000000            | 0.000000            | 0.000000            | 0.000000            |
| 2011-06-01    | 100.0              | 17.355372          | 15.702479          | 26.446281          | 23.140496          | 33.471074          | 9.504132           | 0.000000           | 0.000000           | 0.000000            | 0.000000            | 0.000000            | 0.000000            |
| 2011-07-01    | 100.0              | 18.085106          | 20.744681          | 22.340426          | 27.127660          | 11.170213          | 0.000000           | 0.000000           | 0.000000           | 0.000000            | 0.000000            | 0.000000            | 0.000000            |
| 2011-08-01    | 100.0              | 20.710059          | 24.852071          | 24.260355          | 12.426036          | 0.000000           | 0.000000           | 0.000000           | 0.000000           | 0.000000            | 0.000000            | 0.000000            | 0.000000            |
| 2011-09-01    | 100.0              | 23.411371          | 30.100334          | 11.371237          | 0.000000           | 0.000000           | 0.000000           | 0.000000           | 0.000000           | 0.000000            | 0.000000            | 0.000000            | 0.000000            |
| 2011-10-01    | 100.0              | 24.022346          | 11.452514          | 0.000000           | 0.000000           | 0.000000           | 0.000000           | 0.000000           | 0.000000           | 0.000000            | 0.000000            | 0.000000            | 0.000000            |
| 2011-11-01    | 100.0              | 11.145511          | 0.000000           | 0.000000           | 0.000000           | 0.000000           | 0.000000           | 0.000000           | 0.000000           | 0.000000            | 0.000000            | 0.000000            | 0.000000            |
| 2011-12-01    | 100.0              | 0.000000           | 0.000000           | 0.000000           | 0.000000           | 0.000000           | 0.000000           | 0.000000           | 0.000000           | 0.000000            | 0.000000            | 0.000000            | 0.000000            |

Step 2: rename columns and replace 0 with nan to remove the '0' from the table
```python
df.columns = [1, 2, 3, 4, 5,6,7,8,9,10,11,12,13]
df = df.round(1)
df = df.replace(0,np.nan)
```

Step 3: Create the heatmap table using Seaborn
```python
import seaborn as sns
import matplotlib.pyplot as plt
plt.figure(figsize =(10,8))
plt.title('Retention Rates')
sns.heatmap(data = df, annot = True,fmt = '.1f',vmin = 0.0,vmax = 50,cmap='BuGn')
```

![image](https://github.com/forgek153/Projects/assets/132448826/0ec00f1b-4d0b-469e-9b4b-b8434b3235c8)
***
## Quick Analysis

Analyzing the table:

The cohort starting on 2010-12-01 begins with 100% retention, and shows a decline over time. This is normal as attrition occurs.

Looking at the color distribution, the retention rates seem to drop most significantly after the first month.

There seems to be a pattern where certain months have higher retention rates later in the lifecycle. For instance, the 10th month for several cohorts shows a relatively higher retention rate (darker color) compared to neighboring months.































