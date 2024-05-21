# Case Study: Analyzing the Growth of the Top 3 Oil and Gas Companies in Indonesia by Market Cap and Their Relationship with Indonesia's Oil and Gas Export Values (2021-2023)
### Author: Fedri Anwari, B.A.
### Created date: 22/05/2024

![Screenshot 2024-05-18 220921](https://github.com/MrAnwari/casestudy/assets/162018078/af031984-0101-4015-9306-30d92e556ead)


## Introduction

This case study aims to explore the comparative dynamics between Indonesia's oil and gas export values and the stock prices of the top three oil and gas companies by market capitalization, which are $AKRA, $MEDC, and $PGAS, over the period from 2021 to 2023. By analyzing this timeframe, we seek to understand how fluctuations in export values impact the market performance of these major companies. Given the significant role of the oil and gas sector in Indonesia's economy, this study will provide valuable insights into the correlation between export performance and stock market valuation, offering a comprehensive view of the sector's financial health and investor sentiment during these years.

In order to answer the business questions, follow the steps of the data analysis process: Ask, Prepare, Process, Analyze, Share, and Act.

## Ask
Three questions this case study tries to discover:
1. How did the performance of these 3 stocks compare to Indonesia's Oil & Gas Export Values from Q1 2021 to Q4 2023, and what factors influenced any disparities in performance?
2. What is the degree of correlation between the performance of each of these 3 stocks and Indonesia's Oil & Gas Export Values from Q1 2021 to Q4 2023?
3. What insights can be derived for future investment strategies from these interactions?

## Prepare
We utilize two public datasets for this analysis. The first dataset presents historical stock price data for $AKRA, $MEDC, and $PGAS from either their initial public offering (IPO) date or from April 16, 2001, until January 6, 2023. This dataset was made available by Kaggle user @muamkh and has received an 8.4 usability score on Kaggle, indicating positive reception within the Kaggle community.

The second dataset provides information on the value of Indonesia's oil exports from 2021 to 2023, and it is sourced from Badan Pusat Statistik (BPS), Indonesia's Central Statistics Agency. BPS is a reputable governmental organization responsible for collecting, processing, and disseminating statistical data in Indonesia. The dataset's credibility and trustworthiness stem from BPS's established reputation for maintaining high standards of data collection, accuracy, and transparency. As a government agency, BPS adheres to rigorous methodologies and protocols in gathering and validating statistical information, ensuring reliability for policymaking, research, and decision-making purposes. Therefore, the dataset from BPS is widely regarded as credible and trusted within both academic and professional circles.

Link to the first dataset [here](https://www.kaggle.com/datasets/7415fcfb5010d225fc1951e426d81c89cf473603d49f46bc6553bd3fe2688b68)

Link to the second dataset -> [here](https://www.bps.go.id/en/statistics-table/2/MTc1MyMy/value-of-export-oil-gas---non-oil-gas--million-us--.html)

### Steps taken during this process:
1. Download and store the dataset. For the first dataset, go to the daily folder and download only the akra.csv, medc.csv, and pgas.csv files. For the second dataset, download only the data from 2021 to 2023.
2. Check how the data is organized. It organized in CSV format.
3. We have verified the credibility of both datasets and ensured they are not biased. As previously explained, both datasets are trusted and credible sources. Furthermore, for the first dataset obtained from Kaggle, we conducted additional verification by cross-referencing it with the stock price data provided by [tradingview.com](tradingview.com). Through this process, I can confirm that both datasets meet the criteria of being Reliable, Original, Comprehensive, Current, and Cited (ROCCC).
4. Prepare the data by sorting and filtering it on Google BigQuery using SQL and Google Sheet. 

## Process
### Steps taken during this process:

1) The first dataset will be processed and cleaned on a Google Big Query using SQL since it is a large dataset which contains over 5k rows. 
The steps are as follows:
- Create a dataset on BigQuery and create a table by uploading the three files we have downloaded: akra.csv, medc.csv, and pgas.csv.
- Open each file, check the schema, and open the preview menu to ensure the data are complete and in the correct format. For the timestamp column, the data should be in DATE format, and the price data should be in Integer format.
- The dataset provides historical daily price data for each stock from 2001 or their respective IPO dates up to 2022. Since I only require the data from 2021 to 2022 and specifically the closing price data, filter the dataset by executing the following query:

```sql
SELECT  
  timestamp,
  close
FROM
`mypractice-421606.case_study.akra`
WHERE timestamp BETWEEN ‘2021-01-01’ AND ‘2023-01-01’ 
ORDER BY timestamp;
```

- Open a new query tab. Since we need monthly closing price data and the data presents daily closing price data, we will use the previous query result to manually find the end date for each month. Then, we will use these end dates to further filter the data. We filter it by writing the following query:

```sql
SELECT  
  timestamp,
  close
FROM
`mypractice-421606.case_study.akra`
WHERE timestamp IN ('2021-01-29', '2021-02-26', '2021-03-31', '2021-04-30', '2021-05-31', '2021-06-30', '2021-07-30', '2021-08-31', '2021-09-30', '2021-10-29', '2021-11-30', '2021-12-31',
'2022-01-03', '2022-02-28', '2022-03-31', '2022-04-29', '2022-05-31', '2022-06-30', '2022-07-29', '2022-08-31', '2022-09-30', '2022-10-31', '2022-11-30', '2022-12-30')
ORDER BY timestamp;
```

Repeat the same process for the other two tables (medc and pgas) by simply changing the table name in this query:

```sql
FROM
`mypractice-421606.case_study.xxxx`
```
- Save the query results in CSV format and name the files akra_2021-2023.csv, medc_2021-2023.csv, and pgas_2021-2023.csv. 
- Upload those files/tables back to BigQuery. We can remove the old tables since we won’t use them anymore.
- Join the tables. Please write the following query to join those three tables into one for easier analysis, and save the query result in csv format:

```sql
SELECT
    t1.timestamp, t1.close AS AKRA, t2.close AS MEDC, t3.close AS PGAS
FROM
    `mypractice-421606.case_study.akra_2021-2023` AS t1
INNER JOIN
    `mypractice-421606.case_study.medc_2021-2023` AS t2 ON t1.timestamp = t2.timestamp
INNER JOIN
    `mypractice-421606.case_study.pgas_2021-2023` AS t3 ON t1.timestamp = t3.timestamp;
```

- Upload the joined table back to BigQuery and name it stock_price_2021-2023. We can remove the old tables since we won’t use them anymore.
- Since the data is only available until 2022, we will use proxy data from tradingview.com for the monthly closing prices of each stock for the 2023 timeframe. Use the following query to add this data to the table:

```sql
INSERT INTO `mypractice-421606.case_study.stock_price_2021-2023` (timestamp, akra, medc, pgas)
VALUES
  ('2023-01-31', 1310, 1395, 1545),
  ('2023-02-28', 1385, 1150, 1565),
  ('2023-03-31', 1550, 1010, 1380),
  ('2023-04-28', 1620, 1010, 1430),
  ('2023-05-31', 1365, 905, 1430),
  ('2023-06-27', 1420, 890, 1305),
  ('2023-07-31', 1385, 1130, 1365),
  ('2023-08-31', 1400, 1070, 1375),
  ('2023-09-29', 1545, 1610, 1375),
  ('2023-10-31', 1490, 1275, 1255),
  ('2023-11-30', 1435, 1155, 1115),
  ('2023-12-29', 1475, 1155, 1130);
```
- The data is cleaned and ready to be analyzed. Download the data in xlsx format.

2) The second dataset will be processed and cleaned on a spreadsheet since it is a small dataset. The steps are as follows:
- We only used the data needed for this analysis, which is the oil & gas export values, and removed the rest as it won’t be needed.
- We noticed that this dataset has three separate sets of data based on the timeframes: 2021, 2022, and 2023. We will put all three datasets into one file but in separate sheets, naming each sheet based on the respective timeframe.
- The data is in a wide format. Since long data format is better suited for graphing and statistical analysis, We converted it to a long data format using the following formula for easier processing:
```excel
=ARRAYFORMULA({
  FLATTEN(range);
  FLATTEN(range)
})
```
- For easier analysis, we combined all three datasets into one sheet. Before doing that, we changed the data format in the timeframe column from Month (e.g., January, February, March) to the custom date format: Jan_21, Jan_22, Feb_22, Apr_23, and so on. Next, we combined the data by simply copying and pasting it, as the dataset is small. If you are working with a larger dataset, you can combine it using =QUERY & =IMPORTRANGE, or =ARRAYFORMULA.

## Analyze
Now that the data is stored appropriately and has been prepared for analysis, start putting it to work.

### Steps taken during this process:
1. First Analysis
- Combine the two datasets into one table in spreadsheet.
- Calculate the monthly percentage growth using this example formula:
```excel
=(B2-B3)/B2
```
*To calculate the January 2021 growth data, check tradingview.com for the stock prices in January 2021 and refer to the BPS database for the export values in December 2020. Then, use these values to calculate the growth.*

- Calculate the yearly growth, overall growth, and average monthly growth.
- Calculate the number of 'Green months,' 'Red months,' and 'Unchanged months' for each of the stock price and export values growth using the following formula:
```excel
=COUNTIF(range,>0)
=COUNTIF(range,<0)
=COUNTIF(range,=0)
```

2. xxx


This repository showcases the process and completion of the case study project for the Google Data Analytics Certificate.
