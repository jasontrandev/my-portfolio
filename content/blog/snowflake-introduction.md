---
title: "Introduction to Snowflake"
description: "Basic information and instructions for beginner users of Snowflake."
dateString: September 07, 2024
draft: false
tags: ["Snowflake", "Data Engineer", "Data Warehouse", "Data Lake"]
weight: 201
cover:
  image: "/blog/snowflake-introduction/cover.jpg"
---

## What is Snowflake?

Developed in 2012, **Snowflake** is a fully managed **Data Cloud** that provides a single platform for data warehousing, data lakes, data engineering, data science, data application development, and secure sharing and consumption of real-time / shared data. It operates on Amazon Web Services, Microsoft Azure, and Google Cloud Platform.

## Snowflake Architecture

Snowflake uses a central data repository for persisted data that is accessible from all compute nodes in the platform.

![](/static/blog/snowflake-introduction/architecture.png)

Snowflake's architecture consists of three key layers:

- Storage
- Compute
- Cloud Services

### Storage

The storage layer is responsible for managing and storing data efficiently. When data is loaded into Snowflake, Snowflake reorganize that data into its internal optimized, compressed, columnar format.

The functions supported by the storage layer are:

- **Elasticity:** Allowing organizations to scale their storage needs independently of their compute resources. It ensures handling of varying data volumes without compromising performance

- **Cloud-Based Object Storage:** Separation of storage and compute allows for cost-effective and scalable data storage

- **Data Clustering:** Snowflake organizes data into small partitions in object storage, and these small partitions are clustered based on metadata.

- **Zero-copy Dedupliaction:** Snowflake enables efficient data replication. This feature allows users to create a copy of a data set without copying the actual data, saving both time and storage costs.

### Compute (Virtual Warehouse)

Also called **Query Processing** which performed in the processing layer. Snowflake processes queries using "virtual warehouses". Each virtual warehouse is an MPP (massively parallel processing) compute cluster composed of multiple computes nodes allocated by Snowflake from a cloud provider.

> _**Note**: Each virtual warehouse has no impact on the performance of other virtual warehouses._

### Cloud Services

Services managed in this layer include:

- Authentication
- Infrastructure management
- Metadata management
- Query parsing and optimization
- Access control

## Key features

Time Travel

## Single Platform

Snowflake's single platform eliminates the need for multiple systems to handle different data workloads. Its ability to handle data warehouseing, data lakes, real-time analytics, and machine learning in one place simplifies architecture, reduce costs, and accelerates time to insight.

There are three main layers that form the basis of the solution:

- Optimized Storage (Data Management)
- Elastic Performance Engine
- Intelligent Infrastructure

![](/static/blog/snowflake-introduction/platform.png)

## Classic Console vs Snowsight

Snowflake has two different UI with common features, the more robust one is the Classic Console which has specific features (yet in 2022), and the new one Snowsight which is more user-friendly. For most of the tasks, you can do in both.

![](/static/blog/snowflake-introduction/ui-classic-console.png)

![](/static/blog/snowflake-introduction/ui-snowsight.png)

## Connecting to Snowflake

Snowflake support multiple ways of connecting to the service:

- A web-based user interface
- Command line clients (e.g. SnowSQL)
- ODBC and JDBC drivers that can be used by other applications (e.g. Tableau)
- Native connectors (e.g. Python, Spark) that can be used to develop application
- Third-party connectors such as ETL tools (e.g. Informatica) and BI tools

## Virtual Hands-on Lab

This is a hands-on tutorial - Zero to Snowflake in 90 mins to help you master Snowflake basics and are ready to apply these fundamentals to your own projects.

[Github Repository](https://github.com/jasontrandev/snowflake-labs/tree/main/samples/0-getting-started-with-snowflake)

## Reference

- https://www.snowflake.com/virtual-hands-on-lab/
