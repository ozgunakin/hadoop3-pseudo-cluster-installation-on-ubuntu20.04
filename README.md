# Hadoop3 Pseudo Cluster Installation on Ubuntu20.04

This is a guide for Hadoop3 Pseudo Cluster Mode (One Node) installation on Ubuntu 20.04

## Step 1 - Install Java

If java is not installed on your servers, you can install java by following the instructions in;&#x20;

NOTE: We will be using java 8 for this project.&#x20;

{% embed url="https://github.com/ozgunakin/java-installation-on-ubuntu20.04" %}

## Step 2 - Configure SSH

You need to install open ssh the node and you need to configure a passwordless ssh connection.

* [x] Install Open SSH server and client.

```
sudo apt-get install openssh-server openssh-client
```

* [x] Generate Key Pairs

```
ssh-keygen -t rsa
```

* [x] Add the content of .ssh/id_rsa.pub(of master) to .ssh/authorized\__keys

```
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

* [x] Configure file permissions.

```
chmod 640 ~/.ssh/authorized_keys
```

* [x] Test your connection

```
ssh localhost
```

## Step 3 - Install Hadoop3

* [x] Download Hadoop

```
wget https://downloads.apache.org/hadoop/common/hadoop-3.3.1/hadoop-3.3.1.tar.gz
```

* [x] Extract the file

```
tar -xvzf hadoop-3.3.1.tar.gz
```

* [x] Move the extracted file to the opt directory

```
sudo mv hadoop-3.3.1.tar.gz /opt/
```

* [x] Open .bashrc file

```
nano ~/.bashrc
```

* [x] Add following lines into .bashrc file

```
#HADOOP ENVIRONMENT VARIABLES
export HADOOP_HOME=/opt/hadoop-3.3.1
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=${HADOOP_HOME}
export HADOOP_COMMON_HOME=${HADOOP_HOME}
export HADOOP_HDFS_HOME=${HADOOP_HOME}
export YARN_HOME=${HADOOP_HOME}
```

* [x] Source .bashrc file to apply the changes&#x20;

```
source ~/.bashrc
```

## Step 4 - Configure Hadoop3

We will configure 5 important files of Hadoop.

#### File 1 : hadoop-env.sh

* [x] Open hadoop-env.sh file.

```
nano $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```

* [x] Add following lines into hadoop-env.sh file

```
#JAVA HOME PATH
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

* [x] Create necessary directories for hadoop and change the owner of the directories.

```
mkdir  /usr/local/hadoop/hdfs/datanode
mkdir  /usr/local/hadoop/hdfs/namenode
sudo chown ubuntu:ubuntu -R /usr/local/hadoop/hdfs/datanode
sudo chown ubuntu:ubuntu -R /usr/local/hadoop/hdfs/namenode
```

#### File 2 : core-site.xml

* [x] Open core-site.xml file.

```
nano $HADOOP_HOME/etc/hadoop/core-site.xml
```

* [x] Add following lines into core-site.xml file

```
<configuration>
   <property>
      <name>fs.default.name</name>
      <value>hdfs://localhost:9000</value>
      <description>The default file system URI</description>
   </property>
</configuration>
```

#### File 3 : hdfs-site.xml

* [x] Open hdfs-site.xml file.

```
nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```

* [x] Add following lines into hdfs-site.xml file

```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/hadoop/hdfs/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/local/hadoop/hdfs/datanode</value>
    </property>
</configuration>
```

#### File 4 : mapred-site.xml

* [x] Open mapred-site.xml file.

```
nano $HADOOP_HOME/etc/hadoop/mapred-site.xml
```

* [x] Add following lines into hdfs-site.xml file

```
<configuration>
   <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
   </property>
<property>
  <name>yarn.app.mapreduce.am.env</name>
  <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
</property>
<property>
  <name>mapreduce.map.env</name>
  <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
</property>
<property>
  <name>mapreduce.reduce.env</name>
  <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
</property>
</configuration>
```

#### File 5 : yarn-site.xml

* [x] Open yarn-site.xml file.

```
nano $HADOOP_HOME/etc/hadoop/yarn-site.xml
```

* [x] Add following lines into yarn-site.xml file

```
<configuration>
<!-- Site specific YARN configuration properties -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>127.0.0.1</value>
    </property>
</configuration>
```

## Step 5 - Run Hadoop3

* [x] We need to format namenode before starting hadoop (this is one time task do not run this more than one)

```
hdfs namenode -format
```

* [x] Start Distributed File System

```
start-dfs.sh
```

* [x] Start Yarn

```
start-yarn.sh
```

## Step 6 - Test Your Installation

* [x] Run jps command to see the status of your hadoop system.

```
jps
```

* [x] The output.

![](<.gitbook/assets/image (1).png>)

#### Pi Example on MapReduce

* [x] To run the pi example with 16 maps and 100000 samples, run the following command

```
hadoop jar /opt/hadoop-3.3.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.1.jar pi 16 100000
```

* [x] The output.

![](.gitbook/assets/image.png)



#### WordCount Example on MapReduce

* [x] Create necessary files and put some txt file in hdfs. In this case we will use alice.txt file.

```
hdfs dfs -mkdir -p /user/ubuntu
wget -O alice.txt https://www.gutenberg.org/files/11/11-0.txt
hdfs dfs -put ./alice.txt books/alice.txt
hdfs dfs -ls
hdfs dfs -ls books/alice.txt

hdfs dfs -mkdir -p /user/ubuntu/outputs
```

* [x] Run MapReduce WordCount Job to count words in alice.txt

```
hadoop jar /opt/hadoop-3.3.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.1.jar wordcount /user/ubuntu/books/alice.txt  /user/ubuntu/outputs/trial
```

* [x] Read the output

```
hadoop fs -cat /user/ubuntu/outputs/trial/part-r-00000
```

* [x] The output.

![](<.gitbook/assets/image (2).png>)

## &#x20;Step 7 - Run Hadoop & Yarn on Reboot

* [x] We will set cron job to run hadoop and yarn on reboot.

```
crontab -e
```

* [x] Add following lines.

```
#Start Hadoop on Reboot
@reboot start-dfs.sh
@reboot start-yarn.sh
```

## Congratulations :)

Your Hadoop is up and running !!!
