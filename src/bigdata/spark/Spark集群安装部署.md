# 先决条件

- 安装ZooKeeper集群并启动



# 服务规划

| 服务器 | Master          | Worker  |
| ------ | --------------- | ------- |
| node01 | &radic; alive   |         |
| node02 |                 | &radic; |
| node03 | &radic; standby | &radic; |



# 安装

```bash
scp spark-2.3.3-bin-hadoop2.7.tgz hadoop@node01:/bigdata/soft

tar -zxvf spark-2.3.3-bin-hadoop2.7.tgz -C /bigdata/install/
```



# 修改配置

## spark-env.sh

```bash
cp spark-env.sh.template spark-env.sh

vi spark-env.sh

# 配置java的环境变量
export JAVA_HOME=/bigdata/install/jdk1.8.0_141

# 配置zk相关信息
export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER  -Dspark.deploy.zookeeper.url=node01:2181,node02:2181,node03:2181  -Dspark.deploy.zookeeper.dir=/spark"

# 指向hadoop配置文件路径
export HADOOP_CONF_DIR=/bigdata/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop
```



## slaves

```bash
cp slaves.template slaves

# 指定spark集群的worker节点
node02
node03
```



# 分发

```bash
scp -r /bigdata/install/spark-2.3.3-bin-hadoop2.7 node02:/bigdata/install
scp -r /bigdata/install/spark-2.3.3-bin-hadoop2.7 node03:/bigdata/install
```



# 配置环境变量

三台机器均需修改

```bash
sudo vi /etc/profile

export SPARK_HOME=/bigdata/install/spark-2.3.3-bin-hadoop2.7
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin

# 立即生效
source /etc/profile
```



# 启动

node01执行

```bash
# 切换到sbin下运行，会有同名文件冲突
cd /bigdata/install/spark-2.3.3-bin-hadoop2.7/sbin

# 启动master和worker
# 注意启动的机器就是master，而worker由slave决定
start-all.sh
```



node03执行

```bash
# 高可用
start-master.sh
```



# 停止

node01执行

```bash
cd /bigdata/install/spark-2.3.3-bin-hadoop2.7/sbin

stop-all.sh
```



node03执行

```bash
stop-master.sh
```



# 验证

访问页面

alive：`http://node01:8080/`

standby：`http://node03:8080/`

