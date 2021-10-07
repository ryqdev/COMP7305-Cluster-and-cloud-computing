---
layout:     post
title:      Spark installation
date:       2021-09-24
author:     Yukun SHANG
catalog: 	true
tags:       [Basic]
---
# Spark installation

# 1. Spark installation

## 1.1 Download Spark

on the` master node`

```shell
cd /opt
sudo scp student@202.45.128.135:~/comp7305/spark-2.4.0-bin-hadoop2.7.tgz .
sudo tar xvf spark-2.4.0-bin-hadoop2.7.tgz
sudo chown -R hduser:hadoop ./spark-2.4.0-bin-hadoop2.7
```

## 1.2 Configure Spark

on the` master node`, use hduser account

```shell
cp /opt/spark-2.4.0-bin-hadoop2.7/conf/spark-env.sh.template /opt/spark-2.4.0-bin-hadoop2.7/conf/spark-env.sh 
sudo vim /opt/spark-2.4.0-bin-hadoop2.7/conf/spark-env.sh
```

Add

```shell
HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop LD_LIBRARY_PATH=/opt/hadoop-2.7.5/lib/native:$LD_LIBRARY_PATH
```



```shell
cp /opt/spark-2.4.0-bin-hadoop2.7/conf/spark-defaults.conf.template /opt/spark-2.4.0-bin-hadoop2.7/conf/spark-defaults.conf
sudo vim /opt/spark-2.4.0-bin-hadoop2.7/conf/spark-defaults.conf
```

Add

```shell
spark.master spark://student5-master:7077 
spark.serializer org.apache.spark.serializer.KryoSerializer
spark.executor.instances 11 
spark.eventLog.enabled true
spark.eventLog.dir hdfs://student5-master:9000/tmp/sparkLog 
spark.history.fs.logDirectory hdfs://student5-master:9000/tmp/sparkLog 
spark.eventLog.logBlockUpdates.enabled=true
```



```shell
hdfs dfs -mkdir /tmp/sparkLog 
hdfs dfs -chmod -R 777 /tmp/sparkLog
```

### 1.3 Setting up environment

on the` master node`

```shell
sudo vim /etc/profile
```

add:

```
export SPARK_HOME=/opt/spark-2.4.0-bin-hadoop2.7 
export PATH=$PATH:$SPARK_HOME/bin 
export PATH=$PATH:$SPARK_HOME/sbin
```



```shell
source /etc/profile
```

### 1.4 Zip and Copy Spark to all other containers

on the` master node`

```shell
cd /opt
tar cvf ~/spark-7305.tgz spark-2.4.0-bin-hadoop2.7
```



on the` slave node`

```shell
sudo scp hduser@student5-master:spark-7305.tgz /opt
cd /opt 
sudo tar xvf spark-7305.tgz
sudo chown -R hduser:hadoop /opt/spark-2.4.0-bin-hadoop2.7
```

### 1.5 Run Spark

use syk account

#### 1.5.1 Run SparkPi using ”spark-submit”

```shell
su syk
source /etc/profile
# Run program using YARN Client mode
spark-submit --class org.apache.spark.examples.SparkPi --master yarn --deploy-mode client /opt/spark-2.4.0-bin-hadoop2.7/examples/jars/spark-examples_2.11-2.4.0.jar 3
```

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-30%20at%2012.05.55%20AM.png)

```shell
# spart exaple 
spark-submit --class org.apache.spark.examples.SparkPi --master yarn --deploy-mode cluster /opt/spark-2.4.0-bin-hadoop2.7/examples/jars/spark-examples_2.11-2.4.0.jar 3
```

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-30%20at%203.41.38%20PM.png)



#### 1.5.2 Running Spark via spark-shell

```shell
spark-shell --master yarn
```

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-30%20at%203.43.30%20PM.png)



### 1.6 Save container to docker image

on physical machine

#### 1.6.1 commit

```shell
docker commit --author "YourName" Container_ID 10.42.0.102:5000/groupXX-YourName:v1

# e.g.
docker commit --author "Yukun Shang" 917cb0739437 10.42.0.102:5000/group04-syk:v1
```

#### 1.6.2 push

```shell
docker push 10.42.0.102:5000/groupXX-YourName:v1

# e.g.
docker push 10.42.0.102:5000/group04-syk:v1
```

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-30%20at%208.36.52%20PM.png)

`docker push` -> Push an image or a repository to a **Docker Hub** or **self-hosted registry**.

In the class, we maintain a private docker registry. The address is 10.42.0.102:5000.

## 2. Spark Web Interfaces

### 2.1 start history server

on master node, use hduser account

```shell
start-history-server.sh
```

### 2.2 set up SSH tunneling on studentXX and COC-Server

On the physical machine

```shell
ssh -Nf -L 10.42.0.15:10505:10.244.103.23:18080 hduser@10.244.103.23
```

On the COC-Server (the group machine)

```shell
ssh -Nf -L 202.45.128.135:10505:10.42.0.15:10505 student@10.42.0.15
```

Then access `202.45.128.135:10505` to the Spark UI

<img src="https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-30%20at%204.36.22%20PM.png" style="zoom:50%;" />



## 3. Useful Spark operations and examples

### 3.1 WordCount Example

#### 3.1.1 preparation

on the master node, use syk account

```shell
# Download data file “books.tar.gz” from COC-Server
scp student@10.42.0.1:~/comp7305/books.tar.gz ./
tar zxvf books.tar.gz
# Finally, copy the whole folder to HDFS
hdfs dfs -copyFromLocal books /user/syk/books

# list the files 
hdfs dfs -ls /user/syk/books

# Show the block location datanodes
hdfs fsck /user/syk/books -files -blocks -locations
```

#### 3.1.2 Run WordCount with Spark Shell

```shell
spark-shell --master yarn
# Read all files in /books
var textfile = sc.textFile("hdfs:///user/syk/books");
# Transformation: filter lines that are non-empty
var lines = textfile.filter(line => line.length>0);
# Action: count number of non-empty lines
var count = lines.count();
```

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-30%20at%204.49.32%20PM.png)

### 3.2 Spark: log levels

```shell
sc.setLogLevel("INFO")
```



## 4. Building Spark applications

#### 4.1 Install maven

use hduser account

```shell
sudo apt-get update
sudo apt-get install maven
```

#### 4.2 Check Scala version used in Spark

```shell
spark-shell --master=yarn
```

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-30%20at%205.06.21%20PM.png)

#### 4.3 Folder structure

use syk account

```shell
scp student@10.42.0.1:~/comp7305/spark-example.tar.gz ./

tar zxvf spark-example.tar.gz
```



#### 4.4 SimpleApp.scala

```shell
scp -r student@10.42.0.1:~/comp7305/sim/ ./
cd sim
cat src/main/scala/SimpleApp.scala
```

Build program

```shell
cd sim 
mvn package
```

Since it will take about 20 minutes, this part skipped.



#### 4.5 PySpark

use hduser

#### 4.5.1 preparation

Install the following on 12 containers

```shell
sudo apt-get update
sudo apt-get install python
sudo apt install python-pip

sudo pip install numpy
sudo pip install pandas
sudo pip install pandas_datareader
```

#### 4.5.2 launch pyspark

On master node, use hduser

* use Python shell

```shell
pyspark --master yarn
```

* Spark-submit

```shell
spark-submit --master yarn /opt/spark-2.4.0-binhadoop2.7/examples/src/main/python/pi.py
```



## 5. Stock Recommendation Project Demo on Spark

skipped



## 6. Spark TeraSort

Ta has given his configurations, we need to find the better parameters to beat him.



## 7. Run Logistic Regressing (lr.scala, with Iteration = 100)

### 7.0 preparetion

On master node, use syk account

```shell
cd ~ 
scp student@10.42.0.1:~/comp7305/lr.scala ./
```

Execute in Spark Shell

```shell
scala> sc.setLogLevel("INFO") 
scala> :load /home/syk/lr.scala
```

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-10-07%20at%201.57.39%20PM.png)

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-10-07%20at%202.02.05%20PM.png)

### 7.1 Change iteration number from 5 to 100

on the master node, use syk account

```shell
cd ~
vim lr.scala
```

Modify this :

```
val ITERATIONS = 100
```

### 7.2 Execute in spark shell

```
scala> :load /home//lr.scala
```

### 7.3 During running, kill an executor.

ssh to one of the slave nodes (student5-x1)

```shell

jps # to find the pid of CoarseGrainedExecutorBackend

kill -9 [pid]
```

<img src="https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-10-07%20at%202.15.10%20PM.png" style="zoom:50%;" />

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-10-07%20at%202.10.04%20PM.png)

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-10-07%20at%202.13.31%20PM.png)

A new executor “12” is automatically added



















## 8.TF-IDF

### 8.0 Preparation

Maybe on master node, use syk account

```shell
scp student@10.42.0.1:~/comp7305/td-idf.scala ./
scp -r student@10.42.0.1:~/large_stories ./
```



Use command

```shell
ls large_stories/ | wc -l
```

to show how many files are stored in the large_stories directory

use

```shell
du -sh large_stories/
```

to. check the total size

<img src="https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-10-07%20at%202.31.07%20PM.png" style="zoom:50%;" />

Upload the data to your HDFS

```shell
hdfs dfs -copyFromLocal large_stories /user/syk/
```

Edit `td-idf.scala` to change the input file name/path:

```shell
vim ~/tf-idf.scala
```

Modify to 

```shell
sc.wholeTextFiles("hdfs:///user/syk/large_stories")
```

### 8.1 Execute td-idf in Spark Shell on the master node 

* Default configuration:

```shell
scala> :load /home/syk/td-idf.scala
```

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-10-07%20at%203.01.42%20PM.png)



![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-10-07%20at%203.04.51%20PM.png)

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-10-07%20at%203.05.38%20PM.png)





Q: As Spark Shell is running in client mode, the Driver code is not running in any Yarn controlled containers, but an Application Master is still needed. Can you tell which node (K8S container) was used to run the Application Master for serving your td-idf application? Show a screenshot as evidence.

A: 







* spark.executor.memory=2G, dfs.blocksize=64m, dfs.replication=2

```shell
sudo vim /opt/spark-2.4.0-bin-hadoop2.7/conf/spark-defaults.conf
```

add

```
spark.executor.memory 2G
dfs.blocksize 64m
dfs.replication 2
```

<img src="https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-10-07%20at%203.15.59%20PM.png" style="zoom:50%;" />

```shell
scala> :load /home/syk/td-idf.scala
```

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-10-07%20at%203.18.35%20PM.png)

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-10-07%20at%203.19.08%20PM.png)



### 8.2 Improve the execution time of td-idf.scala

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-10-07%20at%203.45.01%20PM.png)

increase the executor memory

```shell
spark.master spark://student5-master:7077
spark.serializer org.apache.spark.serializer.KryoSerializer
spark.executor.instances 11
spark.eventLog.enabled true
spark.eventLog.dir hdfs://student5-master:9000/tmp/sparkLog
spark.history.fs.logDirectory hdfs://student5-master:9000/tmp/sparkLog
spark.eventLog.logBlockUpdates.enabled=true
spark.executor.memory 4G   # modify here
dfs.blocksize 64m
dfs.replication 2
```

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-10-07%20at%203.52.03%20PM.png)





* spark.executor.memory 3G

```shell
spark.master spark://student5-master:7077
spark.serializer org.apache.spark.serializer.KryoSerializer
spark.executor.instances 11
spark.eventLog.enabled true
spark.eventLog.dir hdfs://student5-master:9000/tmp/sparkLog
spark.history.fs.logDirectory hdfs://student5-master:9000/tmp/sparkLog
spark.eventLog.logBlockUpdates.enabled=true
spark.executor.memory 3G   # modify here
dfs.blocksize 64m
dfs.replication 2
```

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-10-07%20at%204.27.00%20PM.png)





* repatition:

```shell
vim ~/td-idf.scala
```

add repartition()

```shell
sc.wholeTextFiles("hdfs:///user/syk/large_stories").repartition(11).cache()
```



```
spark.master spark://student5-master:7077
spark.serializer org.apache.spark.serializer.KryoSerializer
spark.executor.instances 11
spark.eventLog.enabled true
spark.eventLog.dir hdfs://student5-master:9000/tmp/sparkLog
spark.history.fs.logDirectory hdfs://student5-master:9000/tmp/sparkLog
spark.eventLog.logBlockUpdates.enabled=true
spark.executor.memory 2G
dfs.blocksize 64m
dfs.replication 2
```

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-10-07%20at%204.48.15%20PM.png)



* repatition, with 3G mem:

```
spark.master spark://student5-master:7077
spark.serializer org.apache.spark.serializer.KryoSerializer
spark.executor.instances 11
spark.eventLog.enabled true
spark.eventLog.dir hdfs://student5-master:9000/tmp/sparkLog
spark.history.fs.logDirectory hdfs://student5-master:9000/tmp/sparkLog
spark.eventLog.logBlockUpdates.enabled=true
spark.executor.memory 3G
dfs.blocksize 64m
dfs.replication 2
```

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-10-07%20at%204.48.59%20PM.png)





