
# HDFS basic usage

HDFS is the Hadoop Filesystem distributed among many nodes


Let's deploy a YARN cluster in the most convenient way that is to create a Dataproc cluster.

#### Create GCP project

For convenience we can use CG Shell https://console.cloud.google.com/

```shell
PROJECT_ID="ufv-hdfs-playaround-javi"
gcloud projects create $PROJECT_ID
gcloud projects list --sort-by=projectId --limit=5
gcloud config set project $PROJECT_ID
```
##### Enable billing
```shell
gcloud alpha billing accounts list
BILLING_ACCOUNT=`gcloud alpha billing accounts list | grep ACCOUNT | awk -F ':' '{print $2}'`
# check awk worked fine or correct
echo $BILLING_ACCOUNT
gcloud alpha billing projects link $PROJECT_ID --billing-account $BILLING_ACCOUNT
```

##### Choose a region
```shell
# Choose one of the zones for the project
gcloud compute zones list --format="value(selfLink.scope())"
REGION=us-central1
```

##### Enable APIs
In order to use Dataproc we need to enable dataproc.googleapis.com API

```shell
gcloud services enable dataproc.googleapis.com
```

### Create a Dataproc cluster
```shell
gcloud dataproc clusters create $PROJECT_ID --region=$REGION
# This will take minutes to procure
gcloud dataproc clusters list --region=$REGION
# We can see engines have been created associated to the cluster
gcloud compute instances list --format="json"
```
Let's see in the Web Console in Compute Engine section that now we have 3 Virtual Machines up and running.

In the Dataproc section we can see that these machines correspond to a Dataproc Cluster.
https://console.cloud.google.com/dataproc/clusters


##### See YARN UI

* YARN ResourceManager http://master-host-name:8088
* HDFS NameNode	http://master-host-name:9870

We see it is not working, the last versions makes it more difficult to get this monitoring information, security reasons probably, let's fix this.


#### Assign roles to a AIM user

Here you can see all IAM roles for Dataproc:

```shell
gcloud iam roles list | grep "name:" | grep "dataproc"
```

```shell
gcloud iam roles describe roles/dataproc.admin
```

how do you assign a role (and therefore all the associated permissions), to a user account?

There are two ways to attach a role:

* To the user and an organization
* To a user and a project, we will do this

With the following we can see the roles associated with the project:
```shell
gcloud projects get-iam-policy $PROJECT_ID
USERID=`gcloud projects get-iam-policy $PROJECT_ID | grep "user" | awk -F ':' '{print $2}'`
```

With this command we give the user the role of Dataproc Admin
```shell
gcloud projects add-iam-policy-binding $PROJECT_ID \
--member user:$USERID \
--role=roles/dataproc.admin
```

Decommission the cluster to then create it again with UIs opened
```shell
gcloud dataproc clusters delete $PROJECT_ID --region $REGION
gcloud dataproc clusters list --region=$REGION
```

```shell
gcloud dataproc clusters delete $PROJECT_ID --region $REGION
gcloud dataproc clusters list --region=$REGION

gcloud dataproc clusters create $PROJECT_ID --region=$REGION --enable-component-gateway
gcloud dataproc clusters list --region=$REGION

```
In the new cluster created let's see the Interfaces created:
```shell
gcloud dataproc clusters describe $PROJECT_ID --region $REGION
pip3 install yq
gcloud dataproc clusters describe $PROJECT_ID --region $REGION | yq
```

You can see here the UI for main apps:
```yaml
 "HDFS NameNode": "https://wyjua2wvt5amzolbo2e7jb5mye-dot-us-central1.dataproc.googleusercontent.com/hdfs/dfshealth.html",
 "MapReduce Job History": "https://wyjua2wvt5amzolbo2e7jb5mye-dot-us-central1.dataproc.googleusercontent.com/jobhistory/",
 "Spark History Server": "https://wyjua2wvt5amzolbo2e7jb5mye-dot-us-central1.dataproc.googleusercontent.com/sparkhistory/",
 "Tez": "https://wyjua2wvt5amzolbo2e7jb5mye-dot-us-central1.dataproc.googleusercontent.com/apphistory/tez-ui/",
 "YARN Application Timeline": "https://wyjua2wvt5amzolbo2e7jb5mye-dot-us-central1.dataproc.googleusercontent.com/apphistory/",
 "YARN ResourceManager": "https://wyjua2wvt5amzolbo2e7jb5mye-dot-us-central1.dataproc.googleusercontent.com/yarn/"
```
Open HDFS Namenode UI in any browser.

Explore the nodes composing the HDFS cluster, have a look to the available HDFS space.

Explore the Datanodes, have a look to the space they have each in the volume (aka disk)

Examine the logs, look for some familiar information within, whenever there is an error this would be the first place to look at.

You can see there are dozens of Java threads working concurrently in a single Datanode, each probably with a clear mission all acting together, you can have a feel of the complexity happening under the covers for something as apparently simple as serving files.

Just think that every three seconds this machine is sending a heartbeat to show it is alive, the cluster is continuously self monitored to guarantee availability.

### Travel the filesystem
In the Namenode UI go to Utilities, Browse the File System

You can see some familiarity with any OS Command Line you have used in the past

Now let's upload a file from the GC Shell, first SSH to the Main node of the Dataproc Cluster
```shell

hdfs dfs -ls /
echo "First Line" >> text.txt
echo "Second Line" >> text.txt
echo "Thirs Line" >> text.txt
# create a file
hdfs dfs -mkdir /tmp/practica
hdfs dfs -put text.txt /tmp/practica 
# also you could hdfs dfs -copyFromLocal text.txt /tmp/practica
hdfs dfs -ls /tmp/practica
hdfs dfs -cat /tmp/practica/text.txt
```

voila

You can also check in the UI the file is now there!

We can now delete the file

```shell
mkdir ne
hadoop fs -get /tmp/practica/text.txt ./ne
ls ne
rm ne/text.txt
hdfs dfs -get /tmp/practica/text.txt ./ne
ls ne
hdfs dfs -rm /tmp/practica/text.txt 
hdfs dfs -ls /tmp/practica
```

empty!

Now we can decomission the cluster and the project to avoid any future inconvenience



### Decommissioning
##### Decommission Cluster
```shell
gcloud dataproc clusters delete $PROJECT_ID --region $REGION
# Let's check the cluster is no longer there
gcloud dataproc clusters list --region=$REGION
```
##### Decommission Project
```shell
gcloud projects list --format="table(projectId,parent.id.yesno(yes="YES", no=”NO”):label='Has Parent':sort=2)"
gcloud projects delete $PROJECT_ID
gcloud projects list --format="table(projectId,parent.id.yesno(yes="YES", no=”NO”):label='Has Parent':sort=2)"
```


