---
title: "Building Data Warehouse"
description: "How to build Data Warehouse with Azure Synapse"
dateString: June 10, 2024
draft: false
tags: ["Data Engineer", "Data Modeling", "Data Warehouse", "NetSuite", "ETL"]
weight: 201
cover:
  image: "/projects/build-data-warehouse/cover.png"
---

## Prerequisites

To understand all the techniques of designing both data model and data warehouse, you need to be equiped with robust skills set and knowledge as followings:

- ER Diagram
- OLTP (Online Transaction Processing)
- OLAP (Online Analytical Processing)
- ETL

## Step-by-step Guide

1. Understanding transaction system
2. Collecting business requirements
3. Building data warehouse on Azure Synapse

## Understanding Transaction System

TrueCommerce is a high-performing global supply chain network that provides EDI (Electronic Data Interchange) solution to exchange key order documents between you and your supply chain partners automatically.

At TrueCommerce, our software system includes many sub systems, each of it serves for difference platforms. In this scope, we will focus on **NetSuite** which is a cloud-based ERP and some modules on it.

- ### Data Model for Case Management

![](/projects/build-data-warehouse/img1.jpg)

| No  | Table         | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| :-: | :------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
|  1  | Employee      | Store all employees/supervisor/department/country in UKG system                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
|  2  | Customer      | Store all customers signed contract with TrueCommerce in NetSuite system                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|  3  | Subsidiary    | The subsidiary record enables you to manage data for a hierarchical structure of seperate legal entities                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|  4  | HJSupportTeam | Store support teams mapped with multiple employee                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
|  5  | EmployeeTeam  | Store teams contain employee information and platform                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|  6  | CreateCases   | Cases track issues your customers report and the responses your support representatives give. Cases are created when your customers report problems/ask questions/communicate. <br> Cases are created in four ways: <br> <ul><li>A support rep creates a case record in NetSuite for customer who calls in.</li> <li>A customer completes an online case form. </li> <li>A customer sends an email to your support address.</li> <li>A customer clicks the Contact Support link in the Customer Center or your Web site and fills out an external case record.</li></ul>                                                                    |
|  7  | CloseCases    | When you reply to a customer with a definitive answer, you can change the case status to **Closed**. Closed cases are saved in your Cases list, but the list filters out closed cases, by default. You can view closed cases, but they are not visible with the open cases. <br> If a customer replies to the email message sent from the case, the case is re-opened. <br> If the customer clicks Contact Support or an online case form link again, a new case is created, and the case workflow begins again. <br> A case can be closed multiple times due to re-opened. Any changes in case status will be captured in log system note. |
|  8  | LogCase       | Track changes made to a record of a case. A system note for a change on a record captures the date when the change was made, who made the change, the role of the user who made the change, the type of change, and the old and new value in the record.                                                                                                                                                                                                                                                                                                                                                                                    |

- ### Data Model for Project Management

## Collecting Business Requirements

Depending on cross-functional department, we might have some different metrics by level, e.g., director/manager/leader. In this scope, our documentation define all list of metrics and describe a formula corresponding with it.

| ID  | Mesurements             | Description                                                                                                                                                                                                        |
| :-: | :---------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|  1  | Cases Inflow            | All creation cases come in as many different ways including Portal Submission, In-App Chat Support, Call into Support Queue, E-Mail Received                                                                       |
|  2  | Cases Outflow           | All closure cases that occured the last time, ignore re-opened cases                                                                                                                                               |
|  3  | Average Time to Resolve | Average duration (hour) of cases resolved from creation date to closure date excluding "Non-Timer" values. <br> <ul><li>Awaiting Customer Reply</li> <li>On Hold</li> <li>Resolution/Workaround Provided</li></ul> |
|  4  | Re-opened Tickets       | How many cases is re-opened when it had been closed                                                                                                                                                                |
|  5  | % Share of Cases Closed | Percentage of cases closed broken down by region                                                                                                                                                                   |

## Building Data Warehouse on Azure Synapse

After completing whole business requirements with stackholders, we have to organize a central repositories where it can store structured and semi-structured data from various sources.

When building a data warehouse with Azure Synapse, there are tons of tasks job to complete end-to-end data pipeline, so I summarize some basic steps:

1. Ingest data
   <br> The first step is to extract data source from OLTP system. Our company use NetSuite SaaS applications and provides RESTful APIs so I use **generic REST connector** in Azure Synapse to copy data from and to a REST endpoint.

   ![](/projects/build-data-warehouse/img2.png)
   ![](/projects/build-data-warehouse/img3.png)

2. Transform data
   <br> Data transformation activities in Synapse pipelines can help you to transform and process raw data. There are several ways to do it and we choose **mapping data flows** to clean data with low-code.

   ![](/projects/build-data-warehouse/img4.png)

3. Create pipelines
   <br> The activities in a pipeline define actions to perform on your data. For example, say you have a pipeline that executes at 8:00 AM, 9:00 AM, and 10:00 AM. A pipeline can be executed either manually or by using a **trigger**.

   ![](/projects/build-data-warehouse/img5.png)

4. Create triggers
   <br> To kick off all pipelines run data periodically (hourly, daily, etc.), you need to create a **schedule trigger**, configure parameters (start date, recurrence, end date etc.) for the trigger, and associate with a pipeline. In this project, I set a trigger run daily at **1PM** to capture in a previous day's data because TrueCommerce headquarters is located in US country and our stakeholders need reports according to US time zone.

   ![](/projects/build-data-warehouse/img6.png)
   ![](/projects/build-data-warehouse/img7.png)

5. Monitor pipeline
   <br> With Azure Synapse Analytics, you can create complex pipelines that can automate and integrate your data movement, data transformation, and compute activities within your solution. You can author and monitor these pipelines using Synapse Studio.

   ![](/projects/build-data-warehouse/img8.png)

## Resources

A few links that can be a great document to understand deep dive into data warehouse with Azure Synapse

- [MyProject - ETL Pipeline with Azure Synapse](/content//projects/truecommerce-company/azure-synapse-netsuite-api.md)
- [Data transformation activities](https://learn.microsoft.com/en-us/azure/data-factory/concepts-pipelines-activities?tabs=synapse-analytics#data-transformation-activities)
