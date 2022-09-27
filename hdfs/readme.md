# HDFS

First we need an HDFS we have accesses to communicate with.

Let's go to the fastest way and let's procure one with GC


# block size of your computer
Physical Block size is the smallest amount of data that can be written at one time to a disk. Recent versions of Macs use 4096 bytes block size (4k sector size), while the default sector size on earlier Macs is 512 bytes.
diskutil info / | grep "Block Size"

# block size of Hadoop
HDFS is given a block size of 128 MB by default in Hadoop 2.x (64 MB in Hadoop 1.x)
But Hadoop is an abstraction, not an actual file system, under the covers metal hardware is giving support to Hadoop with 4kB size blocks
But here almost all is an abstraction
Abstractions are unefficient but makes our life easier
And this is when one can realize that organization beats efficiency

But, the underlying system must be efficient to allow our inefficiency to be affordable.

Here this is not a problem as underlyinh machines run very efficient Linux OS written in C close to the metal



https://console.cloud.google.com/flows/enableapi?apiid=bigtable,bigtableadmin.googleapis.com&_ga=2.256366539.-1994314811.1556263678

gcloud bigtable instances create INSTANCE_ID \
--cluster=CLUSTER_ID \
--cluster-zone=CLUSTER_ZONE \
--display-name=DISPLAY_NAME \
[--cluster-num-nodes=CLUSTER_NUM_NODES] \
[--cluster-storage-type=CLUSTER_STORAGE_TYPE] \
[--instance-type=INSTANCE_TYPE]

For instance:
gcloud bigtable instances create INSTANCE_ID \
--cluster=ufv-demo-cluster \
--cluster-zone=europe-southwest1-a \
--display-name=hadoop \
--cluster-num-nodes=4 \
--cluster-storage-type=HDD \




Cloud Bigtable, Cloud Bigtable Admin, Cloud Dataproc, and Cloud Storage JSON APIs.
https://console.cloud.google.com/flows/enableapi?apiid=bigtable.googleapis.com,bigtableadmin.googleapis.com,dataproc.googleapis.com,storage-api.googleapis.com&_ga=2.51696745.-1994314811.1556263678
