---
title: "Adding NetSuite Saved Searches to Synapse"
description: "ETL Pipeline with Azure Synapse using Data Flows transformation"
dateString: January 04, 2023
draft: false
tags: ["Data Engineer", "Azure Synapse Analytics", "Data Lake", "Data Lakehouse", "NetSuite"]
weight: 201
cover:
    image: "/projects/azure-synapse-netsuite-api/cover.png"
---

The data lakehouse is a great destination for analytical data as it offers many different ways to access and query data in an efficient way. Setting up the pipelines, however, can be tedious. This guide walks though adding a NetSuite saved search to the lakehouse.

## Prerequisites
Before you get started, you'll need the following:
- The internal ID of a saved search in NetSuite (usually "customsearch#####").
- Contributor access to Azure Synapse.

## Creating the Data Flow
The first component to set up is the data flow. This is what actually extracts the data from NetSuite and stores it in Azure Storage for use later.
1. Go to https://web.azuresynapse.net and sign in.
2. Click on the **Develop** menu item on the left, then open the context menu for **NetSuiteSavedSearches** and select **New data flow**.

![](/projects/azure-synapse-netsuite-api/img1.png)

3. On the right side of the screen on the properties pane, provide a name and description for the data flow. I recommend using the NetSuite search ID as the name and include the name and URL of the search in the description.

![](/projects/azure-synapse-netsuite-api/img2.png)

4. Next, click **Add source** in the main builder panel. Under **Source settings**, select the **Inline** source type and set **Inline dataset type** to **REST**. The **Linked service** setting should be set to **rest_netsuite_saved_search**

![](/projects/azure-synapse-netsuite-api/img3.png)

5. On the **Source options** tab, set the **Relative URL** to the internal ID of the saved search you want to ingest.

![](/projects/azure-synapse-netsuite-api/img4.png)

6. On the same tab, scroll down to the **Pagination rules** and mirror the settings below. This ensures Synapse can request one page of search results at a time, preventing timeouts for large result sets.

![](/projects/azure-synapse-netsuite-api/img5.png)

7. Under **JSON settings**, select **Array of documents**.

![](/projects/azure-synapse-netsuite-api/img6.png)

8. Open the **Projection** tab and click **Import schema**. If this button is disabled, you'll need to turn on **Data flow debug** (the switch is above the data flow builder pane).

![](/projects/azure-synapse-netsuite-api/img7.png)

9. Once the schema has been imported (which can take some time to complete), click **Overwrite schema**, then click **Define complex type.**

![](/projects/azure-synapse-netsuite-api/img8.png)
![](/projects/azure-synapse-netsuite-api/img9.png)
![](/projects/azure-synapse-netsuite-api/img10.png)

10. In the **Expression** box, just add an open and close square bracket to the end of the existing expression. This just tells synapse to expect an array of records using this schema. Click **Save and finish** when done.

![](/projects/azure-synapse-netsuite-api/img11.png)

11.	Click the "+" to the right of the source data flow item and select the **Flatten** formatter. Under the **Flatten settings** tab, set **Unroll by** to **body**, then click **Reset** under **Input columns**.

![](/projects/azure-synapse-netsuite-api/img12.png)

12.	At this point, you can open the **Data preview** tab and click **Refresh** to confirm the data is being pulled from the saved search correctly. This preview can be performed on any step in the flow to see what the data looks like at that point in the pipeline.

![](/projects/azure-synapse-netsuite-api/img13.png)

13.	This step is only required if you have column name with spaces in them. Click the "+" to the right of the last step and select the **Select** schema modifier. On the **Select settings** tab under the **Name as** column, rename any column as needed to ensure no column names after this step will exist with spaces.

![](/projects/azure-synapse-netsuite-api/img14.png)

14.	This step is only required if you have dates in your result set. Click the "+" to the right of the last step and select the **Derived** Column schema modifier. Under **Derived column's settings**, click **Open expression builder**. Add a new column for each date column here, each with the same name as the existing column. We'll use the **toTimestamp** function to parse the NetSuite date/time value into a strongly-typed date/time for Synapse. NetSuite uses the "M/d/yyyy h:mm a" format for date/times and the "M/d/yyyy" format for dates. Click **Save and finish** when you're done.

![](/projects/azure-synapse-netsuite-api/img15.png)

15.	Once again, click the "+" to the right of the last step and select the **Sink** destination. Under the **Sink** tab, set **Sink type** to **Inline** and **Inline dataset type** to **Parquet**. The **Linked service** should be set to **adls_tcopsanalytics**.
16.	Under the **Settings** tab, set the folder path's **File system** to **netsuite-searches** and the **Folder path** to the ID of the saved search you're ingesting. The **Compression type** should be set to **Snappy**.
17.	For most searches, you'll want to check the **Clear the folder** box. This ensures there is no duplicated data.
18.	Preview your data once more on the sink step to verify the data is formatted properly and how you'd expect. Then, click **Commit all** in the top menu bar. Don't publish quite yet!

![](/projects/azure-synapse-netsuite-api/img16.png)

## Scheduling Ingestion
Data flows can't be scheduled directly, so we must use a pipeline to kick off a data flow and ingest our data. While we could create a new pipeline for each data flow and attach a trigger to each of those pipelines, we're limited to the number of concurrent connection to NetSuite. We want to ensure we're doing our best to minimize the number of concurrent requests to NetSuite to prevent slowing down access for others using the same API. To do that, we'll use a single pipeline that maintains a list of data flows to run and we only ever allow two data flows to run at one time.
1. Click on the **Integrate** menu item and find the **Nightly NetSuite Ingestion** pipeline.

![](/projects/azure-synapse-netsuite-api/img17.png)

2. Add your data flow as a step in the Nightly NetSuite ingestion. Please ensure your dataflow is added *after* an existing dataflow to ensure we're not running more than two dataflows at any given time.
3. Click **Commit all** once again, then click **Publish**.
4. At this point, your ingestion is active! There is one last step to expose the data as a SQL database.

## Exposing the Data in the Lakehouse
At this point, your data is being ingested and stored in the TC-OPS data lake. This data can be accessed by apps like Excel and Power BI. But sometimes, it is easier to access the data using a SQL Client or and app that can connect to a SQL database. We enable that by creating a lakehouse database table.
1. Before completing the rest of this guide, your data has to have been ingested at least once as we'll be importing the ingested data to generate a lakehouse table schema.
2. Click on the Data menu item and open the NetSuiteSearches database.

![](/projects/azure-synapse-netsuite-api/img18.png)

3. Open the **Table** dropdown and select **From data lake**.

![](/projects/azure-synapse-netsuite-api/img19.png)

4. Set the **External table name** to the saved search ID and set the **Linked service** to **adls_tcopsanalytics**. The **Input file or folder** setting should be set to **netsuite-searches/<search_id>**. Click **Continue** and wait for Synapse to import the schema.
5. Click **Create, Commit all**, and **Publish**.

## Resources
- [Data flows in Azure Synapse Analytics](https://learn.microsoft.com/en-us/azure/synapse-analytics/concepts-data-flow-overview)
- [Building the Lakehouse - Implementing a Data Lake Strategy with Azure Synapse](https://techcommunity.microsoft.com/t5/azure-synapse-analytics-blog/building-the-lakehouse-implementing-a-data-lake-strategy-with/ba-p/3612291)