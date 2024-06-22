---
title: "SCD in Data Warehouse"
description: "How to apply Slowly Changing Dimension (SCD) to organize a Data Warehouse"
dateString: June 20, 2023
draft: false
tags: ["Data Warehouse", "Power BI"]
weight: 204
cover:
    image: "projects/scd-data-warehouse/cover.jpg"
--- 

A **slowly changing dimension (SCD)** is a data warehousing concept, which manages the change of dimension over time. This means that it uses to track of a historical and current values for analytical data. A good example of an SCD is a customer and employee dimension. SCD is about how changes in the source systems reflect the data in data warehouse.

## Overview
Dealing with SCDs is not always as simple as you might think. From a data warehousing perspective, we have different options to deal with the data depending on the business requirements, leading us to different [types of SCDs](https://learn.microsoft.com/en-us/training/modules/populate-slowly-changing-dimensions-azure-synapse-analytics-pipelines/3-choose-between-dimension-types?ns-enrollment-type=learningpath&ns-enrollment-id=learn.wwl.data-integration-scale-azure-data-factory). This topic will go through a few cases where SCD is applicable.

## Case Study 1
**Scenario:** In TrueCommerce DiCentral company, a **human resources (HR)** system having an **Employee** dimension including many fields: *Employee Number, Employee Name, Employee Email, Title, Supervisor Name, Start Date*.

**Our goal:** Keep the history of data changes in Employee dimension. So, we choose SCD type 2 (SCD 2) to maintain the historical employee data.

To apply the SCD 2 technique, please remember that we cannot use the **EmployeeNumber** column as the primary key of the dimension. Hence, we need to introduce a new set of columns to guarantee the uniqueness of every row of the data, as follows:
- A new key column that guarantees rows' uniqueness in the **Employee** dimension. This new key column is simply an index representing each row of data stored in a data warehouse dimension. But, we still need to maintain the source system's primary key.
- A **Start Date** and an **End Date** column represent the timeframe during which a row of data is in its current state.
- Another column shows the status of each row of data.

The following screenshot shows the data in the **Employee** dimensions in the data warehouse before it changes a few items:

![](/projects/scd-data-warehouse/img1.jpg)

The **StartDate** column shows the date **Lai, Tan Dat** started his job as **Integration Mapping Anlyt**, the **End Date** column has been left blank (null), and the **Status** column shows **Current**. Let's assume that we change some information about the employee as following:
- Lai, Tan Dat was changed from Services to Support department.
- Add a new employee named Phung, Ngoc Toan under Pham, Van Tri leader. 

![](/projects/scd-data-warehouse/img2.jpg)

As the above image shows, **Lai, Tan Dat** started his new department as **650-Worldwide Support** on **16/12/2023** and finished his job as **610-Professional Services** on **15/12/2023**. Phung, Ngoc Toan is a new employee which is inserted a new row into a dimension. So, the data is transformed while moving from the source system into the data warehouse. As you see, handling SCDs is one of the most crucial tasks in the **ETL** processes.

Let's see what SCD 2 means when it comes to data modeling in Power BI. After we load the data into the data model in Power BI Desktop, we have all current and historical data in the dimension tables. Therefore, we have to be careful when deadling with SCDs. For instance, the following screeshot shows timecard hours for employees:

![](/projects/scd-data-warehouse/img3.jpg)

At the beginning, the numbers seem to be correct. Well, they may be right / wrong. It depends on what the business expects to see on a report. Look at **Lai, Tan Dat** employee. He had some values when he was in Services department (**610-Professional Services**). But after his changes (**650-Worldwide Support**), his data must shown in Support department. We did not consider SCD when we created the preceding table, which means we consider Lai's timecard hours value (**610-Professional Services**).

What if the business needs to only show timecard hours values for employees when their status is **Current**? In that case, we should have to set the SCD and add the **Status** column as a filter in the visualizations, while in other cases, we might need to modify the measures by adding the **Start Date**, **End Date**, and **Status** columns to filter the results. 

## Case Study 2
