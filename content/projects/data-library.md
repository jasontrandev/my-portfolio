---
title: "Data Library"
description: "How to access and connect to a Data Lakehouse on Azure Synapse"
dateString: April 03, 2023
draft: false
tags: ["Azure", "Power BI", "Azure Data Studio"]
showToc: false
weight: 203
cover:
    image: "projects/data-library/cover.jpg"
--- 

## Overview
All of our reports rely on data that is spread across disparate systems within the organization. To improve the quality of data that is fed into our reports, we've curated a central Data Lakehouse - a data source that aggregates information from various databases, tools, and products into a one-stop shop for your analytics platform of choice.

## How it Works
We use a process known as "Extract, Transform, Load" (or ETL) to extract data from a system, transform it into a clean, meaningful set of data, and load it into a library for consumption. The exact ETL pipeline for each record type or set of data may be different, but we document that as well so you know exactly where your data comes from.

The resulting data library is known as a "Data Lakehouse". This term is a combination of "Data Lake" and "Data Warehouse" because it combines the benefits of both into one platform.

## Accessing the Data Lakehouse

### Using Power BI or Excel
This method of access is most common as it brings the data directly into a BI/reporting tool where it is often needed most.

**Option1: Basic Import**

This option is best for smaller datasets as all records for the selected tables will be imported into your report or spreadsheet.
In the **Get Data** dialog, choose the **Azure Synapse Analytics workspace** connector, then choose the **tcops-analytics​​​​​​**​ workspace.

**Option2: SQL Import or SQL DirectQuery**

This option is best for larger datasets as you can use T-SQL to import a subset of data or use DirectQuery to pull data from the Lakehouse on demand instead of importing it into your report or spreadsheet.

In the **Get Data** dialog, choose the **Azure Synapse Analytics SQL** connector.

|            |           |
| ---------: | --------- | 
|__Server__| tcops-analytics-ondemand.sql.azuresynapse.net |
|__Available Databases__| ​​​​​​​B2BGateway, NetSuiteSearches, TimecardService |

### Using Azure Data Studio
This method of access is great for developers or power users that want to explore the data using SQL queries before building visual reports or use Notebooks to analyze data.

|            |           |
| ---------: | --------- | 
|__Connection Type__| Microsoft SQL Server |
|__Server__| ​tcops-analytics-ondemand.sql.azuresynapse.net |
|__Authentication Type__| Azure Active Directory - Universal with MFA Support |
|__Available Databases__| B2BGateway, NetSuiteSearches, TimecardService |