# GitHub空项目

在GitHub上创建空项目，如：hadoop-main



## IDEA创建project

- check out from version control | 选择 git

- 指定URL [`https://github.com/sciatta/hadoop-main.git`](https://github.com/sciatta/hadoop-main.git) 和本地目录 | clone | 选择 create project from existing sources

- 后续默认选择

- 初始化project `git flow init` 后续在develop分支开发



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

