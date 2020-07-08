# 编译源码

在Mac OS环境下编译hadoop2.7.0

## 下载源码

```shell
git clone https://github.com/sciatta/hadoop.git
```



## 切换到目标版本

```shell
cd hadoop
git checkout -b work-2.7.0 release-2.7.0
```



## 安装依赖

### JDK

```shell
# 安装
jdk-8u231-macosx-x64.dmg

# 配置环境变量
# Mac使用的是iTerm2
vi ~/.zshrc

export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH

# 立即生效
source ~/.zshrc
```

### Maven

```shell
# 安装
apache-maven-3.0.5-bin.tar.gz

tar -zxvf apache-maven-3.0.5-bin.tar.gz -C ../install/

# 配置环境变量
export MAVEN_HOME=/Users/yangxiaoyu/work/install/apache-maven-3.0.5
export PATH=$MAVEN_HOME/bin:$PATH

# 立即生效
source ~/.zshrc

# 修改镜像地址
cd /Users/yangxiaoyu/work/install/apache-maven-3.0.5/conf
vi settings.xml
# 增加
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

### ProtocolBuffer 2.5.0

```shell
# 安装
protobuf-2.5.0.tar.gz

tar -zxvf protobuf-2.5.0.tar.gz -C ../install/

# 执行
cd /Users/yangxiaoyu/work/install/protobuf-2.5.0
./configure

# 编译
make && make install

# 验证安装情况
# libprotoc 2.5.0
protoc --version
```

### cmake

```shell
# 安装
brew install cmake
```

### openssl

```shell
# 安装
brew install openssl

# 配置环境变量
# hadoop2.7.0不支持高版本openssl@1.1/1.1.1g
export OPENSSL_ROOT_DIR=/usr/local/Cellar/openssl/1.0.2n
export OPENSSL_INCLUDE_DIR=/usr/local/Cellar/openssl/1.0.2n/include

# 立即生效
source ~/.zshrc
```



## 编译

```shell
# 编译
# -P 执行profile
mvn package -Pdist,native -DskipTests -Dtar

# 成功后生成文件
/Users/yangxiaoyu/work/bigdata/project/hadoop/hadoop-dist/target/hadoop-2.7.0.tar.gz

# 编译指定模块
mvn package -Pnative -DskipTests -pl hadoop-hdfs-project/hadoop-hdfs
```



# 项目配置

基于IEDA开发环境

## 导入项目

`open | 打开hadoop项目目录` 成功后自动识别Maven模块。

## 修正项目

### 修正hadoop-streaming引用错误conf问题

修改hadoop-streaming模块

- 将hadoop-yarn-server-resourcemanager模块下的conf文件夹转移到hadoop-streaming模块下

- 修改pom.xml

  ```xml
  <testResource>
    <!-- 修改相对路径 -->
  	<directory>${basedir}/conf</directory>
  	<includes>
  		<include>capacity-scheduler.xml</include>
  	</includes>
  	<filtering>false</filtering>
  </testResource>
  ```



### 修正Debug无法找到类的问题

`Run | Edit Configurations... | Application | NameNode`

选择 Include dependencies with “Provided” scope



### 修正hadoop-hdfs模块无法正常打印log4j日志

target/classes目录下新建log4j.properties

```properties
log4j.rootLogger=info,stdout
log4j.threshhold=ALL
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{ISO8601} %-5p %c{2} (%F:%M(%L)) - %m%n
```



### 修正hadoop-hdfs模块自定义用户配置

- target/classes目录下新建hdfs-site.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
  
  <configuration>
      <property>
          <name>dfs.namenode.http-address</name>
          <value>localhost:50070</value>
          <description>
              The address and the base port where the dfs namenode web ui will listen on.
          </description>
      </property>
  </configuration>
  ```

- target/classes目录下新建core-site.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
  
  <configuration>
      <property>
          <name>fs.defaultFS</name>
          <value>hdfs://localhost:8020</value>
          <description>The name of the default file system.  A URI whose
              scheme and authority determine the FileSystem implementation.  The
              uri's scheme determines the config property (fs.SCHEME.impl) naming
              the FileSystem implementation class.  The uri's authority is used to
              determine the host, port, etc. for a filesystem.</description>
      </property>
  </configuration>
  ```



### 修正类路径无法找到webapps/hdfs

将src/main/webapps复制一份到target/classes目录



### 修正NameNodeRpcServer找不到类ClientNamenodeProtocol的问题

因为ClientNamenodeProtocol所在的类ClientNamenodeProtocolProtos过大，idea无法加载。

`Help | Edit Custom Properties... | Create`

idea.properties

```properties
idea.max.intellisense.filesize=5000
```

重启idea。



# NameNode

## 启动脚本

### hadoop-daemon.sh

`hadoop-daemon.sh start namenode` 在主节点启动NameNode

- 包含 `hadoop-config.sh` 导出环境配置变量
  - 包含 `hadoop-env.sh` 导出环境变量，如：JAVA_HOME，可以避免ssh登录后执行的JAVA_HOME不正确的问题
- 包含 `hadoop-env.sh` 导出环境变量
- 调用 `hdfs`

### start-dfs.sh

`start-dfs.sh` 在主节点启动NameNode和DataNode

- 调用 `hadoop-daemon.sh`

### hdfs

- 包含 `hadoop-config.sh` 导出环境配置变量
- 入口类 `org.apache.hadoop.hdfs.server.namenode.NameNode` 。运行主类，启动NameNode服务



## 启动流程

- 加载配置文件core-default.xml、 core-site.xml、hdfs-default.xml和hdfs-site.xml。其中，core-default.xml和hdfs-default.xml存在于项目的类路径下
- 启动HttpServer
- 加载元数据
- 启动RpcServer
- 启动FSNamesystem
  - 资源检查
  - 数据块报送



## 核心功能

### HttpServer

对外提供HTTPServer服务，用户可以通过浏览器访问元数据、文件和日志等。

![hdfs_namenode_httpserver](HDFS源码分析.assets/hdfs_namenode_httpserver.png)

### FSNamesystem

#### 目录结构



#### 格式化

![hdfs_namenode_fsnamesystem_format](HDFS源码分析.assets/hdfs_namenode_fsnamesystem_format.png)

#### 加载镜像

![hdfs_namenode_fsnamesystem_loadimage](HDFS源码分析.assets/hdfs_namenode_fsnamesystem_loadimage.png)

### RpcServer

#### 启动

![hdfs_namenode_rpcserver](HDFS源码分析.assets/hdfs_namenode_rpcserver.png)

#### 核心类结构

![hdfs_namenode_rpcserver_class](HDFS源码分析.assets/hdfs_namenode_rpcserver_class.png)







