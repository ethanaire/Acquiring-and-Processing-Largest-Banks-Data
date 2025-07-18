# Acquiring and Processing Information on the World's Largest Banks

[![Python 3.11](https://img.shields.io/badge/Python-3.11-green.svg)](https://shields.io/)

## Disclaimer

This repository contains my submission for the ***Final Project: Acquiring and processing information on world's largest banks***. The original files were provided by the IBM Skills Network as part of the **[Python Project for Data Engineering](https://www.coursera.org/learn/python-project-for-data-engineering)** course on Coursera.

## Objectives

* Extract real-world data from a public website using Webscraping and Requests API in Python
* Transform the data as per the problem statement
* Load the data in the required file format as well as a SQLite database
* Query the database to retrieve filtered information from the table

## Project Scenario

This project requires you to compile the list of the top 10 largest banks in the world ranked by market capitalization in billion USD. Further, you need to transform the data and store it in USD, GBP, EUR, and INR per the exchange rate information made available to you as a CSV file. You should save the processed information table locally in a CSV format and as a database table. Managers from different countries will query the database table to extract the list and note the market capitalization value in their own currency.

Project structure:

| Parameter                               | Value                                                                                                                             |
| --------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Code name                               | `banks_project.py`                                                                                                                |
| Data URL                                | `https://web.archive.org/web/20230908091635/https://en.wikipedia.org/wiki/List_of_largest_banks`                                  |
| Exchange rate CSV path                  | `https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMSkillsNetwork-PY0221EN-Coursera/labs/v2/exchange_rate.csv` |
| Table Attributes (upon Extraction only) | `Name`, `MC_USD_Billion`                                                                                                          |
| Table Attributes (final)                | `Name`, `MC_USD_Billion`, `MC_GBP_Billion`, `MC_EUR_Billion`, `MC_INR_Billion`                                                    |
| Output CSV Path                         | `./Largest_banks_data.csv`                                                                                                        |
| Database name                           | `Banks.db`                                                                                                                        |
| Table name                              | `Largest_banks`                                                                                                                   |
| Log file                                | `code_log.txt`                                                                                                                    |

## Directions

1. Write a function to extract the tabular information from the given URL under the heading By Market Capitalization, and save it to a data frame.
```python
def extract(url, table_attribs):
    ''' This function aims to extract the required
    information from the website and save it to a data frame. The
    function returns the data frame for further processing. '''

    page = requests.get(url).text
    soup = BeautifulSoup(page, "html.parser")

    df = pd.DataFrame(columns=table_attribs)

    tables = soup.find_all("tbody")
    rows = tables[0].find_all("tr")

    for row in rows:
        col = row.find_all("td")
        if len(col) != 0:
            data_dict = {"Name": col[1].find_all("a")[1]["title"],
                         "MC_USD_Billion": float(col[2].contents[0][:-1])}
            df1 = pd.DataFrame(data_dict, index=[0])
            df = pd.concat([df, df1], ignore_index=True)

    return df
```
2. Write a function to transform the data frame by adding columns for Market Capitalization in GBP, EUR, and INR, rounded to 2 decimal places, based on the exchange rate information shared as a CSV file.
```python
def transform(df, csv_path):
    ''' This function accesses the CSV file for exchange rate
    information, and adds three columns to the data frame, each
    containing the transformed version of Market Cap column to
    respective currencies'''

    # Read exchange rate CSV file
    exchange_rate = pd.read_csv(csv_path)

    # Convert to a dictionary with "Currency" as keys and "Rate" as values
    exchange_rate = exchange_rate.set_index("Currency").to_dict()["Rate"]

    # Add MC_GBP_Billion, MC_EUR_Billion, and MC_INR_Billion
    # columns to dataframe. Round off to two decimals
    df["MC_GBP_Billion"] = [np.round(x * exchange_rate["GBP"], 2) for x in df["MC_USD_Billion"]]
    df["MC_EUR_Billion"] = [np.round(x * exchange_rate["EUR"], 2) for x in df["MC_USD_Billion"]]
    df["MC_INR_Billion"] = [np.round(x * exchange_rate["INR"], 2) for x in df["MC_USD_Billion"]]
    
    return df
```
3. Write a function to load the transformed data frame to an output CSV file.
```python
def load_to_csv(df, output_path):
    ''' This function saves the final data frame as a CSV file in
    the provided path. Function returns nothing.'''

    df.to_csv(output_path)
```
4. Write a function to load the transformed data frame to an SQL database server as a table.
```python
def load_to_db(df, sql_connection, table_name):
    ''' This function saves the final data frame to a database
    table with the provided name. Function returns nothing.'''

    df.to_sql(table_name, sql_connection, if_exists='replace', index='False')
```
5. Write a function to run queries on the database table.
```python
def run_query(query_statement, sql_connection):
    ''' This function runs the query on the database table and
    prints the output on the terminal. Function returns nothing. '''

    print(query_statement)
    query_ouput = pd.read_sql(query_statement, sql_connection)
    print(query_ouput)
```
7. Run the following queries on the database table:
    - Extract the information for the London office, that is Name and MC_GBP_Billion
      ```python
      query_statement = f"SELECT * from {table_name}"
      run_query(query_statement, sql_connection)
      ```
    - Extract the information for the Berlin office, that is Name and MC_EUR_Billion
      ```python
      query_statement = f"SELECT AVG(MC_GBP_Billion) FROM {table_name}"
      run_query(query_statement, sql_connection)
      ```
    - Extract the information for New Delhi office, that is Name and MC_INR_Billion
      ```python
      query_statement = f"SELECT Name from {table_name} LIMIT 5"
      run_query(query_statement, sql_connection)
      ```
8. Write a function to log the progress of the code.
9. While executing the data initialization commands and function calls, maintain appropriate log entries.

## Setup

Install the required libraries using the provided `requirements.txt` file. The command syntax is:

```bash
python3.11 -m pip install -r requirements.txt
```

Download the required exchange rate file using the terminal command:

```bash
wget https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMSkillsNetwork-PY0221EN-Coursera/labs/v2/exchange_rate.csv
```

Execute the code using the command:

```bash
python3.11 banks_project.py
```

## Learner

[Hai Hoang Nguyen](https://www.linkedin.com/in/hoang-ngn/)

## Acknowledgments

* IBM Skills Network © IBM Corporation 2025. All rights reserved.
