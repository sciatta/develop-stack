# 先决条件

- 安装对应版本的hadoop集群并启动
- 安装对应版本的zookeeper集群并启动



# 服务规划



| IP                   | HMaster | 备份HMaster | HRegionServer |
| -------------------- | ------- | ----------- | ------------- |
| 192.168.1.100 node01 | &radic; |             | &radic;       |
| 192.168.1.101 node02 |         | &radic;     | &radic;       |
| 192.168.1.102 node03 |         |             | &radic;       |



# 安装

```bash
# 上传压缩包到node01
scp hbase-1.2.0-cdh5.14.2.tar.gz hadoop@192.168.2.100:/bigdata/soft

# 解压缩
tar -xzvf hbase-1.2.0-cdh5.14.2.tar.gz -C /bigdata/install/
```



# 修改配置

## hbase-env.sh

```bash
cd /bigdata/install/hbase-1.2.0-cdh5.14.2/conf
vi hbase-env.sh
```



```shell
export JAVA_HOME=/bigdata/install/jdk1.8.0_141
export HBASE_MANAGES_ZK=false
```



## hbase-site.xml

```bash
vi hbase-site.xml
```



```xml
<configuration>
	<property>
		<name>hbase.rootdir</name>
		<value>hdfs://node01:8020/hbase</value>  
	</property>
	<property>
		<name>hbase.cluster.distributed</name>
		<value>true</value>
	</property>
	<!-- 0.98后的新变动，之前版本没有.port。默认端口为60000 -->
	<property>
		<name>hbase.master.port</name>
		<value>16000</value>
	</property>
	<property>
		<name>hbase.zookeeper.quorum</name>
		<value>node01,node02,node03</value>
	</property>
    <!-- 此属性可省略，默认值就是2181 -->
	<property>
		<name>hbase.zookeeper.property.clientPort</name>
		<value>2181</value>
	</property>
	<property>
		<name>hbase.zookeeper.property.dataDir</name>
		<value>/bigdata/install/zookeeper-3.4.5-cdh5.14.2/zkdatas</value>
	</property>
    <!-- 此属性可省略，默认值就是/hbase -->
	<property>
		<name>zookeeper.znode.parent</name>
		<value>/hbase</value>
	</property>
</configuration>
```



## regionservers

```bash
vi regionservers
```



- 指定HBase集群的从节点；原内容清空，添加如下三行

```txt
node01
node02
node03
```



## back-masters

- 创建back-masters配置文件，包含备份HMaster节点的主机名，每个机器独占一行，实现HMaster的高可用

```bash
vi backup-masters
```



- 将node02作为备份的HMaster节点

```text
node02
```



# 分发

```bash
cd /bigdata/install
scp -r hbase-1.2.0-cdh5.14.2/ node02:$PWD
scp -r hbase-1.2.0-cdh5.14.2/ node03:$PWD
```



# 创建软连接

- 注意：<font color=red>三台机器</font>均做如下操作
- 因为HBase集群需要读取hadoop的core-site.xml、hdfs-site.xml的配置文件信息，所以我们三台机器都要执行以下命令，在相应的目录创建这两个配置文件的软连接

```bash
ln -s /bigdata/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop/core-site.xml  /bigdata/install/hbase-1.2.0-cdh5.14.2/conf/core-site.xml

ln -s /bigdata/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop/hdfs-site.xml  /bigdata/install/hbase-1.2.0-cdh5.14.2/conf/hdfs-site.xml
```



# 添加HBase环境变量

- 注意：<font color=red>三台机器</font>均做如下操作

```bash
sudo vi /etc/profile
```



- 文件末尾添加如下内容

```
export HBASE_HOME=/bigdata/install/hbase-1.2.0-cdh5.14.2
export PATH=$PATH:$HBASE_HOME/bin
```



- 重新编译/etc/profile使环境变量立即生效

```bash
source /etc/profile
```



# HBase启动和停止

- 启动

```bash
# node01 执行
# 启动 hdfs
start-dfs.sh

# node01 执行
# 启动 yarn
start-yarn.sh

# node01 执行
# 启动 jobhistory
mr-jobhistory-daemon.sh start historyserver

# node01 node02 node03 分别执行
# 启动 zookeeper
# 检查 zkServer.sh status
zkServer.sh start

# node01 执行
# 启动 hbase
start-hbase.sh
```



- 停止

```bash
# node01执行
# 停止 hbase
stop-hbase.sh

# node01 node02 node03 分别执行
# 停止 zookeeper
zkServer.sh stop

# node01执行
# 停止 jobhistory
mr-jobhistory-daemon.sh stop historyserver

# node01执行
# 停止 yarn
stop-yarn.sh

# node01 执行
# 停止 hdfs
stop-dfs.sh
```



- web页面

| 服务       | web页面                    |
| ---------- | -------------------------- |
| hdfs       | http://192.168.2.100:50070 |
| yarn       | http://192.168.2.100:8088  |
| jobhistory | http://192.168.2.100:19888 |
| hbase      | http://192.168.2.100:60010 |

