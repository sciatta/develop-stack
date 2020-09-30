# 概述

Azkaban是由Linkedin开源的一个批量工作流任务调度器。用于在一个工作流内以一个特定的顺序运行一组工作和流程。Azkaban定义了一种KV文件（properties）格式来建立任务之间的依赖关系，并提供一个易于使用的web用户界面维护和跟踪工作流。

- 功能

  - 兼容任何版本的Hadoop

    在编译的时候，只需要指定Hadoop的版本

  - 易于使用的Web UI

  - 简单的web和http工作流上传

    只需要预先将workflow定义好以后，就可以通过浏览器把我们需要的job的配置文件传到Azkaban的web server

  - 项目工作区

    不同的项目可以归属于不同的空间，而且不同的空间又可以设置不同的权限。多个项目之间是不会产生任何的影响与干扰

  - 工作流调度

    可以手工运行，也可以定时处理

  - 模块化和可插拔的插件机制

  - 认证和授权

  - 跟踪用户的行为

  - 发送失败和成功的电子邮件警报

  - SLA警告

  - 失败作业的重试机制

 # 基本架构

- Azkaban Web Server

  提供了Web UI，是azkaban的主要管理者，包括 project 的管理，认证，调度，对工作流执行过程的监控等。

- Azkaban Executor Server

  负责具体的工作流和任务的调度提交。

- Mysql

  用于保存项目、日志或者执行计划之类的信息。

# 运行模式

- solo server mode

  web server 和 executor server运行在一个进程。最简单的模式，数据库内置的H2数据库，管理服务器和执行服务器都在一个进程中运行，任务量不大项目可以采用此模式。

- two server mode

  web server 和 executor server运行在不同的进程。数据库为mysql，管理服务器和执行服务器在不同进程，这种模式下，管理服务器和执行服务器互不影响。

- multiple executor mode

  web server 和 executor server运行在不同的进程，executor server有多个。该模式下，执行服务器和管理服务器在不同主机上，且执行服务器可以有多个。

# 示例

Azkaba内置的任务类型支持command、java。

## Command类型单一job

- 创建job

  ```shell
  # 创建job文件
  touch mycommand.job
  vi mycommand.job
  
  type=command
  command=echo 'hello azkaban'
  
  # 创建zip
  zip -r -q mycommand.zip *
  ```

- 创建 `command` project

- 上传 mycommand.zip（必须上传zip文件）

- 执行 Execute Flow

## Command类型多个job

- 创建job

  ```shell
  # 创建job文件
  touch start.job
  touch task1.job
  touch task2.job
  touch stop.job
  
  vi start.job
  type=command
  command=echo '========start========'
  
  vi task1.job
  type=command
  dependencies=start
  command=echo '========task1========'
  
  vi task2.job
  type=command
  dependencies=start
  command=echo "========task2========"
  
  vi stop.job
  type=command
  dependencies=task1,task2
  command=echo "========stop========"
  
  # 创建zip
  zip -r -q dependencies.zip *
  ```

- 创建 `dependencies` project

- 上传 dependencies.zip

- 执行

## HDFS任务

- 创建job

  ```shell
  # 创建job文件
  touch fs.job
  vi fs.job
  
  type=command
  command=echo "start execute"
  command.1=/bigdata/install/hadoop-2.6.0-cdh5.14.2/bin/hdfs dfs -mkdir /test/azkaban
  command.2=/bigdata/install/hadoop-2.6.0-cdh5.14.2/bin/hdfs dfs -put /etc/profile /test/azkaban
  
  # 创建zip
  zip -r -q fs.zip *
  ```

- 创建 `fs` project

- 上传 fs.zip

- 执行

## MapReduce任务

- 创建job

  ```shell
  # 创建job文件
  touch mr.job
  vi mr.job
  
  type=command
  command=/bigdata/install/hadoop-2.6.0-cdh5.14.2/bin/hadoop jar /bigdata/install/hadoop-2.6.0-cdh5.14.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0-cdh5.14.2.jar pi 3 5
  
  # 创建zip
  zip -rq mr.zip *
  ```

- 创建 `mr` project

- 上传 mr.zip

- 执行

## Hive任务

- 创建job

  - sql脚本hive.sql

    ```mysql
    create database if not exists azhive;
    use azhive;
    create table if not exists aztest(id string,name string) row format delimited fields terminated by '\t';
    ```

  - job文件hive.job

    ```shell
    type=command
    command=/bigdata/install/hive-1.1.0-cdh5.14.2/bin/hive -f 'hive.sql'
    ```

  - zip文件

    ```shell
    zip -rq hive.zip *
    ```


- 创建 `hive` project
- 上传 hive.zip
- 执行

## 定时任务

使用azkaban的scheduler功能可以实现对我们的作业任务进行定时调度功能。

- Execute Flow

- Schedule

  - 选项

  | 字段         | 强制性 | 范围                                 | 特殊字符                                                     |
  | ------------ | ------ | ------------------------------------ | ------------------------------------------------------------ |
  | Min          | 可选   | 0-59                                 | * 任意匹配值<br />, 分隔<br />- 范围值<br />/ 单位间隔       |
  | Hours        | 必须   | 0-23                                 | * 任意匹配值<br />, 分隔<br />- 范围值<br />/ 单位间隔       |
  | Day of Month | yes    | 1-31                                 | * 任意匹配值<br />, 分隔<br />- 范围值<br />/ 单位间隔<br />? 为空值，当两个相似概念的字段中，只有一个字段起作用，另一个不起作用，可以使用?，同Day of Week搭配使用 |
  | Month        | yes    | 1-12                                 | * 任意匹配值<br />, 分隔<br />- 范围值<br />/ 单位间隔       |
  | Day of Week  | yes    | 1-7<br />SUN MON TUE WED THU FRI SAT | * 任意匹配值<br />, 分隔<br />- 范围值<br />/ 单位间隔<br />? 为空值 |
  | Year         | no     |                                      | * 任意匹配值<br />, 分隔<br />- 范围值<br />/ 单位间隔       |

  - 举例
    - */1 * ? * *

      每分钟执行一次定时调度任务

    - 0 1 ? * *

      每天晚上凌晨一点执行调度任务

    - 0 */2 ? * *

      每隔两个小时定时执行调度任务

    - 30 21 ? * *

      每天晚上九点半定时执行调度任务

## WebUI参数传递

- 创建job

  ```shell
  vi param.job
  
  type=command
  # ${param}解析页面传递的参数
  # parameter声明一个变量
  myparam=${param}
  command=echo ${myparam}
  ```

- 创建 `param` project
- 上传 param.zip
- 执行
  - flow parameters，添加传入的参数 name=param，value=lucky coming...

