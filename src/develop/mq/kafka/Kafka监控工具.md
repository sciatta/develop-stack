# Kafka Manager

kafkaManager是由雅虎开源的可以监控整个kafka集群相关信息的一个工具。

- 可以管理几个不同的集群
- 监控集群的状态(topics, brokers, 副本分布, 分区分布)
- 创建topic、修改topic相关配置

## 安装

```shell
# 安装zip工具
sudo yum install -y zip
sudo yum install -y unzip

scp kafka-manager-1.3.0.4.zip hadoop@node01:/bigdata/soft

unzip kafka-manager-1.3.0.4.zip -d ../install
```

## 配置

```shell
cd /bigdata/install/kafka-manager-1.3.0.4/conf
vi application.conf

kafka-manager.zkhosts="node01:2181,node02:2181,node03:2181"
```

## 启动

- 启动ZooKeeper集群

- 启动kafka集群

- 启动 kafka manager 服务

  <font color=red>注意：必须使用root用户执行命令启动 kafka manager 服务</font>

  ```shell
  cd /bigdata/install/kafka-manager-1.3.0.4
  # 切换到root用户下执行命令
  su root
  nohup bin/kafka-manager -Dconfig.file=conf/application.conf -Dhttp.port=8080 &
  ```

## 停止

```shell
# 查看后台运行进程
jobs -l
# 结束进程
kill -9 28754
```

## 使用

- 访问服务地址 `http://node01:8080/`

- 添加Cluster

  Cluster | Add Cluster

  - Cluster name: kafka
  - Cluster ZooKeeper hosts: node01:2181,node02:2181,node03:2181



# KafkaOffsetMonitor

该监控是基于一个jar包的形式运行，部署较为方便。只有监控功能，使用起来也较为安全。

- 消费者组列表
- 查看topic的历史消费信息
- 每个topic的所有parition列表
- 对consumer消费情况进行监控，并可列出每个consumer offset，滞后数据

## 安装

```shell
cd /bigdata/install/
mkdir kafka_offset_moitor
scp KafkaOffsetMonitor-assembly-0.2.0.jar hadoop@node01:/bigdata/install/kafka_offset_moitor
```

在 `kafka_offset_moitor` 目录下新建 `start_kafka_web.sh`

```shell
vi start_kafka_web.sh

#!/bin/sh
java -cp KafkaOffsetMonitor-assembly-0.2.0.jar com.quantifind.kafka.offsetapp.OffsetGetterWeb --zk node01:2181,node02:2181,node03:2181 --port 8089 --refresh 10.seconds --retain 1.days
```

## 启动

```shell
nohup sh start_kafka_web.sh &
```

## 停止

```shell
jps
# OffsetGetterWeb
kill -9 29966
```

## 使用

- 访问服务地址 `http://node01:8089/`



# Kafka Eagle（推荐）

## 安装

```shell
scp kafka-eagle-bin-1.2.3.tar.gz hadoop@node01:/bigdata/soft/
tar -xzvf kafka-eagle-bin-1.2.3.tar.gz -C ../install/

cd /bigdata/install/kafka-eagle-bin-1.2.3
tar -zxvf kafka-eagle-web-1.2.3-bin.tar.gz
```

## 配置

```shell
cd /bigdata/install/kafka-eagle-bin-1.2.3/kafka-eagle-web-1.2.3/conf
vi system-config.properties
```

system-config.properties

```properties
kafka.eagle.zk.cluster.alias=cluster1
cluster1.zk.list=node01:2181,node02:2181,node03:2181

kafka.eagle.sasl.client=/bigdata/install/kafka-eagle-bin-1.2.3/kafka-eagle-web-1.2.3/conf/kafka_client_jaas.conf

# 数据库自动构建
kafka.eagle.driver=com.mysql.jdbc.Driver
kafka.eagle.url=jdbc:mysql://node03:3306/ke?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
kafka.eagle.username=root
kafka.eagle.password=root
```

环境变量

```shell
sudo vi /etc/profile

export KE_HOME=/bigdata/install/kafka-eagle-bin-1.2.3/kafka-eagle-web-1.2.3
export PATH=$PATH:$KE_HOME/bin

# 立即生效
source /etc/profile
```

## 启动

- 启动mysql服务

- 运行 Kafka Eagle

  ```shell
  cd /bigdata/install/kafka-eagle-bin-1.2.3/kafka-eagle-web-1.2.3/bin
  # Bootstrap java进程
  sh ke.sh start
  ```

## 停止

```shell
cd /bigdata/install/kafka-eagle-bin-1.2.3/kafka-eagle-web-1.2.3/bin
sh ke.sh stop
```

## 使用

- 访问服务地址 `http://node01:8048/ke`
  - 默认用户名：admin
  - 默认密码：123456

