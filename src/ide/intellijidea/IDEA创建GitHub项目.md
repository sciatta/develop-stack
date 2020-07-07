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



# GitHub已有项目

## GitHub下载源码

```shell
# 下载源码
git clone https://github.com/sciatta/hadoop.git

# 切换到目标版本
cd hadoop
git checkout -b work-2.7.0 release-2.7.0
```



## 导入GitHub项目

`open | 指定项目路径`

加载即可。如果是maven项目，则等待更新依赖，成功后会正确显示maven项目。