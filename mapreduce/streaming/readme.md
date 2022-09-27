## Create Dataproc Cluster

We can do it from console and then SSH the Gateway

```bash
hadoop jar /usr/lib/hadoop-mapreduce/hadoop-streaming.jar \
    -files gs://data-604-hadoop/mapper.py,gs://data-604-hadoop/reducer.py \
    -mapper ./mapper.py \
    -reducer ./reducer.py \
    -input gs://data-604-hadoop/books/pg20417.txt \
    -output gs://data-604-hadoop/output
```



#### Try following commands in Dataproc Gateway

```bash
#upload a meaningless file into HDFS
echo foo > foo.txt
hdfs dfs -mkdir hdfs:///foo
hdfs dfs -put foo.txt hdfs:///foo/foo.txt

#Shell script that just tell the version of Python we are running
echo '#!/bin/bash' > info.sh
echo 'which python' >> info.sh
echo 'python --version 2>&1' >> info.sh

# MapReduce job using the streaming API that will just tell what version of Python are the workers using here
hadoop jar /usr/lib/hadoop-mapreduce/hadoop-streaming.jar \
     -Dmapred.reduce.tasks=0 \
     -Dmapred.map.tasks=1 \
     -files info.sh \
     -input hdfs:///foo \
     -output hdfs:///info-output \
     -mapper ./info.sh
```