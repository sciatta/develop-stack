# 集群规划

- 集群共3个节点，分别是node01、node02和node03
- node01上运行 Active NameNode，node02上运行 Standby NameNode
- node02上运行 Active ResourceManager，node03上运行 Standby ResourceManager

| 运行进程         | node01  | node02  | node03  |
| ---------------- | ------- | ------- | ------- |
| NameNode         | &radic; | &radic; |         |
| zkfc             | &radic; | &radic; |         |
| Datanode         | &radic; | &radic; | &radic; |
| Journalnode      | &radic; | &radic; | &radic; |
| ResourceManager  |         | &radic; | &radic; |
| NodeManager      | &radic; | &radic; | &radic; |
| JobHistoryServer |         |         | &radic; |
| ZooKeeper        | &radic; | &radic; | &radic; |



# 环境搭建

## 运行环境配置

运行环境配置参考

[Hadoop集群安装部署](Hadoop集群安装部署.md)

[ZooKeeper集群安装部署](../zookeeper/ZooKeeper集群安装部署.md)



## Hadoop HA 搭建

### 解压hadoop压缩包

node01执行

```shell
cd /bigdata/soft
mkdir ../install/hadoop-2.6.0-cdh5.14.2-ha
tar -xzvf hadoop-2.6.0-cdh5.14.2_after_compile.tar.gz -C ../install/hadoop-2.6.0-cdh5.14.2-ha
```



### 修改hadoop-env.sh

node01执行

```shell
cd /bigdata/install/hadoop-2.6.0-cdh5.14.2-ha/hadoop-2.6.0-cdh5.14.2/etc/hadoop

# 修改hadoop-env.sh
export JAVA_HOME=/bigdata/install/jdk1.8.0_141
```



### 修改core-site.xml

node01执行

```xml
<configuration>
	<!-- 指定hdfs的nameservice id为ns1 -->
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://ns1</value>
	</property>
	<!-- 指定hadoop临时文件存储的基目录 -->
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/bigdata/install/hadoop-2.6.0-cdh5.14.2-ha/hadoop-2.6.0-cdh5.14.2/tmp</value>
	</property>
	<!-- 指定zookeeper地址，ZKFailoverController使用 -->
	<property>
		<name>ha.zookeeper.quorum</name>
		<value>node01:2181,node02:2181,node03:2181</value>
	</property>
</configuration>
```



### 修改hdfs-site.xml

node01执行

```xml
<configuration>
	<!--指定hdfs的nameservice列表，多个之间逗号分隔；此处只有一个ns1，需要和core-site.xml中的保持一致 -->
	<property>
		<name>dfs.nameservices</name>
		<value>ns1</value>
	</property>
	<!-- ns1下面有两个NameNode，分别是nn1，nn2 -->
	<property>
		<name>dfs.ha.namenodes.ns1</name>
		<value>nn1,nn2</value>
	</property>
	<!-- nn1的RPC通信地址 -->
	<property>
		<name>dfs.namenode.rpc-address.ns1.nn1</name>
		<value>node01:8020</value>
	</property>
	<!-- nn1的http通信地址,web访问地址 -->
	<property>
		<name>dfs.namenode.http-address.ns1.nn1</name>
		<value>node01:50070</value>
	</property>
	<!-- nn2的RPC通信地址 -->
	<property>
		<name>dfs.namenode.rpc-address.ns1.nn2</name>
		<value>node02:8020</value>
	</property>
	<!-- nn2的http通信地址,web访问地址 -->
	<property>
		<name>dfs.namenode.http-address.ns1.nn2</name>
		<value>node02:50070</value>
	</property>
	<!-- 指定NameNode的元数据在JournalNode上的存放位置,主机名修改成实际安装zookeeper的虚拟机的主机名 -->
	<property>
		<name>dfs.namenode.shared.edits.dir</name>
		<value>qjournal://node01:8485;node02:8485;node03:8485/ns1</value>
	</property>
	<!-- 指定JournalNode在本地磁盘存放数据的位置 -->
	<property>
		<name>dfs.journalnode.edits.dir</name>
		<value>/bigdata/install/hadoop-2.6.0-cdh5.14.2-ha/hadoop-2.6.0-cdh5.14.2/journal</value>
	</property>
	<!-- 开启NameNode失败自动切换 -->
	<property>
		<name>dfs.ha.automatic-failover.enabled</name>
		<value>true</value>
	</property>
	<!-- 此类决定哪个namenode是active，切换active和standby -->
	<property>
		<name>dfs.client.failover.proxy.provider.ns1</name>
		<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
	</property>
	<!-- 配置隔离机制方法，多个机制用换行分割，即每个机制用一行-->
	<property>
		<name>dfs.ha.fencing.methods</name>
		<value>
		sshfence
		shell(/bin/true)
		</value>
	</property>
	<!-- 使用sshfence隔离机制时需要ssh免密登陆到目标机器 -->
	<property>
		<name>dfs.ha.fencing.ssh.private-key-files</name>
		<value>/home/hadoop/.ssh/id_rsa</value>
	</property>
	<!-- 配置sshfence隔离机制超时时间 -->
	<property>
		<name>dfs.ha.fencing.ssh.connect-timeout</name>
		<value>30000</value>
	</property>
</configuration>
```



### 修改mapred-site.xml

node01执行

```xml
<configuration>
	<!-- 指定运行 mr job 的运行时框架为yarn -->
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
    <!-- MapReduce JobHistory Server IPC host:port -->
	<property>
		<name>mapreduce.jobhistory.address</name>
		<value>node03:10020</value>
	</property>
	<!-- MapReduce JobHistory Server Web UI host:port -->
	<property>
		<name>mapreduce.jobhistory.webapp.address</name>
		<value>node03:19888</value>
	</property>
</configuration>
```



### 修改yarn-site.xml

node01执行

```xml
<configuration>
  <!-- 是否启用日志聚合.应用程序完成后,日志汇总收集每个容器的日志,这些日志移动到文件系统,例如HDFS. -->
	<!-- 用户可以通过配置"yarn.nodemanager.remote-app-log-dir"、"yarn.nodemanager.remote-app-log-dir-suffix"来确定日志移动到的位置 -->
	<!-- 用户可以通过应用程序时间服务器访问日志 -->
	<!-- 启用日志聚合功能，应用程序完成后，收集各个节点的日志到一起便于查看 -->
	<property>
			<name>yarn.log-aggregation-enable</name>
			<value>true</value>
	</property>
	<!-- 开启RM高可靠 -->
	<property>
		<name>yarn.resourcemanager.ha.enabled</name>
		<value>true</value>
	</property>
	<!-- 指定RM的cluster id为yrc，意为yarn cluster -->
	<property>
		<name>yarn.resourcemanager.cluster-id</name>
		<value>yrc</value>
	</property>
	<!-- 指定RM的名字 -->
	<property>
		<name>yarn.resourcemanager.ha.rm-ids</name>
		<value>rm1,rm2</value>
	</property>
	<!-- 指定第一个RM的地址 -->
	<property>
		<name>yarn.resourcemanager.hostname.rm1</name>
		<value>node02</value>
	</property>
    <!-- 指定第二个RM的地址 -->
	<property>
		<name>yarn.resourcemanager.hostname.rm2</name>
		<value>node03</value>
	</property>
  <!-- 配置第一台机器的resourceManager通信地址 -->
	<!-- 客户端通过该地址向RM提交对应用程序操作 -->
	<property>
		<name>yarn.resourcemanager.address.rm1</name>
		<value>node02:8032</value>
	</property>
	<!-- 向RM调度资源地址 --> 
	<property>
		<name>yarn.resourcemanager.scheduler.address.rm1</name>
		<value>node02:8030</value>
	</property>
	<!-- NodeManager通过该地址交换信息 -->
	<property>
		<name>yarn.resourcemanager.resource-tracker.address.rm1</name>
		<value>node02:8031</value>
	</property>
	<!-- 管理员通过该地址向RM发送管理命令 -->
	<property>
		<name>yarn.resourcemanager.admin.address.rm1</name>
		<value>node02:8033</value>
	</property>
	<!-- RM HTTP访问地址,查看集群信息 -->
	<property>
		<name>yarn.resourcemanager.webapp.address.rm1</name>
		<value>node02:8088</value>
	</property>
	<!-- 配置第二台机器的resourceManager通信地址 -->
	<property>
		<name>yarn.resourcemanager.address.rm2</name>
		<value>node03:8032</value>
	</property>
	<property>
		<name>yarn.resourcemanager.scheduler.address.rm2</name>
		<value>node03:8030</value>
	</property>
	<property>
		<name>yarn.resourcemanager.resource-tracker.address.rm2</name>
		<value>node03:8031</value>
	</property>
	<property>
		<name>yarn.resourcemanager.admin.address.rm2</name>
		<value>node03:8033</value>
	</property>
	<property>
		<name>yarn.resourcemanager.webapp.address.rm2</name>
		<value>node03:8088</value>
	</property>
    <!-- 开启resourcemanager自动恢复功能 -->
	<property>
		<name>yarn.resourcemanager.recovery.enabled</name>
		<value>true</value>
	</property>	
  <!-- 在node2上配置rm1,在node3上配置rm2,注意：一般都喜欢把配置好的文件远程复制到其它机器上，但这个在YARN的另一个机器上一定要修改，其他机器上不配置此项 -->
	<!--
    <property>       
		<name>yarn.resourcemanager.ha.id</name>
		<value>rm1</value>
	   <description>If we want to launch more than one RM in single node, we need this configuration</description>
	</property>
	-->
	<!--用于持久存储的类。尝试开启-->
	<property>
		<name>yarn.resourcemanager.store.class</name>
		<!-- 基于zookeeper的实现 -->
		<value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
	</property>
  <!-- 单个任务可申请最少内存，默认1024MB -->
	<property>
		<name>yarn.scheduler.minimum-allocation-mb</name>
		<value>512</value>
	</property>
	<!-- 多长时间聚合删除一次日志 此处 -->
	<property>
		<name>yarn.log-aggregation.retain-seconds</name>
		<value>2592000</value><!--30 day-->
	</property>
	<!-- 时间在几秒钟内保留用户日志。只适用于如果日志聚合是禁用的 -->
	<property>
		<name>yarn.nodemanager.log.retain-seconds</name>
		<value>604800</value><!--7 day-->
	</property>
	<!-- 指定zk集群地址 -->
	<property>
		<name>yarn.resourcemanager.zk-address</name>
		<value>node01:2181,node02:2181,node03:2181</value>
	</property>
  <!-- 逗号隔开的服务列表，列表名称应该只包含a-zA-Z0-9_,不能以数字开始-->
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
</configuration>
```



### 修改slaves

node01执行

```txt
node01
node02
node03
```



### 远程拷贝hadoop文件夹

node01执行

```shell
scp -r /bigdata/install/hadoop-2.6.0-cdh5.14.2-ha node02:/bigdata/install
scp -r /bigdata/install/hadoop-2.6.0-cdh5.14.2-ha node03:/bigdata/install
```



###修改两个RM的yarn-site.xml

<font color=red>node02执行</font>

```xml
<property>
                <name>yarn.resourcemanager.ha.id</name>
                <value>rm1</value>
                <description>If we want to launch more than one RM in single node, we need this configuration</description>
</property>
```



<font color=red>node03执行</font>

```xml
<property>
                <name>yarn.resourcemanager.ha.id</name>
                <value>rm2</value>
                <description>If we want to launch more than one RM in single node, we need this configuration</description>
</property>
```



### 配置环境变量

<font color=red>node01、node02和node03执行</font>

```shell
sudo vi /etc/profile

export HADOOP_HOME=/bigdata/install/hadoop-2.6.0-cdh5.14.2-ha/hadoop-2.6.0-cdh5.14.2
export HADOOP_CONF_DIR=/bigdata/install/hadoop-2.6.0-cdh5.14.2-ha/hadoop-2.6.0-cdh5.14.2/etc/hadoop

# 立即生效
source /etc/profile
```



# 启动集群

## 启动ZooKeeper集群

<font color=red>node01、node02和node03执行</font>

- QuorumPeerMain进程

```shell
cd /bigdata/install/zookeeper-3.4.5-cdh5.14.2

# 启动ZooKeeper
bin/zkServer.sh start

# 查看ZooKeeper运行状态，一个leader，两个follower
bin/zkServer.sh status
```



## 启动hdfs

### 格式化ZK

node01执行

- 集群ns1中有两个NameNode，其中node01上是active NameNode，node02上是standby NameNode
- 每个NameNode上都有一个zkfc进程，在active NameNode node01 上格式化zkfc

```shell
# 在ZooKeeper上创建 /hadoop-ha/ns1
bin/hdfs zkfc -formatZK
```



### 启动journalnode

node01执行

- 启动node01、node02和node03上的journalnode
- JournalNode进程

```shell
sbin/hadoop-daemons.sh start journalnode
```



### 格式化hdfs

node01执行

- 只在active NameNode node01上格式化hdfs；默认 `hadoop.tmp.dir` 指定位置存储fsimage

```shell
bin/hdfs namenode -format
```



### 初始化元数据、启动active NameNode

node01执行

- NameNode进程（node01）
- DataNode进程（node01、node02和node03）
- DFSZKFailoverController进程（node01和node02）

```shell
# 初始化元数据
bin/hdfs namenode -initializeSharedEdits -force
# 启动hdfs集群
# 在ZooKeeper上创建 /hadoop-ha/ns1/ActiveBreadCrumb 持久节点
# 在ZooKeeper上创建 /hadoop-ha/ns1/ActiveStandbyElectorLock 临时节点
sbin/start-dfs.sh
```



### 同步元数据信息、启动standby NameNode

node02执行

- NameNode进程（node02）

```shell
# 同步元数据信息(拉取fsimage)
bin/hdfs namenode -bootstrapStandby
# 启动NameNode，并设置standby状态
sbin/hadoop-daemon.sh start namenode
```



## 启动yarn

### 启动active ResourceManager

node02执行

- <font color=red>node02启动yarn之前，需要ssh分别登陆node01、node02和node03（首次登陆需要交互）</font>
- ResourceManager进程（node02）
- NodeManager（node01、node02和node03）

```shell
sbin/start-yarn.sh
```



### 启动standby ResourceManager

node03执行

- ResourceManager进程（node03）

```shell
sbin/yarn-daemon.sh start resourcemanager
```



### 查看ResourceManager状态

在集群任意节点运行命令

```shell
# active
bin/yarn rmadmin -getServiceState rm1
# standby
bin/yarn rmadmin -getServiceState rm2
```



## 启动JobHistory

node03执行

- JobHistoryServer进程

```shell
sbin/mr-jobhistory-daemon.sh start historyserver
```



# 验证集群

## 验证hdfs ha

### 访问web ui

- http://node01:50070/dfshealth.html#tab-overview
- http://node02:50070/dfshealth.html#tab-overview



### 模拟主备切换

```shell
# node01执行，之后node02切换为active状态
sbin/hadoop-daemon.sh stop namenode

# 集群任意节点检查nn2状态
bin/hdfs haadmin -getServiceState nn2

# node01执行，之后node01切换为standby状态
sbin/hadoop-daemon.sh start namenode
```



### HDFS Client API验证

- 初始化目录修改访问权限

  ```shell
  hdfs dfs -mkdir /hdfstest
  hdfs dfs -chmod 777 /hdfstest
  ```

- Java API 测试故障转移

  ```java
  public class HATests {
      private Configuration configuration;
  
      @Before
      public void init() {
          configuration = new Configuration();
          // namespace基础配置
          configuration.set("dfs.nameservices","ns1");
          configuration.set("dfs.ha.namenodes.ns1","nn1,nn2");
          configuration.set("dfs.namenode.rpc-address.ns1.nn1","node01:8020");
          configuration.set("dfs.namenode.rpc-address.ns1.nn2","node02:8020");
          configuration.set("dfs.client.failover.proxy.provider.ns1","org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider");    // 必须配置
  
          // 需要访问哪一个namespace
          configuration.set("fs.defaultFS", "hdfs://ns1");
      }
  
      @Test
      public void testPut() throws IOException {
          FileSystem fileSystem = FileSystem.get(configuration);
          fileSystem.copyFromLocalFile(new Path("/Users/yangxiaoyu/work/test/hdfsdatas/hello"), new Path("/hdfstest/hello"));
          fileSystem.close();
      }
  
      @Test
      public void testList() throws IOException {
          FileSystem fileSystem = FileSystem.get(configuration);
  
          RemoteIterator<LocatedFileStatus> remoteIterator = fileSystem.listFiles(new Path("/hdfstest"), true);
  
          while (remoteIterator.hasNext()) {
              LocatedFileStatus next = remoteIterator.next();
              System.out.println(next.getPath().getName());
          }
      }
  }
  ```

  

## 验证yarn ha

### 访问web ui

- http://node02:8088/cluster/cluster
- http://node03:8088/cluster/cluster



### 模拟主备切换

```shell
# node02执行，之后node03切换为active状态
sbin/yarn-daemon.sh stop resourcemanager

# 集群任意节点检查rm2状态
bin/yarn rmadmin -getServiceState rm2

# node02执行，之后node02切换为standby状态
sbin/yarn-daemon.sh start resourcemanager
```



### MapReduce示例验证

```shell
# 上传测试文件
bin/hdfs dfs -mkdir /mrtest
bin/hadoop fs -put /bigdata/install/hadoop-2.6.0-cdh5.14.2-ha/hadoop-2.6.0-cdh5.14.2/LICENSE.txt /t
# 运行单词统计
# 显示Failing over to rm2
bin/hadoop jar /bigdata/install/hadoop-2.6.0-cdh5.14.2-ha/hadoop-2.6.0-cdh5.14.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0-cdh5.14.2.jar wordcount /mrtest/LICENSE.txt /mrtest/output
```



# 关闭集群

``` shell
# active NameNode运行
sbin/stop-dfs.sh
# active ResourceManager运行
sbin/stop-yarn.sh
# standby ResourceManager运行
sbin/yarn-daemon.sh stop resourcemanager
# node03运行
sbin/mr-jobhistory-daemon.sh stop historyserver
# node01，node02和node03分别运行
bin/zkServer.sh stop
```

