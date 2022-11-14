# Hadoop hands on

Hadoop is the most known and used Distributed File System at the moment

While times are everchanging and big players such as Google Cloud or AWS impose their own distributed storage solutions this open source is still used by many companies and still the king of Big Data at 2022

Let's face some hands-on trainings: 

1. Hadoop basic use of the filesystem
2. Use of MapReduce basic
3. Used if MapReduce with a language different from Java, Python using the Streaming Interface
4. Use of Hive as an abstraction of MapReduce giving the impression that we have a database using SQL for non programmers
5. Use of Pig to to a stream of actions over MapReduce without even knowing, I prefer Pig while everybody uses Hive, a pity.

I was thinking about doing some exercise with HBASE or Spark but while both are born in the echosystem of Hadoop, HBASE just happens to be born using Hadoop Filesystem but could be intedependent and Spark is totally independent, indeed the last versions of Spark tend to ignore Hadoop and instead of YARN Spark tends to be deployed in Kubernetes, meaning in any cluster.


