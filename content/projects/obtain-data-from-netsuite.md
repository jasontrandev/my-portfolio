---
title: "How to Obtain Data from NetSuite for Analytics"
description: "Different ways to connect data from NetSuite in a custom application"
dateString: March 15, 2023
draft: false
tags: ["Azure", "Power BI", "NetSuite"]
showToc: false
weight: 202
cover:
    image: "projects/obtain-data-from-netsuite/cover.jpg"
--- 

From surfacing data in a custom application to visualizing data in Power BI reports, there are multiple ways to get your data out of NetSuite. This article will explain the "analytics stack" the Customer Success Operations team uses to report on data sourced from NetSuite (as well as other sources) and provide the information required for you to tap in at any level of that stack.

## Using the Saved Search Data Source API
The next level up in our stack is the Saved Search Data Source API. This is a containerized API that accepts a saved search ID and page number and returns a JSON array of search results. This API is used by our Azure Synapse data flow automations, but additional instances of the API can be spun up for other uses if needed. This API can be used by a custom app or called directly by Power BI or Excel using an HTTP connector.

If you are interested in using this approach, please reach out to CS-OPS Engineering to get an instance of the API provisioned.

## Using the Data Lakehouse
The final layer in the stack is our "data lakehouse". The lakehouse is a virtual SQL database that contains a curated data model that consists of data from NetSuite, TC.NET, and other disparate data sources. There are a few advantages of using the lakehouse:
- The data is immediately available by using any SQL Server client (including Power BI, Excel, SQL Server Management Studio, Azure Data Studio, etc).
- It is "source agnostic", so while we do track the original primary key and name of the source system, one can easily join data across multiple sources into one result set.
- The schema of the lakehouse's "CoreLibrary" database is heavily curated, but additional domain-specific databases can be provisioned to expose more customized data models for reporting.

If you are interested in using the lakehouse, please see the [Data Library](/projects/data-library) documentation for information on how to connect.

If you would like additional data sources (including NetSuite records and fields) added to the lakehouse, please reach out to CS-OPS Engineering. Additional data can be added relatively quickly.