# Install Apache Hive

This guide explains how to install and configure Apache Hive 3.1.3 on Ubuntu using Java 8 and Derby as the default metastore. Before install Apache Hive, You must install Apache Hadoop first. [Apache Hadoop Installation](https://github.com/Nhaaa4/Data-Engineering-Tools/blob/main/hadoop_installation.md)

## 1. Install Java (JDK 8)

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
```

Then, Hive directory must be accessible by the Hadoop user (or the user running HDFS).

```bash
sudo chown -R hadoop:hadoop /usr/local/hive
sudo chmod -R 755 /usr/local/hive
```

If your Hadoop username is different, replace `hadoop` accordingly.

## 3. Configure Hive

### 3.1 Update hive-site.xml

Go to the configuration directory:

```bash
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

### 3.2 Set Hive metastore path

Find the property for `ConnectionURL` and change it from:

```xml
<property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:derby:;databaseName=metastore_db;create=true</value>
    <description>
      JDBC connect string for a JDBC metastore.
      To use SSL to encrypt/authenticate the connection, provide database-specific SSL flag in the connection URL.
      For example, jdbc:postgresql://myhost/db?ssl=true for postgres database.
    </description>
  </property>
```

to:

```xml
<property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:derby:;databaseName=/usr/local/hive/metastore_db;create=true</value>
    <description>
      JDBC connect string for a JDBC metastore.
      To use SSL to encrypt/authenticate the connection, provide database-specific SSL flag in the connection URL.
      For example, jdbc:postgresql://myhost/db?ssl=true for postgres database.
    </description>
  </property>
```

### 3.3 Fix invalid XML character

At around line 3223, remove the `&#8;` characters inside the description.

Change this:

```xml
 <property>
    <name>hive.txn.xlock.iow</name>
    <value>true</value>
    <description>
      Ensures commands with OVERWRITE (such as INSERT OVERWRITE) acquire Exclusive locks for&#8;transactional tables.  This ensures that inserts (w/o overwr
      are not hidden by the INSERT OVERWRITE.
    </description>
  </property>
```

to:

```xml
 <property>
    <name>hive.txn.xlock.iow</name>
    <value>true</value>
    <description>
      Ensures commands with OVERWRITE (such as INSERT OVERWRITE) acquire Exclusive locks for transactional tables.  This ensures that inserts (w/o overwr
      are not hidden by the INSERT OVERWRITE.
    </description>
  </property>
```

Save the file.

## 4. Configure environment variables

Open your bash profile:

``` bash
nano ~/.bashrc
```

Add:

```shell
# Hive
export HIVE_HOME=/usr/local/hive
export HIVE_CONF_DIR=$HIVE_HOME/conf
export PATH=$PATH:$HIVE_HOME/bin
export CLASSPATH=$CLASSPATH:$HADOOP_HOME/lib/*:.
export CLASSPATH=$CLASSPATH:$HIVE_HOME/lib/*:.
```

Apply:

```bash
source ~/.bashrc
```

## 5. Configure hive-env.sh

``` bash
cp hive-env.sh.template hive-env.sh
nano hive-env.sh
```

Add:
```
HADOOP_HOME=/usr/local/hadoop
```
## 6. Fix Guava version conflict

Hive 3.1.3 must use the same guava version as Hadoop.

``` bash
cd $HIVE_HOME/lib
rm guava-*.0.jar
cp $HADOOP_HOME/share/hadoop/common/lib/guava-*.0-jre.jar $HIVE_HOME/lib
```

## 7. Fix Derby schema

Go to the derby schema upgrade folder:

``` bash
cd $HIVE_HOME/scripts/metastore/upgrade/derby/
```

Open the latest Derby schema file:

```bash
nano hive-schema-x.y.z.derby.sql
```

Comment out the first two lines (they cause duplicate table creation).

## 8. Install Missing Collections Library (if needed)

If you see an error about org.apache.commons.collections, install:

``` bash
cd $HIVE_HOME/lib
wget https://repo1.maven.org/maven2/commons-collections/commons-collections/3.2.2/commons-collections-3.2.2.jar
```

## 9. Initialize Hive schema

``` bash
cd $HIVE_HOME
schematool -dbType derby -initSchema
```

## 10. Create Hive directories in HDFS

``` bash
hdfs dfs -mkdir -p /user/hive/warehouse
hdfs dfs -mkdir /tmp

hdfs dfs -chmod g+wx /user
hdfs dfs -chmod g+w /tmp
```

## 11. Start hive

Start Hive CLI:

```bash
hive
```

Or connect using Beeline:

```bash
beeline -u jdbc:hive2://
```

## 12. References:
- [IvyProSchool](https://share.google/FKNciZwK9lmi30K6N)
