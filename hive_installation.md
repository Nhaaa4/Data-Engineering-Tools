# Install Apache Hive

## 1. Install Java

Install **Java 8** to avoid version conflicts with Apache Hadoop and
Hive.

``` bash
sudo apt install openjdk-8-jdk -y
```

## 2. Install Hive

Install Apache Hive **version 3.1.3** (works well with Java 8).

``` bash
wget https://archive.apache.org/dist/hive/hive-3.1.3/apache-hive-3.1.3-bin.tar.gz
tar -xzf apache-hive-3.1.3-bin.tar.gz
sudo mv apache-hive-3.1.3-bin /usr/local/hive
cd /usr/local/hive/conf
cp hive-default.xml.template hive-site.xml
nano hive-site.xml
```

Add the following to **hive-site.xml**:
```xml
  <property>
    <name>system:java.io.tmpdir</name>
    <value>/tmp/hive/java</value>
  </property>
  <property>
    <name>system:user.name</name>
    <value>${user.name}</value>
  </property>
```
### Configure environment variables

``` bash
nano ~/.bashrc

# Hive
export HIVE_HOME=/usr/local/hive
export HIVE_CONF_DIR=$HIVE_HOME/conf
export PATH=$PATH:$HIVE_HOME/bin
export CLASSPATH=$CLASSPATH:$HADOOP_HOME/lib/*:.
export CLASSPATH=$CLASSPATH:$HIVE_HOME/lib/*:.

source ~/.bashrc
```

### Create Hive directories in HDFS

``` bash
hdfs dfs -mkdir -p /user/hive/warehouse
hdfs dfs -mkdir /tmp

hdfs dfs -chmod g+wx /user
hdfs dfs -chmod g+w /tmp
```

### Configure hive-env.sh

``` bash
cp hive-env.sh.template hive-env.sh
nano hive-env.sh
```

Add:
```
HADOOP_HOME=/usr/local/Hadoop
```
### Fix Guava version conflict

``` bash
cd $HIVE_HOME/lib
rm guava-*.0.jar
cp $HADOOP_HOME/share/hadoop/common/lib/guava-*.0-jre.jar $HIVE_HOME/lib
```

### Fix Derby schema

``` bash
cd $HIVE_HOME/scripts/metastore/upgrade/derby/
nano hive-schema-x.y.z.derby.sql
```

Comment out the **first two lines**.

### Install required library

``` bash
cd $HIVE_HOME/lib
wget https://repo1.maven.org/maven2/commons-collections/commons-collections/3.2.2/commons-collections-3.2.2.jar
```

### Initialize Hive schema

``` bash
cd ~
schematool -dbType derby -initSchema
```

### Start hive

```bash
hive
```

or

```bash
beeline -u jdbc:hive2://
```
