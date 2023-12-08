# :soccer: UA Combine

This project involves web scraping athlete performance data from the Competition Corner website using Selenium, a web automation tool. The primary goal is to extract detailed information on individual athletes participating in a fitness competition. The script navigates through pages, clicks on consent pop-ups, retrieves table data, and consolidates it into a structured DataFrame using the Pandas library.

## 📚TABLE OF CONTENTS

1. [Introduction](#1.-introduction)
   
   - [Website Layout & Strategy](#website-layout-&-strategy)

3. [Scraping Combine Fact Table](#scraping-combine-fact-table)
   
   - [Scraping Data](#1.1-scraping-combine-fact-table)
   
   - [Creating Dataframes](#1.2-creating-dataframes)
   
   - [Scraping Data](#1.3-scraping-combine-fact-table)

4. [Scraping Contestant's Attributes](#scraping-contestant's-attributes)
   
   - [Read Data from Excel, Format Names](#2.1-read-data-from-excel,-format-names)
     
   - [Loop Through Formatted Names, Visit Athlete Pages, and Extract Information](#2.2-loop-through-formatted-names,-visit-athlete-pages,-and-extract-information)
  
   - [Create a DataFrame from Extracted Information and Save to Excel](#2.3-create-a-dataFrame-from-extracted-information-and-save-to-excel)

## 1. Introduction
The UA Combine is a fitness testing competition organized by the sports equipment company Under Armour. Competitions are being held in September and October 2022, in Singapore, Sydney, Kuala Lumpur, and Bangkok.

Each competition involves athletes performing eight fitness tests in a single day. Points will be earned for each completed test and athletes will be ranked according to their results.
The winners get cash prizes. For example, the Australian competition had prizes of $5000, $2500 and $1000 for individuals in the first three places. There is also a team competition.

This project is focused on scraping data only from Male Singapore UA Combine.

### Website Layout & Strategy
![image](https://github.com/forgek153/Projects/assets/132448826/3be2dd2e-24ef-4550-bd1d-877446986318)

One page contains participants and their performance. We have to "click" next workout to get more information about their other performances.
We will use Selenium to extract data and click through "Next Workout" to extract all of their performance.



## 2. Scraping Combine Fact Table

### 2.1 Scraping Data
This step involves automating the extraction of data from multiple pages of a competition results table on the Competition Corner website. The script uses Selenium to handle cookies agreement and navigate through pages.



```python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import pandas as pd

# Initialize Chrome WebDriver
#set_window_size to maximum as the table column scales with screen width 
driver = webdriver.Chrome()
driver.set_window_size(9999, 9999)

# Navigate to the competition results page
web = driver.get("https://competitioncorner.net/ff/9749/results#male_60258")

try:
   
    while True:
        # Click agree to cookies
        ClickAgree = driver.find_element(By.XPATH, "//*[@id='agreeToCookie']/i")
        ClickAgree.click()

        # Extract content from the first table
        Content = WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.CLASS_NAME, "ReactVirtualized__Grid__innerScrollContainer")))
        Contenttext = Content.text

        # Click to the next page
        Clickable = WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.XPATH, "//div[@class='stepper__next']")))
        Clickable.click()

        # Extract content from the second table
        Content2 = WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.CLASS_NAME, "ReactVirtualized__Grid__innerScrollContainer")))
        Contenttext2 = Content2.text

except:
    # Close the WebDriver in case of an exception or completion
    driver.quit()
```
### 2.2 Creating Dataframes

In this step, the extracted text is split into rows, and two separate DataFrames are created to organize the data.


```python
# 1st Content
# Split the text into rows
rows = Contenttext.split("\n")
```

The result:

![image](https://github.com/forgek153/Projects/assets/132448826/c105485c-22e1-4ef7-ba5f-84ea0bd4d30d)

The result that we've gotten isnt the tabular format that we want. We're going to have to reformat the data by using pandas dataframe.

```python
# Define column names for the first DataFrame
columns = ['Rank', 'Name', 'Total Points', 'Speed/Points', 'Speed/Rank', 'Speed/Time',
           'Agility/Points', 'Agility/Rank', 'Agility/Time', 'Power/Points', 'Power/Rank', 'Power/Time',
           'Cognition/Points', 'Cognition/Rank', 'Cognition/Reps', 'Stamina/Points', 'Stamina/Rank', 'Stamina/Reps',
           'Strength/Points', 'Strength/Rank', 'Strength/KG']

# Create the first DataFrame
# Loops through the content and inserts it under the corresponding column. For example, 1st and 22nd values are Rank, 2nd and 23rd values are Name.
df = pd.DataFrame([rows[i:i+21] for i in range(0, len(rows), 21)], columns=columns)
```

![image](https://github.com/forgek153/Projects/assets/132448826/c4f20ef3-ad36-472e-8aa9-83a755682fea)


```sql
# 2nd Content
# Split the text into rows
rows = Contenttext2.split("\n")

# Define column names for the second DataFrame
columns2 = ['Rank', 'Name', 'Total Points', 'Power/Points', 'Power/Rank', 'Power/Time',
            'Cognition/Points', 'Cognition/Rank', 'Cognition/Reps', 'Stamina/Points', 'Stamina/Rank', 'Stamina/Reps',
            'Strength/Points', 'Strength/Rank', 'Strength/KG', 'Vertical/Point', 'Vertical/Rank',
            'Vertical/CM', 'Endurance/Point', 'Endurance/Rank', 'Endurance/Levels']

# Create the second DataFrame
df2 = pd.DataFrame([rows[i:i+21] for i in range(0, len(rows), 21)], columns=columns2)
```

### 2.3 Merging and Exporting
The DataFrames are merged based on the 'Rank' column, and the final DataFrame is saved to an Excel file.


```python
# Merge the two DataFrames on the 'Rank' column
final_df = pd.merge(df, df2, on='Rank', how="outer")

# Print the final DataFrame
print(final_df)

#drop columns that contain _x (duplicte columns from merging) and rename columns to remove _y from column headers
columns_to_drop = [col for col in final_df.columns if '_x' in col]
final_df = final_df.drop(columns=columns_to_drop)
final_df.columns = [col.replace('_y', '') for col in final_df.columns]

# Save the final DataFrame to an Excel file
final_df.to_excel('UA_Data.xlsx', index=False)
```

## 3. Scraping Contestant's Attributes

### 3.1 Read Data from Excel, Format Names
```python
# Read data from Excel file
df = pd.read_excel('UA_Data.xlsx')

# Extract names and format them
# Names are in the format of ("firstname + " " + lastname), we need to change it to ("firstname + "-" + lastname) for the URL format
name_list = df['Name'].values.tolist()
formatted_names = [name.replace(' ', '-') for name in name_list]

# Initialize WebDriver
rows = []
driver = webdriver.Chrome()
driver.set_window_size(1280, 720)
first = True
```
### 3.2 Loop Through Formatted Names, Visit Athlete Pages, and Extract Information

``` python
for name in formatted_names:
    # Navigate to athlete page
    driver.get("https://competitioncorner.net/athletepage/" + name)
    time.sleep(2)
    
    # Sleep for additional time on the first iteration
    if first:
        time.sleep(8)
        first = False
    
    try:
        # Click agree to cookies (if present)
        ClickAgree = driver.find_element(By.XPATH, "//*[@id='agreeToCookie']/i")
        ClickAgree.click()
    except:
        pass

    try:
        # Wait for athlete information to be present
        Content = WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.CLASS_NAME, "card-athlete-info")))

        # Extract information from the athlete page
        items = Content.find_elements(By.XPATH, ".//mat-list-item[@role='listitem']")
        
        row = {"Name": name}
        for item in items:
            key = item.find_elements(By.XPATH, ".//span[string-length()>0 and not(@class)]")[0]
            val = item.find_elements(By.XPATH, ".//span[@class='ng-star-inserted']")
            if len(val) == 0:
                val = math.nan
            else:
                val = val[0].text
            row[key.text] = val
        rows.append(pd.Series(row))
    except Exception as ex:
        print("web driver exception", ex)
```

### 3.3 Create a DataFrame from Extracted Information and Save to Excel

```python
# Create a DataFrame from the extracted information
df3 = pd.DataFrame(rows)

# Save the DataFrame to a new Excel file
df3.to_excel('UA_Data_Attr.xlsx', index=False)
```
