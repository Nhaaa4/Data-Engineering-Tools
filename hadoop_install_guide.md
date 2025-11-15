# Installing Hadoop on Ubuntu 24.04

This guide provides a human-friendly overview of how to install Hadoop
on **Ubuntu 24.04**, following the steps provided by the reference guide
from Cherry Servers.

## Overview

Apache Hadoop is an open-source framework used for distributed storage
and processing of large datasets. This document summarizes installation
steps useful for Data Engineering learners.

## Key Steps

### 1. Update System Packages

``` bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install Java (OpenJDK)

``` bash
sudo apt install openjdk-11-jdk -y
```

### 3. Create Hadoop User

``` bash
sudo adduser hadoop
sudo usermod -aG sudo hadoop
```

### 4. Configure SSH Access

``` bash
ssh-keygen -t rsa -P ""
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

### 5. Download & Install Hadoop

``` bash
wget https://downloads.apache.org/hadoop/common/hadoop-3.4.0/hadoop-3.4.0.tar.gz
tar -xvf hadoop-3.4.0.tar.gz
sudo mv hadoop-3.4.0 /usr/local/hadoop
```

### 6. Set Environment Variables

``` bash
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

Apply:

``` bash
source ~/.bashrc
```

### 7. Configure XML Files

You must configure: - core-site.xml\
- hdfs-site.xml\
- mapred-site.xml\
- yarn-site.xml

### 8. Format Namenode

``` bash
hdfs namenode -format
```

### 9. Start Hadoop Services

``` bash
start-dfs.sh
start-yarn.sh
```

### 10. Verify Installation

    http://localhost:9870   # NameNode UI
    http://localhost:8088   # YARN UI

## Reference

Guide adapted from Cherry Servers:\
https://www.cherryservers.com/blog/install-hadoop-ubuntu-2404
