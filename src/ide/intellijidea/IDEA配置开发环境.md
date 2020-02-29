# Java环境

## 配置JDK环境变量

```bash
vi ~/.zshrc

export SCALA_HOME=/Users/yangxiaoyu/work/install/scala-2.11.8
export PATH=$SCALA_HOME/bin:$PATH

# 即时生效
source ~/.zshrc
```



## 配置JDK集成开发环境

- 项目右键 |  open module settings | sdks | + jdk 指定 jdk home path

- 项目右键 |  open module settings | project 指定sdk

- 项目右键 |  open module settings | modules | language level: 8

- Preferences | Build, Execution, Deployment | compiler | java compiler
  1. module 设置为8
  2. project bytecode version 设置为8



# Scala环境

## 配置SDK环境变量

```bash
tar -zxvf scala-2.11.8.tgz -C ../install

vi ~/.zshrc

export SCALA_HOME=/Users/yangxiaoyu/work/install/scala-2.11.8
export PATH=$SCALA_HOME/bin:$PATH

# 即时生效
source ~/.zshrc
```



## 配置SDK集成开发环境

### 下载 Scala 插件

- Preferences | plugins | Scala



### 新建 pom module



### 新建 jar module



### 指定SDK

- File | Project Structure | Libraries | + | Scala SDK | 选择 scala-2.11.8
- 项目右键 |  open module settings | modules | hadoop-scala-example | Dependencies | + | Libraries | 选择 scala-sdk-2.11.8



### 混编Scala

- src/main 新建 Directory | scala
- Mark Directory as | Sources Root



### 打包运行

- maven | package
- scala -classpath /Users/yangxiaoyu/Desktop/hadoop-scala-example-1.0-SNAPSHOT.jar First



### 下载 Scala 反编译插件

Preferences | plugins | CFR Decompile