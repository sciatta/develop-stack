# GitHub空项目

## GitHub上创建空项目

如：hadoop-main



## IDEA创建project

- check out from version control | 选择 git

- 指定URL [`https://github.com/sciatta/hadoop-main.git`](https://github.com/sciatta/hadoop-main.git) 和本地目录 | clone | 选择 create project from existing sources

- 后续默认选择




## IDEA创建module

- 创建 parent pom module

hadoop-main | new module | maven

```bash
groupid: com.sciatta.hadoop
artifact: hadoop-main

module name: hadoop-main
# hadoop-main根目录作为module parent
content root: /Users/yangxiaoyu/work/bigdata/project/hadoop-main

# 修改pom.xml
<packaging>pom</packaging>
```



- 创建child pom module

hadoop-main | new module | maven

```bash
parent: hadoop-main
add as module to: hadoop-main
artifact: hadoop-hdfs

module name: hadoop-hdfs
content root: /Users/yangxiaoyu/work/bigdata/project/hadoop-main/hadoop-hdfs

# 修改pom.xml
<packaging>pom</packaging>
```



- 创建child jar module

hadoop-hdfs | new module | maven

```bash
parent: hadoop-hdfs
add as module to: hadoop-hdfs
artifact: hadoop-hdfs-example

module name: hadoop-hdfs-example
content root: /Users/yangxiaoyu/work/bigdata/project/hadoop-main/hadoop-hdfs/hadoop-hdfs-example

# 修改pom.xml
<packaging>jar</packaging>
```



## IDEA删除module

- 右键 | remove module

- 右键 | delete module

- 在父pom中删除module

- Preferences | Build, Execution, Deployment | Build Tools | Maven | Ignored Files

  反向勾选被选中的pom.xml

- Reload project



# GitHub已有项目

## HTTPS

### GitHub下载源码

```shell
# 下载源码
git clone https://github.com/sciatta/hadoop.git

# 切换到目标版本
cd hadoop
git checkout -b work-2.7.0 release-2.7.0
```

### 导入GitHub项目

`open | 指定项目路径`

加载即可。如果是maven项目，则等待更新依赖，成功后会正确显示maven项目。



## SSH

### 设置用户信息

```shell
git config --global user.name "yangxiaoyu"
git config --global user.email 15058553030@163.com

# 查看设置是否生效
git config --list
```



### 创建 SSH KEY

```shell
ssh-keygen -t rsa -C "15058553030@163.com"

# 拷贝公钥 id_rsa.pub 内容
cd /Users/yangxiaoyu/.ssh
```



### GITHUB 配置

Settings | SSH and GPG keys | New SSH key | 

- Title: DEV

- Key: id_rsa.pub 内容

```shell
# 验证是否配置成功
# Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
ssh -T git@github.com
```



### 克隆仓库

```shell
git clone git@github.com:sciatta/JAVA-000.git
```



### 创建Project

Create New Project | Maven

- Name: JAVA-000
- Location: ~/work/bigdata/project/JAVA-000

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.sciatta</groupId>
    <artifactId>java-000</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

</project>
```



### 创建Module

 JAVA-000 右键 | New Module

- Parent: java-000
- Name: week-01
- Location: ~/work/bigdata/project/JAVA-000/Week_01

