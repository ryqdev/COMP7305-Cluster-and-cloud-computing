---
layout:     post
title:      Hadoop installation
date:       2021-09-24
author:     Yukun SHANG
catalog: 	true
tags:       [Basic]
---
# Hadoop installation

This tutorial aims to install Hadoop in cluster.

## 1 Setup Containers

### 1.1 Check the Ip address of machines(or containers)

```
10.244.103.23 student5-master
10.244.103.24 student5-x1
10.244.103.25 student5-x2

10.244.131.24 student34-master
10.244.131.25 student34-x1
10.244.131.23 student34-x2

10.244.132.16 student35-master
10.244.132.17 student35-x1
10.244.132.15 student35-x2

10.244.133.27 student36-master
10.244.133.26 student36-x1
10.244.133.28 student36-x2
```

### 1.2 Modity /etc/hosts on each machine

On `all` containers:

```shell
sudo vim /etc/hosts
```

In the hosts file, **remove the last line**, and add the ip infomation at step1

```
...
...


10.244.103.23 student5-master
10.244.103.24 student5-x1
10.244.103.25 student5-x2

10.244.131.24 student34-master
10.244.131.25 student34-x1
10.244.131.23 student34-x2

10.244.132.16 student35-master
10.244.132.17 student35-x1
10.244.132.15 student35-x2

10.244.133.27 student36-master
10.244.133.26 student36-x1
10.244.133.28 student36-x2
```



### 1.3 Disable IPv6 on each machine

Since Hadoop cannot work on IPv6, we should disable IPv6 on` all containers`.

```shell
sudo vim /etc/sysctl.d/99-sysctl.conf
```

In 99-sysctl.conf file, add these three lines:

```
net.ipv6.conf.all.disable_ipv6 = 1 
net.ipv6.conf.default.disable_ipv6 = 1 
net.ipv6.conf.lo.disable_ipv6 = 1
```

Activate setting:

```shell
sudo sysctl –p
```

And then reboot. (Attention!)



### 1.4 Install Utilities

```shell
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install vim
```

### 1.5 Install Java 8

```shell
sudo apt install openjdk-8-jdk -y
```

Check it

```shell
java -version
javac -version
```

Setup Java environment

```shell
sudo vim /etc/profile
```

In `profile` file , add :

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 
export JRE_HOME=$JAVA_HOME/jre 
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib 
export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
```

```shell
source /etc/profile
```

### 1.6 Create a Hadoop user `hduser`, and user group `hadoop`

```shell
sudo addgroup hadoop
```

and then 

```shell
sudo adduser --ingroup hadoop hduser
```

Enter the new Unix password: 123456.

Set others default.

Give “hduser” sudo right

```shell
sudo usermod -a -G sudo hduser
```



## 2 Setup Connection from Master to Slave Nodes

Finish this on `master node`

login hduser:

```
ssh hduser@10.244.103.23
```



```shell
ssh-keygen -t rsa -P ""
```

Add the key to other containers:(repeat this part several times)

```shell
ssh-copy-id hduser@studentXX-x1
```

Test SSH connection

```shell
ssh studentXX-master 
ssh studentXX-x1 
ssh studentXX-x2
```



## 3 Configure Hadoop in the Master Node.

Finish this on `master node`

### 3.1 Download Hadoop

```shell
cd /opt
# Download from COC-Server
sudo scp student@10.42.0.1:~/comp7305/hadoop-2.7.5.tar.gz ./
sudo tar zxvf hadoop-2.7.5.tar.gz
# Change the owner of the extracted files
sudo chown -R hduser:hadoop hadoop-2.7.5
```

### 3.1 Setup Hadoop Environment

```shell
vim /opt/hadoop-2.7.5/etc/hadoop/hadoop-env.sh
```

remove 

```
export JAVA_HOME=${JAVA_HOME}
```

Add 

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 
export HADOOP_HOME=/opt/hadoop-2.7.5 
export HADOOP_CONF_DIR=/opt/hadoop-2.7.5/etc/hadoop
```

### 3.2 Configure the Hadoop

* Modify core-site.xml

```shell
vim /opt/hadoop-2.7.5/etc/hadoop/core-site.xml
```

remove

```
<configuration> 
</configuration>
```

add

```
<configuration>
	<property>
		<name>fs.defaultFS</name>
			<value>hdfs://student5-master:9000</value>
	</property>
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/var/hadoop/hadoop-${user.name}</value>
	</property>
 </configuration>
```

where `student5-master` is master node

* Modify hdfs-site.xml

```shell
vim /opt/hadoop-2.7.5/etc/hadoop/hdfs-site.xml
```

remove

```
<configuration>
</configuration>
```

add

```
<configuration> 
	<property> 
		<name>dfs.replication</name> 
		<value>2</value> 
	</property> 
	<property> 
		<name>dfs.blocksize</name> 
		<value>64m</value>
	</property> 
	<property> 
		<name>dfs.datanode.du.reserved</name> 
		<value>193273528320</value> 
	</property> 
</configuration>
```

* Modify mapred-site.xml

```shell
sudo cp /opt/hadoop-2.7.5/etc/hadoop/mapred-site.xml.template /opt/hadoop-2.7.5/etc/hadoop/mapred-site.xml
vim /opt/hadoop-2.7.5/etc/hadoop/mapred-site.xml
```

remove

```
<configuration> 
</configuration>
```

add

```
<configuration> 
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
	<property>
		<name>mapreduce.map.memory.mb</name>
		<value>200</value>
	</property>
	<property>
		<name>mapreduce.reduce.memory.mb</name>
		<value>300</value>
   </property>
</configuration>
```

* Modify yarn-site.xml

```shell
vim /opt/hadoop-2.7.5/etc/hadoop/yarn-site.xml
```

remove

```
<configuration> 
</configuration>
```

add

```
<configuration> 
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>student5-master</value>
  </property> 
  <property>
  	<name>yarn.nodemanager.aux-services</name>
  	<value>mapreduce_shuffle</value>
  </property> 
  <property> 
  	<name>yarn.nodemanager.vmem-pmem-ratio</name> 
  	<value>4</value> 
  </property> 
 </configuration>
```



## 4. Install Hadoop

### 4.1 configure hadoop

In master node:

```shell
vim /opt/hadoop-2.7.5/etc/hadoop/masters
```

add

```
student5-master
```



```shell
vim /opt/hadoop-2.7.5/etc/hadoop/slaves
```

add

```
student5-x1
student5-x2

student34-master
student34-x1
student34-x2

student35-master
student35-x1
student35-x2

student36-master 
student36-x1
student36-x2
```

### 4.2 Zip the Hadoop folder

```shell
cd /opt
tar cvf ~/hadoop-7305.tar.gz hadoop-2.7.5
```

### 4.3 Copy hadoop-7305.tar.gz to all slave nodes

(Attention!) on ohter containers (**except the master node**)

```
sudo scp hduser@student5-master:~/hadoop-7305.tar.gz /opt
cd /opt
sudo tar xvf hadoop-7305.tar.gz
sudo chown -R hduser:hadoop /opt/hadoop-2.7.5
```

### 4.4 Setting up environment on slave nodes

```shell
sudo vim /etc/profile
```

At the end of profile, add:

```
export HADOOP_HOME=/opt/hadoop-2.7.5 
export CLASSPATH=$CLASSPATH:$HADOOP_HOME/lib 
export PATH=$PATH:$HADOOP_HOME/bin 
export PATH=$PATH:$HADOOP_HOME/sbin
```



```shell
sudo rm -rf /var/hadoop/*
sudo chown -R hduser:hadoop /var/hadoop
```



### 4.5 Setting up environment for the master node

```shell
sudo vim /etc/profile
```

add at the end

```
export HADOOP_HOME=/opt/hadoop-2.7.5 
export CLASSPATH=$CLASSPATH:$HADOOP_HOME/lib 
export PATH=$PATH:$HADOOP_HOME/bin 
export PATH=$PATH:$HADOOP_HOME/sbin
```



```
source /etc/profile
sudo rm -rf /var/hadoop/*
sudo chown -R hduser:hadoop /var/hadoop
```

## 5 Start Hadoop.

### 5.1 start Hadoop

on master node:

```shell
# Format the NameNode
hdfs namenode -format
# Starting up daemons
start-dfs.sh
start-yarn.sh
mr-jobhistory-daemon.sh start historyserver

# check it , Monitoring Hadoop
jps

# check HDFS
hdfs dfsadmin -report
```

### 5.2  SSH Tunneling

login physical machine (master)

```shell
ssh -Nf -L physicalMachineIP:XXXYY:containerIP:50070 hduser@containerIP

# e.g.
ssh -Nf -L 10.42.0.15:10005:10.244.103.23:50070 hduser@10.244.103.23
```



SSH Tunneling at **COC-server (groupXX@202.45.128.135)**

```shell
ssh -Nf -L 202.45.128.135:XXXYY: physicalMachineIP : XXXYY student@ physicalMachineIP

#e.g.
ssh -Nf -L 202.45.128.135:10005:10.42.0.15:10005 student@10.42.0.15
```



... (many steps omitted)

After setting up ssh tunnelling, we can access `202.45.128.135:10005` to get the web UI of Hadoop

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-29%20at%208.11.17%20PM.png)



## 6 Example Hadoop Program

### 6.1 Wordcount program

on the main node, login hduser

#### 6.1.1 Preparation 

```shell
cd ~
mkdir dft
# download file
scp student@10.42.0.1:~/comp7305/dft/large_input/word_input.txt dft/

# Create a directory in HDFS
hdfs dfs -mkdir /dft-group

# Upload this file to Hadoop
hdfs dfs -copyFromLocal /home/hduser/dft/word_input.txt /dft-group

# Check the file “word_input.txt” on HDFS
hdfs dfs -ls /dft-group
```

####  6.1.2 Test the Wordcount

```shell
# The word count program is provided as a jar file in Hadoop, which can be executed in HDFS
yarn jar /opt/hadoop-2.7.5/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.5.jar wordcount /dft-group /dft-group-output-1

# The output is placed in /dft-output in HDFS, we can retrieve it by “copyToLocal”:
hdfs dfs -copyToLocal /dft-group-output-1 ~/
```



#### 6.1.3 Create a new account

```shell
sudo adduser --ingroup hadoop syk

# Create home directory on HDFS for syk
hdfs dfs -mkdir /user
hdfs dfs -mkdir /user/syk
# Set user permission
hdfs dfs -mkdir /tmp
hdfs dfs -chmod -R 777 /tmp
hdfs dfs -chown -R syk:hadoop /user/syk

# run an example using account syk
su syk
source /etc/profile
cd ~
mkdir dft
# download file
scp student@10.42.0.1:~/comp7305/dft/large_input/word_input.txt dft/

# Create a directory in HDFS
hdfs dfs -mkdir /user/syk/dft-group

# Upload this file to Hadoop
hdfs dfs -copyFromLocal /home/syk/dft/word_input.txt /user/syk/dft-group

# The word count program is provided as a jar file in Hadoop, which can be executed in HDFS
yarn jar /opt/hadoop-2.7.5/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.5.jar wordcount /user/syk/dft-group /user/syk/dft-group-output-1
```





* Use Hadoop Web UI “Browse Directory” to show the “owner” of the uploaded word_input.txt file

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-29%20at%208.58.29%20PM.png)



* Use Hadoop Web UI to show the block distribution of your HDFS (all 11 DataNodes should be active)

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-29%20at%209.14.03%20PM.png)

* Report the execution time of the WordCount (also show the number of map/reduce tasks completed)

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-29%20at%209.00.14%20PM.png)



* "Successful MAP attempts" page to show the completed map tasks and their execution location

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-29%20at%209.19.49%20PM.png)









## 7 TeraSort

In syk account:

```shell
cd /opt/hadoop-2.7.5/share/hadoop/mapreduce
# Run TeraGen to generate rows of random data to be sorted.
yarn jar hadoop-mapreduce-examples-2.7.5.jar teragen 100000000 /user/syk/terainput

# Run TeraSort to sort the data
yarn jar hadoop-mapreduce-examples-2.7.5.jar terasort /user/syk/terainput /user/syk/teraoutput-1

# Run TeraValidate to validate the sorted Teragen
yarn jar hadoop-mapreduce-examples-2.7.5.jar teravalidate /user/syk/teraoutput-1 /user/syk/teravalidate
```

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-29%20at%209.08.58%20PM.png)



<img src="https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-29%20at%209.10.21%20PM.png" style="zoom:33%;" />

* Get the Sorting time

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-29%20at%209.24.18%20PM.png)

* Result Validation

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-29%20at%209.23.57%20PM.png)



(Run TeraSort with your own configuration..............to be continue)



## 8 hfs dfs

```shell
# list
hdfs dfs -ls

# list certain dirctory
hdfs dfs -ls [dir]

# rm files
hdfs dfs -rm -r 
```

