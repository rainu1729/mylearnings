**Google BigQuery: Basic Operations**

This guide covers simple operations in Google BigQuery: adding a dataset, creating a table, and uploading data.
**Some basics commands to run in gcloud commandline**
```bash
--list the active account name
gcloud auth list
--list the project ID
gcloud config list project
```

**Add dataset Using Console:**

You can add a dataset using the Google Cloud Console or the `bq` command-line tool.


1. Go to [BigQuery Console](https://console.cloud.google.com/bigquery).
2. Select your project.
3. Click **Create Dataset**.
4. Enter a Dataset ID and configure settings.
5. Click **Create Dataset**.

**Using `bq` CLI:**
```bash
bq mk --dataset project_id:dataset_name

--list the datasets if any available in the current project

bq ls

--list dataset available in specific projectid. Eg: bigquery-public-data

bq ls bigquery-public-data:

```

**Creating a Table**

**Using Console:**
1. Open your dataset.
2. Click **Create Table**.
3. Choose source (Empty table, CSV, JSON, etc.).
4. Define table schema.
5. Click **Create Table**.

**Using `bq` CLI:**
```bash
bq mk --table project_id:dataset_name.table_name name:STRING age:INTEGER

--list tables in dataset 
bq ls dataset_name

--removing dataset 
bq rm -r dataset_name

```

**Uploading Data**

**Using Console:**
1. Open your table.
2. Click **Upload**.
3. Select file (CSV, JSON, etc.).
4. Map columns and upload.

**Using `bq` CLI:**
```bash
bq load --source_format=CSV project_id:dataset_name.table_name ./data.csv name:STRING,age:INTEGER

--show table status like record count, storage bytes 

bq show dataset_name.table_name
```

**Reterive the data on public datasets bigquery Studio not commandline**
``` 
SELECT COUNT(*) as num_duplicate_rows, * FROM
`data-to-insights.ecommerce.all_sessions_raw`
GROUP BY
fullVisitorId, channelGrouping, time, country, city, totalTransactionRevenue, transactions, timeOnSite, pageviews, sessionQualityDim, date, visitId, type, productRefundAmount, productQuantity, productPrice, productRevenue, productSKU, v2ProductName, v2ProductCategory, productVariant, currencyCode, itemQuantity, itemRevenue, transactionRevenue, transactionId, pageTitle, searchKeyword, pagePathLevel1, eCommerceAction_type, eCommerceAction_step, eCommerceAction_option
HAVING num_duplicate_rows > 1;

```
**Running query on public datasets**

```bash 
--examine the schema of the Shakespeare table in the samples dataset
bq show bigquery-public-data:samples.shakespeare
```

**References**

- [BigQuery Documentation](https://cloud.google.com/bigquery/docs)
- [bq Command-Line Tool](https://cloud.google.com/bigquery/docs/bq-command-line-tool)
