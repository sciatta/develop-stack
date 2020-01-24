# 环境准备

准备三台虚拟机

## ip设置

```bash
vi /etc/sysconfig/network-scripts/ifcfg-ens33

BOOTPROTO="static"
IPADDR=192.168.2.100
NETMASK=255.255.255.0
GATEWAY=192.168.2.2
DNS1=192.168.2.2
```

准备三台linux机器，IP地址分别设置成为

第一台机器IP地址：192.168.2.100

第二台机器IP地址：192.168.2.101

第三台机器IP地址：192.168.2.102



## 关闭防火墙

root用户下执行

```bash
systemctl stop firewalld
systemctl disable firewalld
```



## 关闭selinux

root用户下执行

```bash
vi /etc/selinux/config

SELINUX=disabled
```



## 更改主机名

```bash
vi /etc/hostname

node01
```

第一台主机名更改为：node01

第二台主机名更改为：node02

第三台主机名更改为：node03



## 更改主机名与IP地址映射

```bash
vi /etc/hosts

192.168.2.100 node01
192.168.2.101 node02
192.168.2.102 node03
```



## 同步时间

定时同步阿里云服务器时间

```bash
yum -y install ntpdate
crontab -e

*/1 * * * * /usr/sbin/ntpdate time1.aliyun.com
```



## 添加用户

三台linux服务器统一添加普通用户hadoop，并给以sudo权限，用于以后所有的大数据软件的安装

并统一设置普通用户的密码为 hadoop

```bash
useradd hadoop
passwd hadoop
```



为普通用户添加sudo权限

```bash
visudo

hadoop  ALL=(ALL)       ALL
```



## 定义统一目录

定义三台linux服务器软件压缩包存放目录，以及解压后安装目录，三台机器执行以下命令，创建两个文件夹，一个用于存放软件压缩包目录，一个用于存放解压后目录

```bash
# root 用户执行
mkdir -p /bigdata/soft # 软件压缩包存放目录
mkdir -p /bigdata/install # 软件解压后存放目录
chown -R hadoop:hadoop /bigdata # 将文件夹权限更改为hadoop用户
```



## 安装JDK

<font color=red>使用hadoop用户来重新连接三台机器，然后使用hadoop用户来安装jdk软件。</font>上传压缩包到第一台服务器的/bigdata/soft下面，然后进行解压，配置环境变量，三台机器都依次安装

```bash
# 分别上传jdk文件
scp jdk-8u141-linux-x64.tar.gz hadoop@192.168.2.100:/bigdata/soft   
scp jdk-8u141-linux-x64.tar.gz hadoop@192.168.2.101:/bigdata/soft
scp jdk-8u141-linux-x64.tar.gz hadoop@192.168.2.102:/bigdata/soft

cd /bigdata/soft/
tar -zxf jdk-8u141-linux-x64.tar.gz  -C /bigdata/install/
sudo vi /etc/profile

#添加以下配置内容，配置jdk环境变量
export JAVA_HOME=/bigdata/install/jdk1.8.0_141
export PATH=:$JAVA_HOME/bin:$PATH

# 立即生效
source /etc/profile
```



## hadoop用户免密码登

三台机器在hadoop用户下执行以下命令生成公钥与私钥

```bash
# 三台机器在hadoop用户下分别执行
ssh-keygen -t rsa
# 三台机器拷贝公钥到node01
ssh-copy-id node01

# 在node01的hadoop用户下执行，将authorized_keys拷贝到node02和node03
# node01已经存在authorized_keys
cd /home/hadoop/.ssh/
scp authorized_keys node02:$PWD
scp authorized_keys node03:$PWD
```



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



## 启动zookeeper服务

三台机器分别执行

```bash
/bigdata/install/zookeeper-3.4.5-cdh5.14.2/bin/zkServer.sh start

# 查看启动状态
/bigdata/install/zookeeper-3.4.5-cdh5.14.2/bin/zkServer.sh status
```



# 安装hadoop

## CDH软件版本重新进行编译

### 为何编译

CDH和Apache发布包不支持C程序库。本地库可以用于支持压缩算法和c程序调用。



### 安装JDK

<font color=red>需要版本jdk1.7.0_80（1.8编译会出现错误）</font>



### 安装maven

```bash
# 解压缩
tar -zxvf apache-maven-3.0.5-bin.tar.gz -C ../install/

# 配置环境变量
sudo vi /etc/profile
export MAVEN_HOME=/bigdata/install/apache-maven-3.0.5
export MAVEN_OPTS="-Xms4096m -Xmx4096m"
export PATH=:$MAVEN_HOME/bin:$PATH

# 立即生效
source /etc/profile
```



### 安装findbugs

```bash
# 安装wget
sudo yum install -y wget

# 下载findbugs
cd /bigdata/soft
wget --no-check-certificate https://sourceforge.net/projects/findbugs/files/findbugs/1.3.9/findbugs-1.3.9.tar.gz/download -O findbugs-1.3.9.tar.gz

# 解压findbugs
tar -zxvf findbugs-1.3.9.tar.gz -C ../install/

# 配置环境变量
sudo vi /etc/profile
export FINDBUGS_HOME=/bigdata/install/findbugs-1.3.9
export PATH=:$FINDBUGS_HOME/bin:$PATH

# 立即生效
source /etc/profile
```



### 安装依赖

```bash
sudo yum install -y autoconf automake libtool cmake
sudo yum install -y ncurses-devel
sudo yum install -y openssl-devel
sudo yum install -y lzo-devel zlib-devel gcc gcc-c++
sudo yum install -y bzip2-devel
```



### 安装protobuf

```bash
# 解压缩
tar -zxvf protobuf-2.5.0.tar.gz -C ../install/

# 执行
cd /bigdata/install/protobuf-2.5.0
./configure

# root 用户执行
make && make install
```



### 安装snappy

```bash
# 解压缩
cd /bigdata/soft/
tar -zxf snappy-1.1.1.tar.gz -C ../install/

# 执行
cd ../install/snappy-1.1.1/
./configure

# root 用户执行
make && make install
```



### 编译

```bash
# 解压缩
tar -zxvf hadoop-2.6.0-cdh5.14.2-src.tar.gz -C ../install/

# 编译
cd /bigdata/install/hadoop-2.6.0-cdh5.14.2

# 编译不支持snappy压缩
mvn package -Pdist,native -DskipTests -Dtar

# 编译支持snappy压缩
mvn package -DskipTests -Pdist,native -Dtar -Drequire.snappy -e -X
```



## 安装hadoop集群

安装环境服务部署规划

| 服务器IP      | HDFS     | HDFS              | HDFS     | YARN            | YARN        | 历史日志服务器   |
| ------------- | -------- | ----------------- | -------- | --------------- | ----------- | ---------------- |
| 192.168.2.100 | NameNode | SecondaryNameNode | DataNode | ResourceManager | NodeManager | JobHistoryServer |
| 192.168.2.101 |          |                   | DataNode |                 | NodeManager |                  |
| 192.168.2.102 |          |                   | DataNode |                 | NodeManager |                  |



### 上传压缩包并解压

重新编译之后支持snappy压缩的hadoop包上传到第一台服务器并解压

主机执行

```bash
scp hadoop-2.6.0-cdh5.14.2_after_compile.tar.gz hadoop@192.168.2.100:/bigdata/soft
```



node01执行

```bash
cd /bigdata/soft/
tar -zxvf hadoop-2.6.0-cdh5.14.2_after_compile.tar.gz -C ../install/
```



### 查看hadoop支持的压缩方式以及本地库

node01执行

```bash
cd /bigdata/install/hadoop-2.6.0-cdh5.14.2
bin/hadoop checknative
```



如果出现openssl为false，那么所有机器在线安装openssl

```bash
su root
yum -y install openssl-devel
```



### 修改配置文件

- 修改core-site.xml

node01 执行

```bash
cd /bigdata/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop
vi core-site.xml

<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://node01:8020</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/bigdata/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/tempDatas</value>
    </property>
    <!-- 缓冲区大小，实际工作中根据服务器性能动态调整 -->
    <property>
        <name>io.file.buffer.size</name>
        <value>4096</value>
    </property>
    <!-- 开启hdfs的垃圾桶机制，删除掉的数据可以从垃圾桶中回收，单位分钟 -->
    <property>
        <name>fs.trash.interval</name>
        <value>10080</value>
    </property>
</configuration>
```



- 修改hdfs-site.xml

node01 执行

```bash
cd /bigdata/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop
vi hdfs-site.xml

<configuration>
    <!-- 集群动态上下线 <property>
        <name>dfs.hosts</name>
        <value>/bigdata/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop/accept_host</value>
    </property>
    <property>
        <name>dfs.hosts.exclude</name>
        <value>/bigdata/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop/deny_host</value>
    </property>
     -->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>node01:50090</value>
    </property>
    <property>
        <name>dfs.namenode.http-address</name>
        <value>node01:50070</value>
    </property>
    <!-- NameNode存储元数据信息的路径，实际工作中，一般先确定磁盘的挂载目录，然后多个目录用，进行分割 -->
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///bigdata/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/namenodeDatas</value>
    </property>
    <!-- DataNode数据存储的节点位置，实际工作中，一般先确定磁盘的挂载目录，然后多个目录用，进行分割 -->
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///bigdata/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/datanodeDatas</value>
    </property>
    <property>
        <name>dfs.namenode.edits.dir</name>
        <value>file:///bigdata/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/dfs/nn/edits</value>
    </property>
    <property>
        <name>dfs.namenode.checkpoint.dir</name>
        <value>file:///bigdata/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/dfs/snn/name</value>
    </property>
    <property>
        <name>dfs.namenode.checkpoint.edits.dir</name>
        <value>file:///bigdata/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/dfs/nn/snn/edits</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>dfs.permissions</name>
        <value>false</value>
    </property>
    <property>
        <name>dfs.blocksize</name>
        <value>134217728</value>
    </property>
</configuration>
```



- 修改hadoop-env.sh

node01 执行

```bash
cd /bigdata/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop
vi hadoop-env.sh

export JAVA_HOME=/bigdata/install/jdk1.8.0_141
```



- 修改mapred-site.xml

node01 执行

```bash
cd /bigdata/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop
vi mapred-site.xml

<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.job.ubertask.enable</name>
        <value>true</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>node01:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>node01:19888</value>
    </property>
</configuration>
```



- 修改yarn-site.xml

node01 执行

```bash
cd /bigdata/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop
vi yarn-site.xml

<configuration>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>node01</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```



- 修改slaves文件

node01 执行

```bash
cd /bigdata/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop
vi slaves

node01
node02
node03
```



### 创建文件存放目录

node01 执行

```bash
mkdir -p /bigdata/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/tempDatas
mkdir -p /bigdata/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/namenodeDatas
mkdir -p /bigdata/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/datanodeDatas
mkdir -p /bigdata/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/dfs/nn/edits
mkdir -p /bigdata/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/dfs/snn/name
mkdir -p /bigdata/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/dfs/nn/snn/edits
```



### 安装包分发

node01 执行

```bash
cd /bigdata/install/

scp -r hadoop-2.6.0-cdh5.14.2/ node02:$PWD
scp -r hadoop-2.6.0-cdh5.14.2/ node03:$PWD
```



### 配置hadoop的环境变量

三台机器执行

```bash
sudo vi /etc/profile

export HADOOP_HOME=/bigdata/install/hadoop-2.6.0-cdh5.14.2
export PATH=:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH

配置完成之后生效
source /etc/profile
```



### 集群启动

要启动 Hadoop 集群，需要启动 HDFS 和 YARN 两个集群。

<font color=red>注意：首次启动HDFS时，必须对其进行格式化操作。本质上是一些清理和准备工作，因为此时的 HDFS 在物理上还是不存在的。</font>

node01 执行

```bash
hdfs namenode -format 或者 hadoop namenode –format
```



- 单个节点逐一启动

```bash
# 在主节点上使用以下命令启动 HDFS NameNode: 
hadoop-daemon.sh start namenode

# 在每个从节点上使用以下命令启动 HDFS DataNode: 
hadoop-daemon.sh start datanode

# 在主节点上使用以下命令启动 YARN ResourceManager: 
yarn-daemon.sh start resourcemanager

# 在每个从节点上使用以下命令启动 YARN nodemanager: 
yarn-daemon.sh start nodemanager

# 以上脚本位于$HADOOP_PREFIX/sbin/目录下
# 如果想要停止某个节点上某个角色，只需要把命令中的start改为stop即可
```



- 脚本一键启动

如果配置了 etc/hadoop/slaves 和 ssh 免密登录，则可以使用程序脚本启动所有Hadoop两个集群的相关进程，在主节点所设定的机器上执行。

node01 执行

启动集群

```bash
start-dfs.sh
start-yarn.sh
mr-jobhistory-daemon.sh start historyserver
```



停止集群

```bash
stop-dfs.sh
stop-yarn.sh
mr-jobhistory-daemon.sh stop historyserver
```



### 浏览器查看启动页面

hdfs集群访问地址 
http://192.168.2.100:50070/dfshealth.html#tab-overview 



yarn集群访问地址 
http://192.168.2.100:8088/cluster



jobhistory访问地址 
http://192.168.2.100:19888/jobhistory