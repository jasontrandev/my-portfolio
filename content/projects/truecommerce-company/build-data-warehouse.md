---
title: "Building Data Warehouse"
description: "How to build Data Warehouse with Azure Synapse"
dateString: June 10, 2024
draft: false
tags:
  ["Data Engineer", "Data Modeling", "Data Warehouse", "NetSuite", "ETL", "ERD"]
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

## Step-by-Step Guide

1. &nbsp;Understanding transaction system
2. &nbsp;Collecting business requirements
3. &nbsp;Building data warehouse on Azure Synapse

## Understanding Transaction System

TrueCommerce is a high-performing global supply chain network that provides **EDI (Electronic Data Interchange)** solution to exchange key order documents between you and your supply chain partners automatically.

At TrueCommerce, software system includes many sub systems, each of it serves for different platforms. This section will only focus on **NetSuite ERP** which is a SaaS cloud-based that helps organizations operate more effectively by having some modules on system. To make it simple, this project will only concentrate on case management.

- <span style="font-size:1.1em; font-weight:bold">Data Model for Case Management</span>

![](/projects/build-data-warehouse/img1.0.png)

<table style="width:100%; border: 1px solid">
  <tr>
    <th>No</th>
    <th>Table</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>1</td>
    <td>Employee</td>
    <td>Store all employees/supervisor/department/country in UKG system</td>
  </tr>
  <tr>
    <td>2</td>
    <td>Customer</td>
    <td>Store all customers signed contract with TrueCommerce in NetSuite system</td>
  </tr>
  <tr>
    <td>3</td>
    <td>Subsidiary</td>
    <td>The subsidiary record enables you to manage data for a hierarchical structure of seperate legal entities</td>
  </tr>
    <td>4</td>
    <td>HJSupportTeam</td>
    <td>Store support teams mapped with multiple employee</td>
  </tr>
    <td>5</td>
    <td>EmployeeTeam</td>
    <td>Store teams contain employee information and platform</td>
  </tr>
    <td>6</td>
    <td>CreateCases</td>
    <td>
    Cases track issues your customers report and the responses your support representatives give. Cases are created when your customers report problems/ask questions/communicate. <br> Cases are created in four ways: <br> 
      <ul>
        <li>A support rep creates a case record in NetSuite for customer who calls in.</li>
        <li>A customer completes an online case form.</li>
        <li>A customer sends an email to your support address.</li>
        <li>A customer clicks the Contact Support link in the Customer Center or your Web site and fills out an external case record.</li>
      </ul>
    </td>
  </tr>
    <td>7</td>
    <td>CloseCases</td>
    <td>When you reply to a customer with a definitive answer, you can change the case status to **Closed**. Closed cases are saved in your Cases list, but the list filters out closed cases, by default. You can view closed cases, but they are not visible with the open cases. <br> If a customer replies to the email message sent from the case, the case is re-opened. <br> If the customer clicks Contact Support or an online case form link again, a new case is created, and the case workflow begins again. <br> A case can be closed multiple times due to re-opened. Any changes in case status will be captured in log system note</td>
  </tr>
  </tr>
    <td>8</td>
    <td>LogCase</td>
    <td>Track changes made to a record of a case. A system note for a change on a record captures the date when the change was made, who made the change, the role of the user who made the change, the type of change, and the old and new value in the record</td>
  </tr>     
</table>

<br>
Let's talk about the life cycle of case management. When a case is first created by a customer, NetSuite system will add one record. Next, every time there is an update value, it will replace an old record by overwriting new value to an exist record, not create a new record in system. For example, a <b>status field</b> starting with a value 'Not Started' and can be updated to one of the values, like 'In Progress', 'On Hold', 'Awating Customer Reply', 'Closed' etc, .

<br>
All changes history has been saved to a log in NetSuite tab called <b>System Notes</b>. This tab can be used to track changes whenever stakeholders need to query historical data.

![](/projects/build-data-warehouse/img1.1.png)

## Collecting Business Requirements

Depending on cross-functional department, **business metrics** are quantifiable measures used to track business processes to judge the performance level of an organization. With so many possible business metrics, it's important to choose the ones that matter most to your company. In this scope, our documentation define all list of metrics for director/manager belongs to the **Support/Services department**.

<table style="width:100%; border: 1px solid">
  <tr>
    <th>ID</th>
    <th>Measurements</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>A01</td>
    <td>Cases Inflow</td>
    <td>All creation cases come in as many different ways including Portal Submission, In-App Chat Support, Call into Support Queue, E-Mail Received</td>
  </tr>
  <tr>
    <td>A02</td>
    <td>Cases Outflow</td>
    <td>All closure cases that occured the last time, ignore re-opened cases</td>
  </tr>
  <tr>
    <td>A03</td>
    <td>Average Time to Resolve</td>
    <td>Average duration (hour) of cases resolved from creation date to closure date excluding "Non-Timer" values <br> <ul><li>Awaiting Customer Reply</li> <li>On Hold</li> <li>Resolution/Workaround Provided</li></ul></td>
  </tr>
  <tr>
    <td>A04</td>
    <td>Re-opened Tickets</td>
    <td>How many cases are re-opened when it had been closed</td>
  </tr>
  <tr>
    <td>A05</td>
    <td>% Share of Cases Closed</td>
    <td>Percentage of cases closed broken down into region</td>
  </tr>  
</table>

## Building Data Warehouse on Azure Synapse

After completing whole business requirements with stakeholders, we need to organize a **central repositories** where it can store **structured and semi-structured data** from various sources.

When building a data warehouse with Azure Synapse, there are tons of tasks job to complete end-to-end data pipeline, so I summarize some of the basic steps:

<span style="font-size:1.1em; font-weight:bold">1.&nbsp; Ingest data</span>
<br> The first step is to extract data source from OLTP system. Our company use NetSuite SaaS applications and provides RESTful APIs so I use **generic REST connector** in Azure Synapse to copy data from and to a REST endpoint.

![](/projects/build-data-warehouse/img2.0.png)
![](/projects/build-data-warehouse/img2.1.png)

<span style="font-size:1.1em; font-weight:bold">2.&nbsp; Transform data</span>
<br> Data transformation activities in Synapse pipelines can help you to transform and process raw data. There are several ways to do it and we choose **mapping data flows** to clean data with low-code.

![](/projects/build-data-warehouse/img3.0.png)

Once the process of data cleaning and preparation is completed, we might want to load it into data warehouse by creating external table from **Azure data lake**.

![](/projects/build-data-warehouse/img3.1.png)

<span style="font-size:1.1em; font-weight:bold">3.&nbsp; Create pipelines</span>
<br> The activities in a pipeline define actions to perform on your data. For example, a pipeline could contain a set of activities that ingest and clean log data, and then kick off a mapping data flow to analyze the log data. A pipeline can be executed either manually or by using a **trigger**.

![](/projects/build-data-warehouse/img4.png)

<span style="font-size:1.1em; font-weight:bold">4.&nbsp; Create triggers</span>
<br> To kick off all pipelines run data periodically (hourly, daily, etc.), you need to create a **schedule trigger**, configure parameters (start date, recurrence, end date etc.) for the trigger, and associate with a pipeline. In this project, I set a trigger run **daily at 1AM (EST)** to capture in a **previous day's data** because TrueCommerce headquarters is located in US country and our stakeholders need dashboards are refreshed according to **US time zone**.

![](/projects/build-data-warehouse/img5.1.png)
![](/projects/build-data-warehouse/img5.2.png)

<span style="font-size:1.1em; font-weight:bold">5.&nbsp; Monitor pipeline</span>
<br> With Azure Synapse Analytics, you can create complex pipelines that can automate and integrate your data movement, data transformation, and compute activities within your solution. You can author and monitor these pipelines using Synapse Studio.

![](/projects/build-data-warehouse/img6.png)

## Resources

A few links that can be a great document to understand deep dive into data warehouse with Azure Synapse

- [MyProject - ETL Pipeline with Azure Synapse](https://jasontran.netlify.app/projects/truecommerce-company/azure-synapse-netsuite-api/)
- [Data transformation activities](https://learn.microsoft.com/en-us/azure/data-factory/concepts-pipelines-activities?tabs=synapse-analytics#data-transformation-activities)
