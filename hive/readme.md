


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

