# 安装

安装在node03上

```shell
# 拷贝sqoop到node03
scp sqoop-1.4.6-cdh5.14.2.tar.gz hadoop@node03:/bigdata/soft

# 解压
tar -zxf sqoop-1.4.6-cdh5.14.2.tar.gz -C ../install/
```



# 修改配置文件

## sqoop-env.sh

```shell
cd /bigdata/install/sqoop-1.4.6-cdh5.14.2/conf

cp sqoop-env-template.sh sqoop-env.sh

vi sqoop-env.sh

#Set path to where bin/hadoop is available
export HADOOP_COMMON_HOME=/bigdata/install/hadoop-2.6.0-cdh5.14.2

#Set path to where hadoop-*-core.jar is available
export HADOOP_MAPRED_HOME=/bigdata/install/hadoop-2.6.0-cdh5.14.2

#set the path to where bin/hbase is available
export HBASE_HOME=/bigdata/install/hbase-1.2.0-cdh5.14.2

#Set the path to where bin/hive is available
export HIVE_HOME=/bigdata/install/hive-1.1.0-cdh5.14.2
```



# 添加jar

## 支持MySQL

```shell
scp java-json.jar mysql-connector-java-5.1.38.jar hadoop@node03:/bigdata/install/sqoop-1.4.6-cdh5.14.2/lib
```



## 支持Hive

```shell
cd /bigdata/install/hive-1.1.0-cdh5.14.2/lib
cp hive-exec-1.1.0-cdh5.14.2.jar ../../sqoop-1.4.6-cdh5.14.2/lib/
```



# 验证

```shell
# 列出mysql的所有数据库
bin/sqoop list-databases --connect jdbc:mysql://node03:3306/ --username root --password root
```

