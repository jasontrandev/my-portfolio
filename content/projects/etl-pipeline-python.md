---
title: "ETL Pipeline for Web Scraping"
description: "How to Build an ETL Pipeline with Python"
dateString: October 24, 2023
draft: false
tags: ["ETL", "Python", "Web Scraping", "SQLite", "Pandas"]
weight: 206
cover:
    image: "projects/etl-pipeline-python/cover.jpg"
--- 

ETL stands for Extract, Transform, Load. ETL is a type of data integration that extracts data from various sources (API, a database or a file), transforms it to match the destination system’s requirements and loads it into the destination system. ETL pipeline is an important type of workflow in data engineering. I will use Python and in particular pandas library to build a pipeline. Pandas make it super easy to perform ETL operations. I want to showcase how easy it is to streamline ETL process with Python. The full source code used for the ETL is available on [GitHub](https://github.com/jasontrandev/portfolio).

## Prerequisites
Before starting this project, you'll need the following:
- Windows OS
- GitHub Account
- Visual Studio Code
- DBeaver

## Setup
First, we will create a **virtual environment** in Python using this command:
``` python
python -m venv venv
```

Next, we need to install some necessary libraries to run the next lines of code
- requests
- pandas
- beautifulsoup4

**Activating** a virtual environment using a script in its binary directory
``` python
<venv>\Scripts\activate
```

Installing the above modules are done by running the following **pip** command in the terminal:
``` python
!pip install requests
!pip install pandas
!pip install beautifulsoup4
```

## Building ETL Pipeline
There are many different tools and technologies we can use to build an ETL pipeline. In this scope, we will use a core **Python, Pandas** and **SQLite3** to extract and load data into a destination system. 

### Extract
The first thing we need to do is the **Extract** process. In this stage, we must consider a few things.
1. The data source<br>
It is crucial to verify the various of source data. This is because the source determines how the data will be extracted. Data stored in an API will be ingested differently from data in HTML pages. Some common data sources:
    - APIs
    - Databases
    - Webpages
    - Flat files
    - IoT devices
    - ERP / CRM systems

2. The data format<br>
Formating data can also improve the performance and efficiency of the ETL process, as it reduces the amount of data cleansing and transformation required in the later stages. Below are a few examples that you will most likely encounter.
    - CSV
    - Parquet
    - JSON
    - HTML

In this scope, we will extract an **HTML pages** from this [link](https://tis.nhai.gov.in/tollplazasataglance.aspx?language=en#) using **BeautifulSoup** and **requests** library.

The extract function below call a request to a website above and converts it to a HTML page. After that, we use a **find** method to look for a tag named **class = 'PA15'** and then validate if it's found or not.

``` python
import requests
import pandas as pd
import sqlite3
from bs4 import BeautifulSoup
from datetime import date

def extract(self):
    response = requests.get(self.url)
    self.soup = BeautifulSoup(response.text, 'html.parser')
    if self.soup.find(class_='PA15'):
        return True
    return False
```

### Transform
Once **extraction process** is complete, we step forward to transform the data using **Pandas**. At this point, there are various transformations that can be done to data. Some common ones are:
1. handling missing values
2. formating data types
3. inserting and ordering columns

``` python
def transform(self):
    plaza_name = self.soup.find(class_='PA15').find_all('p')[0].find('lable')
    table_html = str(self.soup.find_all('table', class_='tollinfotbl')[0])
    df_info = pd.read_html(table_html)[0].dropna(axis=0, how='all')
    cols = df_info.columns.tolist()
    cols.insert(0, 'Date Scrapped')
    cols.insert(1, 'Plaza Name')
    cols.insert(2, 'TollPlazaID')
    df_info['Date Scrapped'] = date.today()
    df_info['Plaza Name'] = plaza_name.text
    df_info['TollPlazaID'] = self.plaza_id
    self.df_info = df_info[cols]
```

### Load
After **cleansing and transforming** the data in a previous step, the next thing is to load the clean data into **destination system**. Usually, it can be as following:
1. Database
2. Flat file
3. Data warehouse
4. Data lakehouse

In this scope, we will save the file back to a database engine **SQLite** using the load function below.

``` python
def load(self):
    with sqlite3.connect(self.sql_file_path) as conn:
        self.df_info.to_sql(self.sql_table_name, conn, if_exists='append', index=False)
```

### ETL Pipeline
Now, putting it all together, we have an end-to-end ETL pipeline as below.

``` python
import requests
import pandas as pd
import sqlite3
from bs4 import BeautifulSoup
from datetime import date


class ETL:

    def __init__(self, plaza_id, sql_file_path, sql_table_name):
        self.plaza_id = plaza_id
        self.sql_file_path = sql_file_path
        self.sql_table_name = sql_table_name
        self.url = f'https://tis.nhai.gov.in/TollInformation.aspx?TollPlazaID={plaza_id}'
        self.soup = ''
        self.df_info = pd.DataFrame

    def extract(self):
        response = requests.get(self.url)
        self.soup = BeautifulSoup(response.text, 'html.parser')
        if self.soup.find(class_='PA15'):
            return True
        return False

    def transform(self):
        plaza_name = self.soup.find(class_='PA15').find_all('p')[0].find('lable')
        table_html = str(self.soup.find_all('table', class_='tollinfotbl')[0])
        df_info = pd.read_html(table_html)[0].dropna(axis=0, how='all')
        cols = df_info.columns.tolist()
        cols.insert(0, 'Date Scrapped')
        cols.insert(1, 'Plaza Name')
        cols.insert(2, 'TollPlazaID')
        df_info['Date Scrapped'] = date.today()
        df_info['Plaza Name'] = plaza_name.text
        df_info['TollPlazaID'] = self.plaza_id
        self.df_info = df_info[cols]

    def load(self):
        with sqlite3.connect(self.sql_file_path) as conn:
            self.df_info.to_sql(self.sql_table_name, conn, if_exists='append', index=False)

    def run_etl(self):
        if self.extract():
            self.transform()
            self.load()
            print(f'Done with ETL of plaza_id {self.plaza_id}')
        else:
            print(f'Skipped plaza_id {self.plaza_id}')
```
<br>

Notice that we use a concept "**OOP Programming**" in Python to represents the ETL object and its method. Now, we have our complete and simple ETL pipeline.

## Executing the ETL job
Finally, we encapsulate every logic in the main file. At this point, we want to ensure we're having an object corresponding to each ID of plaza. To do that, we'll create an ETL object through ETL class. From here, you can call a **run_etl** function to run the entire pipeline and load the data into storage system. The code below demonstrates how to run pipeline, connect and store data in a **SQLite** database using **Pandas.**

``` python
from etl import ETL
from fetch_plaza_ids import get_all_plaza_ids
from datetime import date
from functools import partial
import concurrent.futures


def create_etl_object_and_run(plaza_id, db_file_path, db_table_name):
    plaza_etl = ETL(plaza_id, db_file_path, db_table_name)
    plaza_etl.run_etl()


if __name__ == "__main__":
    db_file_path = 'nhai_info.db'
    db_table_name = f'nhai_toll_info_{date.today()}'
    ids = get_all_plaza_ids()
    partial_etl_function = partial(create_etl_object_and_run, db_file_path=db_file_path, db_table_name=db_table_name)
    with concurrent.futures.ThreadPoolExecutor(10) as executor:
        executor.map(partial_etl_function, ids)
```
<br>

After running this command, you'll write the data to a file called **nhai_info.db**. You should use one database tool to read, design, and edit database files compatible with **SQLite**. In this case, I choose **DBeaver** to show the data.

![](/projects/etl-pipeline-python/img3.jpg)

Details usage of DBeaver and other tools, please see the links in Resources.

## Resources
- [DBeaver Documentation](https://dbeaver.com/docs/dbeaver/Create-Connection/)
- [DB Browser for SQLite (DB4S)](https://sqlitebrowser.org/)
- [DataGrip - The Cross-Platform IDE for Databases](https://www.jetbrains.com/datagrip/)
