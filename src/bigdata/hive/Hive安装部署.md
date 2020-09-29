# 先决条件

- hive是一个构建数据仓库的工具，只需要在一台服务器上安装，不需要在多台服务器上安装。
- 使用hadoop普通用户在node03上安装
- 搭建好三节点Hadoop集群
- node03上安装MySQL服务



# 安装

```bash
# 拷贝到node03上
scp hive-1.1.0-cdh5.14.2.tar.gz hadoop@192.168.2.102:/bigdata/soft

# 解压
cd /bigdata/soft
tar -xzvf hive-1.1.0-cdh5.14.2.tar.gz -C /bigdata/install/
```



# 修改配置

## hive-env.sh

```bash
cd /bigdata/install/hive-1.1.0-cdh5.14.2/conf
mv hive-env.sh.template hive-env.sh
vi hive-env.sh 
```



修改内容

```shell
# 配置HADOOP_HOME路径
export HADOOP_HOME=/bigdata/install/hadoop-2.6.0-cdh5.14.2/

# 配置HIVE_CONF_DIR路径
export HIVE_CONF_DIR=/bigdata/install/hive-1.1.0-cdh5.14.2/conf
```



## hive-site.xml

```bash
vi hive-site.xml
```



修改内容

```xml
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
        <property>
                <name>javax.jdo.option.ConnectionURL</name>
                <value>jdbc:mysql://node03:3306/hive?createDatabaseIfNotExist=true&amp;characterEncoding=latin1&amp;useSSL=false</value>
        </property>
        <property>
                <name>javax.jdo.option.ConnectionDriverName</name>
                <value>com.mysql.jdbc.Driver</value>
        </property>
        <property>
                <name>javax.jdo.option.ConnectionUserName</name>
                <value>root</value>
        </property>
        <property>
                <name>javax.jdo.option.ConnectionPassword</name>
                <value>root</value>
        </property>
        <property>
                <name>hive.cli.print.current.db</name>
                <value>true</value>
        </property>
        <property>
                <name>hive.cli.print.header</name>
            	<value>true</value>
        </property>
    	<property>
                <name>hive.server2.thrift.bind.host</name>
                <value>node03</value>
        </property>
</configuration>
```



## hive-log4j.properties

```bash
# 创建hive日志存储目录
mkdir -p /bigdata/install/hive-1.1.0-cdh5.14.2/logs/

cd /bigdata/install/hive-1.1.0-cdh5.14.2/conf
mv hive-log4j.properties.template hive-log4j.properties
vi hive-log4j.properties
```



修改内容

```properties
hive.log.dir=/bigdata/install/hive-1.1.0-cdh5.14.2/logs/
```



# 拷贝mysql驱动包

```bash
scp mysql-connector-java-5.1.38.jar hadoop@192.168.2.102:/bigdata/soft

# 由于运行hive时，需要向mysql数据库中读写元数据，所以需要将mysql的驱动包上传到hive的lib目录下
cp mysql-connector-java-5.1.38.jar /bigdata/install/hive-1.1.0-cdh5.14.2/lib/
```



# 配置环境变量

root用户下执行

```bash
su root
vi /etc/profile
```



修改内容

```properties
export HIVE_HOME=/bigdata/install/hive-1.1.0-cdh5.14.2
export PATH=$PATH:$HIVE_HOME/bin
```



切换回hadoop

```bash
su hadoop
source /etc/profile
```



# 验证安装

Node03执行

<font color=red>执行前需要先启动hadoop集群和mysql数据库服务</font>

```bash
# 启动hive cli命令行客户端
hive
```



查看数据库

```mysql
show databases;
```



退出

```mysql
quit;
```

