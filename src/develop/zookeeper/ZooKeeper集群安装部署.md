# 安装zookeeper集群

<font color=red>注意：三台机器一定要保证时钟同步</font>



## zookeeper分发到node01

### 分发

本机执行

```bash
scp zookeeper-3.4.5-cdh5.14.2.tar.gz hadoop@192.168.2.100:/bigdata/soft
```



### 解压

node01执行

```bash
cd /bigdata/soft
tar -zxvf zookeeper-3.4.5-cdh5.14.2.tar.gz  -C /bigdata/install/
```



### 修改配置文件

node01执行

```bash
cd /bigdata/install/zookeeper-3.4.5-cdh5.14.2/conf
cp zoo_sample.cfg zoo.cfg
mkdir -p /bigdata/install/zookeeper-3.4.5-cdh5.14.2/zkdatas
vi zoo.cfg

# 注释原 dataDir=/tmp/zookeeper
dataDir=/bigdata/install/zookeeper-3.4.5-cdh5.14.2/zkdatas
autopurge.snapRetainCount=3
autopurge.purgeInterval=1
server.1=node01:2888:3888
server.2=node02:2888:3888
server.3=node03:2888:3888
```



### 添加myid配置

node01执行

```bash
echo 1 > /bigdata/install/zookeeper-3.4.5-cdh5.14.2/zkdatas/myid
```



## zookeeper分发到node02和node03

### 分发

node01执行

```bash
scp -r /bigdata/install/zookeeper-3.4.5-cdh5.14.2/ node02:/bigdata/install/ 
scp -r /bigdata/install/zookeeper-3.4.5-cdh5.14.2/ node03:/bigdata/install/
```



### 修改myid配置

node02执行

```bash
echo 2 > /bigdata/install/zookeeper-3.4.5-cdh5.14.2/zkdatas/myid
```



node03执行

```bash
echo 3 > /bigdata/install/zookeeper-3.4.5-cdh5.14.2/zkdatas/myid
```



## 配置环境变量

三台机器分别执行

```bash
vi /etc/profile

export ZOOKEEPER_HOME=/bigdata/install/zookeeper-3.4.5-cdh5.14.2
export PATH=$PATH:$ZOOKEEPER_HOME/bin

# 立即生效
source /etc/profile
```



## 启动zookeeper服务

三台机器分别执行

```bash
# 启动
# jps 每个节点上都有QuorumPeerMain进程
zkServer.sh start

# 查看启动状态
# 一个leader、其他follower
zkServer.sh status

# 停止
zkServer.sh stop
```



# 安装ZooKeeper Docker集群

## 下载镜像

```shell
docker pull zookeeper:3.4.14
```



## 启动服务

- docker logs -f zk01 查看服务运行情况
- 当启动容器后，挂载的本地目录自动创建

```shell
# zk01
docker run --name zk01 --net=mynet --ip=172.72.0.10 \
-v /Users/yangxiaoyu/work/test/zkdatas/node01/data:/data \
-v /Users/yangxiaoyu/work/test/zkdatas/node01/datalog:/datalog \
-v /Users/yangxiaoyu/work/test/zkdatas/node01/logs:/logs \
-e ZOO_MY_ID=1 \
-e "ZOO_SERVERS=server.1=172.72.0.10:2888:3888 server.2=172.72.0.11:2888:3888 server.3=172.72.0.12:2888:3888" \
-d zookeeper:3.4.14

# zk02
docker run --name zk02 --net=mynet --ip=172.72.0.11 \
-v /Users/yangxiaoyu/work/test/zkdatas/node02/data:/data \
-v /Users/yangxiaoyu/work/test/zkdatas/node02/datalog:/datalog \
-v /Users/yangxiaoyu/work/test/zkdatas/node02/logs:/logs \
-e ZOO_MY_ID=2 \
-e "ZOO_SERVERS=server.1=172.72.0.10:2888:3888 server.2=172.72.0.11:2888:3888 server.3=172.72.0.12:2888:3888" \
-d zookeeper:3.4.14

# zk03
docker run --name zk03 --net=mynet --ip=172.72.0.12 \
-v /Users/yangxiaoyu/work/test/zkdatas/node03/data:/data \
-v /Users/yangxiaoyu/work/test/zkdatas/node03/datalog:/datalog \
-v /Users/yangxiaoyu/work/test/zkdatas/node03/logs:/logs \
-e ZOO_MY_ID=3 \
-e "ZOO_SERVERS=server.1=172.72.0.10:2888:3888 server.2=172.72.0.11:2888:3888 server.3=172.72.0.12:2888:3888" \
-d zookeeper:3.4.14
```



## 验证

```shell
# 查看zk01、zk02、zk03的Mode，一个leader、其他follower
docker exec -it zk01 bin/zkServer.sh status
docker exec -it zk02 bin/zkServer.sh status
docker exec -it zk03 bin/zkServer.sh status

# 连接客户端
docker exec -it zk01 bin/zkCli.sh -server 172.72.0.10:2181,172.72.0.11:2181,172.72.0.12:2181
```

