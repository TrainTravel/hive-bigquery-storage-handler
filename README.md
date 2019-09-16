# Hive-BigQuery StorageHandler

This is a Hive StorageHandler plugin that enables Hive to interact with BigQuery. It allows you keep
your existing pipelines but move to BigQuery. It utilizes the high throughput 
[BigQuery Storage API](https://cloud.google.com/bigquery/docs/reference/storage/) to read data and
 uses the BigQuery API to write data.
 
 The following steps are performed under Dataproc cluster in Google Cloud Platform. If you need to 
 run in your cluster, you will need setup Google Cloud SDK and Google Cloud Storage connector for 
 Hadoop.

## Getting the StorageHandler

1. Check it out from GitHub.
2. Build it with the new Google [Hadoop BigQuery Connector](https://cloud.google.com/dataproc/docs/concepts/connectors/bigquery) 
``` shell
git clone https://github.com/GoogleCloudPlatform/hive-bigquery-storage-handler
cd hive-bigquery-storage-handler
mvn clean install
```
3. Deploy hive-bigquery-storage-handler-1.0-shaded.jar 

## Using the StorageHandler to access BigQuery
After placing the compiled Jar to a Google Cloud Storage bucket that your Hive cluster have access
to, you can open Hive CLI and load all the necessary Jars:
```shell
beeline> add jar gs://<Jar location>/hive-bigquery-storage-handler-1.0-shaded.jar;
```
At this point you can operate Hive just like you used to do.

### Creating BigQuery tables
If you have BigQuery table already, here is how you can define Hive table that refer to it:
```sql
CREATE TABLE bq_test (word_count bigint, word string)
  STORED BY
    'com.google.cloud.hadoop.io.bigquery.hive.HiveBigQueryStorageHandler'
  TBLPROPERTIES (
  'bq.dataset'='<BigQuery dataset name>',
  'bq.table'='<BigQuery table name>',
  'mapred.bq.project.id'='<Your Project ID>',
  'mapred.bq.temp.gcs.path'='gs://<Bucket name>/<Temporary path>',
  'mapred.bq.gcs.bucket'='<Cloud Storage Bucket name>'
  );
```
You will need to provide the following table properties:

| Property | Value |
|--------|-----|
| bq.dataset | BigQuery dataset id |
| bq.table | BigQuery table name |
| mapred.bq.project.id | Your project id |
| mapred.temp.gcs.path | Temporary file location in GCS bucket |
| mapred.bq.gcs.bucket | Temporary GCS bucket name |

### Issues
1. Writing to BigQuery will fail when using Apache Tez as the execution engine. You can use ```set hive.execution.engine=mr```
to ask Hive use MapReduce as the execution engine to workaround it.

