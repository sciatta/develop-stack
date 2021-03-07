# 先决条件

- 启动ZooKeeper集群

  

# 安装

在node01执行

```shell
tar -xzvf kafka_2.11-1.0.1.tgz -C ../install/
```



# 修改node01配置文件

## server.properties

```properties
# 每个broker唯一
broker.id=0

# 数据存放目录
log.dirs=/bigdata/install/kafka_2.11-1.0.1/kafka-logs

# 指定zk地址
zookeeper.connect=node01:2181,node02:2181,node03:2181

# 指定是否可以删除topic，默认是false 表示不可以删除
delete.topic.enable=true

# 指定broker主机名
host.name=node01
```



# 分发

分发到node02和node03

```shell
scp -r kafka_2.11-1.0.1 node02:/bigdata/install
scp -r kafka_2.11-1.0.1 node03:/bigdata/install
```



# 修改node02配置文件

## server.properties

```properties
broker.id=1

host.name=node02
```



# 修改node03配置文件

## server.properties

```properties
broker.id=2

host.name=node03
```



# 启动

所有节点分别运行

```shell
# /bigdata/install/kafka_2.11-1.0.1/logs 是log4j应用日志
nohup bin/kafka-server-start.sh /bigdata/install/kafka_2.11-1.0.1/config/server.properties > /dev/null 2>&1 &
```



# 停止

```shell
bin/kafka-server-stop.sh
```

