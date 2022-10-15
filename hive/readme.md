
# Hive usage practice from Dataproc

Google recommends persisting the Hive data in Cloud Storage and the Hive metastore in MySQL on Cloud SQL.

Create a MySQL instance on Cloud SQL for the Hive metastore.
Deploy Hive servers on Dataproc.
Install the Cloud SQL Proxy on the Dataproc cluster instances.
Upload Hive data to Cloud Storage.
Run Hive queries on multiple Dataproc clusters.

#### Open google Shell in GCP
https://console.cloud.google.com/


```shell
export PROJECT=$(gcloud info --format='value(config.project)')
export REGION=us-central1
export ZONE=us-central1-a
#set the default Compute Engine zone to the zone where you are going to create your Dataproc clusters
gcloud config set compute/zone ${ZONE}
```


#### Enable the Dataproc and Cloud SQL Admin APIs by running this command in Cloud Shell:
```shell
gcloud services enable dataproc.googleapis.com sqladmin.googleapis.com
```

####  We create the data warehouse 

This is where our in and out data will reside, in a pure Haddop cluster we would use HDFS but here we are using Cloud Storage.

Also this shows how flexible Hive is.

```shell
export WAREHOUSE_BUCKET=my-hive-warehouse
gsutil mb -l ${REGION} gs://${WAREHOUSE_BUCKET}
```

####  Create a MySQL instance to store the Metastore
```shell
gcloud sql instances create hive-metastore \
    --database-version="MYSQL_5_7" \
    --activation-policy=ALWAYS \
    --zone ${ZONE}
```


####  Create Dataproc Cluster

```shell
gcloud dataproc clusters create hive-cluster \
    --scopes sql-admin \
    --region ${REGION} \
    --initialization-actions gs://goog-dataproc-initialization-actions-${REGION}/cloud-sql-proxy/cloud-sql-proxy.sh \
    --properties "hive:hive.metastore.warehouse.dir=gs://${WAREHOUSE_BUCKET}/datasets" \
    --metadata "hive-metastore-instance=${PROJECT}:${REGION}:hive-metastore" \
    --metadata "enable-cloud-sql-proxy-on-workers=false"
```

#### Copy Dataset into Bucket
Then we will use a Schema to read from it
```shell
#  Parquet format and contains thousands of fictitious bank transaction records with three columns: date, amount, and transaction type.
gsutil cp gs://hive-solution/part-00000.parquet \
gs://${WAREHOUSE_BUCKET}/datasets/transactions/part-00000.parquet
```


#### Create External Table for Parquet file

```shell
gcloud dataproc jobs submit hive \
    --cluster hive-cluster \
    --region ${REGION} \
    --execute "
      CREATE EXTERNAL TABLE transactions
      (SubmissionDate DATE, TransactionAmount DOUBLE, TransactionType STRING)
      STORED AS PARQUET
      LOCATION 'gs://${WAREHOUSE_BUCKET}/datasets/transactions';"
```

####  HiveQL query using gcloud Dataproc Jobs API

```shell
gcloud dataproc jobs submit hive \
    --cluster hive-cluster \
    --region ${REGION} \
    --execute "
      SELECT *
      FROM transactions
      LIMIT 10;"
```

####  HiveQL query using Beeline
```shell
# SSH session with the Dataproc's master instance
gcloud compute ssh hive-cluster-m

# Open Beeline session
beeline -u "jdbc:hive2://localhost:10000"

```
When beeline proxy appears:
```roomsql
SELECT TransactionType, AVG(TransactionAmount) AS AverageAmount
FROM transactions
WHERE SubmissionDate = '2017-12-22'
GROUP BY TransactionType;
```
Many people only uses this as their daily job!


#### Now let's query our table with Spark SQL

```shell
pyspark
```
```python
from pyspark.sql import HiveContext
hc = HiveContext(sc)
hc.sql("""
SELECT SubmissionDate, AVG(TransactionAmount) as AvgDebit
FROM transactions
WHERE TransactionType = 'debit'
GROUP BY SubmissionDate
HAVING SubmissionDate >= '2017-10-01' AND SubmissionDate < '2017-10-06'
ORDER BY SubmissionDate
""").show()
```

#### Inspect Hive Metastore

```shell
# start a new MySQL session on the Cloud SQL instance
gcloud sql connect hive-metastore --user=root
```
here info to protect passwords by encryption https://github.com/GoogleCloudDataproc/initialization-actions/tree/master/cloud-sql-proxy#protecting-passwords-with-kms

```
USE hive_metastore;
```

```roomsql
--Verify that the table is correctly referenced in the metastore
SELECT DB_LOCATION_URI FROM DBS;

--Verify that the table's columns are also correctly referenced:
SELECT TBL_NAME, TBL_TYPE FROM TBLS;

--Verify that the input format and location are also correctly referenced:
SELECT COLUMN_NAME, TYPE_NAME
FROM COLUMNS_V2 c, TBLS t
WHERE c.CD_ID = t.SD_ID AND t.TBL_NAME = 'transactions';
```

#### Creating another Dataproc cluster
```shell
gcloud dataproc clusters create other-hive-cluster \
    --scopes cloud-platform \
    --image-version 2.0 \
    --region ${REGION} \
    --initialization-actions gs://goog-dataproc-initialization-actions-${REGION}/cloud-sql-proxy/cloud-sql-proxy.sh \
    --properties "hive:hive.metastore.warehouse.dir=gs://${WAREHOUSE_BUCKET}/datasets" \
    --metadata "hive-metastore-instance=${PROJECT}:${REGION}:hive-metastore"\
    --metadata "enable-cloud-sql-proxy-on-workers=false"
```

```shell
gcloud dataproc jobs submit hive \
    --cluster other-hive-cluster \
    --region ${REGION} \
    --execute "
      SELECT TransactionType, COUNT(TransactionType) as Count
      FROM transactions
      WHERE SubmissionDate = '2017-08-22'
      GROUP BY TransactionType;"
```

#### Clean
```shell
gcloud dataproc clusters delete hive-cluster --region ${REGION} --quiet
gcloud dataproc clusters delete other-hive-cluster --region ${REGION} --quiet
gcloud sql instances delete hive-metastore --quiet
gsutil rm -r gs://${WAREHOUSE_BUCKET}/datasets
```
