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

```
spark.master	spark://student5-master:7077 
spark.serializer  org.apache.spark.serializer.KryoSerializer
spark.executor.instances 11 
spark.eventLog.enabled true
spark.eventLog.dir hdfs://student5-master:9000/tmp/sparkLog 
spark.history.fs.logDirectory hdfs://student5-master:9000/tmp/sparkLog spark.eventLog.logBlockUpdates.enabled=true
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

```shell
su syk
source /etc/profile
# Run program using YARN Client mode
spark-submit --class org.apache.spark.examples.SparkPi --master yarn --deploy-mode client /opt/spark-2.4.0-bin-hadoop2.7/examples/jars/spark-examples_2.11-2.4.0.jar 3
```

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-30%20at%2012.05.55%20AM.png)

