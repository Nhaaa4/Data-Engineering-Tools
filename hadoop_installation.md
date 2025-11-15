# Installing Apache Hadoop on Ubuntu 24.04

## Key Steps

### 1. Update System Packages

``` bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install Java (OpenJDK 8)

``` bash
sudo apt install openjdk-8-jdk -y
```

Verifies installation:

```bash
java -version
```

### 3. Create Hadoop User

``` bash
sudo adduser hadoop
sudo usermod -aG sudo hadoop
su - hadoop
```

### 4. Configure SSH Access

Install OpenSSH of not already present:

```bash
sudo apt install openssh-server openssh-client -y
sudo systemctl enable ssh
```

Generate SSH key pair and authorize for localhost:

``` bash
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
ssh localhost
```

then

```bash
exit
```

### 5. Download & Install Hadoop

``` bash
wget https://downloads.apache.org/hadoop/common/hadoop-3.4.2/hadoop-3.4.2.tar.gz
tar -xvf hadoop-3.4.2.tar.gz
sudo mv hadoop-3.4.2 /usr/local/hadoop
sudo chown -R hadoop:hadoop /usr/local/hadoop
```

### 6. Set Environment Variables

``` bash
# Hadoop Environment
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"

# Java Environment
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin
```

Apply:

``` bash
source ~/.bashrc
```

Then, set Java for Hadoop:

```bash
nano $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```

Add:

```shell
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

Save the file.

### 7. Configure XML Files

Create HDFS storage directories:

```bash
mkdir -p $HADOOP_HOME/hdfs/namenode
mkdir -p $HADOOP_HOME/hdfs/datanode
sudo chown -R hadoop:hadoop $HADOOP_HOME/hdfs
```

You must configure: 
- core-site.xml
- hdfs-site.xml
- mapred-site.xml
- yarn-site.xml

1. Open the `core-site.xml` file:

```bash
nano $HADOOP_HOME/etc/hadoop/core-site.xml
```

Replace the empty `<configuration></configuration>` tags with the following:

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
        <description>The default file system URI</description>
    </property>
</configuration>
```

2. Open the `hdfs-site.xml` file:

```bash
nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```

Add the following between the `<configuratioin></configuratioin>` tags:

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
        <description>Default block replication.</description>
    </property>
    <property>
        <name>dfs.name.dir</name>
        <value>file:///usr/local/hadoop/hdfs/namenode</value>
        <description>Path on the local filesystem where the NameNode stores the namespace and transaction logs.</description>
    </property>
    <property>
        <name>dfs.data.dir</name>
        <value>file:///usr/local/hadoop/hdfs/datanode</value>
        <description>Path on the local filesystem where the DataNode stores its blocks.</description>
    </property>
</configuration>
```

3. Open the `mapred-site.xml` file:

```bash
nano $HADOOP_HOME/etc/hadoop/mapreed-site.xml
```

Replace the empty `<configuration></configuration>` tags with:

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
        <description>The runtime framework for MapReduce. Can be local, classic or yarn.</description>
    </property>
    <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=/usr/local/hadoop</value>
    </property>
    <property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=/usr/local/hadoop</value>
    </property>
    <property>
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=/usr/local/hadoop</value>
    </property>
</configuration>
```

4. Open the `yarn-site.xml` file:

```bash
nano $HADOOP_HOME/etc/hadoop/yarn-site.xml
```

Add the following between the `<configuration>` tags:

```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
        <description>Auxilliary services required by the NodeManager.</description>
    </property>
</configuration>
```

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

- [Guide adapted from Cherry Servers](https://www.cherryservers.com/blog/install-hadoop-ubuntu-2404)
