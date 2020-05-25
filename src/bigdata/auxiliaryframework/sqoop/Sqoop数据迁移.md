# 概述

Sqoop是apache旗下的一款“Hadoop和关系数据库之间传输数据”的工具。

- 导入数据：将MySQL，Oracle导入数据到Hadoop的HDFS、HIVE、HBASE等数据存储系统。

- 导出数据：从Hadoop的文件系统中导出数据到关系数据库。

将导入和导出的命令翻译成MapReduce程序实现，在翻译出的MapReduce中主要是对inputformat和outputformat进行定制。

# Sqoop1和Sqoop2比较

| 比较 | Sqoop1                                                       | Sqoop2                                                       |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 版本 | 1.4.x                                                        | 1.99x                                                        |
| 架构 | 仅使用一个sqoop客户端                                        | 引入了sqoop server，对connector实现了集中的管理              |
| 部署 | 简单，需要root权限，connector必须符合JDBC模型                | 复杂，配置部署更加繁琐                                       |
| 使用 | 命令方式容易出错，格式紧耦合；无法支持所有数据类型；安全机制不够完善，需要指定数据库用户和密码 | 多种交互方式，REST API、 JAVA API、 WEB UI以及CLI控制台方式进行访问；connector集中管理，所有连接完善权限管理，connector规范化，仅负责数据的读写 |

# 数据导入

## 准备mysql数据

```mysql
CREATE DATABASE `userdb`;

USE `userdb`;

DROP TABLE IF EXISTS `emp`;

CREATE TABLE `emp`
(
    `id`          INT(11)            DEFAULT NULL,
    `name`        VARCHAR(100)       DEFAULT NULL,
    `deg`         VARCHAR(100)       DEFAULT NULL,
    `salary`      INT(11)            DEFAULT NULL,
    `dept`        VARCHAR(10)        DEFAULT NULL,
    `create_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `update_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    `is_delete`   BIGINT(20)         DEFAULT '1'
) ENGINE = INNODB
  DEFAULT CHARSET = latin1;
  
INSERT INTO `emp`(`id`, `name`, `deg`, `salary`, `dept`) VALUES (1201, 'gopal', 'manager', 50000, 'TP');
INSERT INTO `emp`(`id`, `name`, `deg`, `salary`, `dept`) VALUES (1202, 'manisha', 'Proof reader', 50000, 'TP');
INSERT INTO `emp`(`id`, `name`, `deg`, `salary`, `dept`) VALUES (1203, 'khalil', 'php dev', 30000, 'AC');
INSERT INTO `emp`(`id`, `name`, `deg`, `salary`, `dept`) VALUES (1204, 'prasanth', 'php dev', 30000, 'AC');
INSERT INTO `emp`(`id`, `name`, `deg`, `salary`, `dept`) VALUES (1205, 'kranthi', 'admin', 20000, 'TP');

DROP TABLE IF EXISTS `emp_add`;

CREATE TABLE `emp_add`
(
    `id`          INT(11)            DEFAULT NULL,
    `hno`         VARCHAR(100)       DEFAULT NULL,
    `street`      VARCHAR(100)       DEFAULT NULL,
    `city`        VARCHAR(100)       DEFAULT NULL,
    `create_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `update_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    `is_delete`   BIGINT(20)         DEFAULT '1'
) ENGINE = INNODB
  DEFAULT CHARSET = latin1;

INSERT INTO `emp_add`(`id`, `hno`, `street`, `city`)
VALUES (1201,  '288A',  'vgiri',  'jublee'),
       (1202,  '108I',  'aoc',  'sec-bad'),
       (1203,  '144Z',  'pgutta',  'hyd'),
       (1204,  '78B',  'old city',  'sec-bad'),
       (1205,  '720X',  'hitec',  'sec-bad');
       
DROP TABLE IF EXISTS `emp_conn`;

CREATE TABLE `emp_conn`
(
    `id`          INT(100)           DEFAULT NULL,
    `phno`        VARCHAR(100)       DEFAULT NULL,
    `email`       VARCHAR(100)       DEFAULT NULL,
    `create_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `update_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    `is_delete`   BIGINT(20)         DEFAULT '1'
) ENGINE = INNODB
  DEFAULT CHARSET = latin1;
  
INSERT INTO `emp_conn`(`id`, `phno`, `email`)
VALUES (1201, '2356742', 'gopal@tp.com'),
       (1202, '1661663', 'manisha@tp.com'),
       (1203, '8887776', 'khalil@ac.com'),
       (1204, '9988774', 'prasanth@ac.com'),
       (1205, '1231231', 'kranthi@tp.com');
```



## 导入表数据到HDFS

### 导入到默认目录

导入数据默认在HDFS的当前用户下，如 `/user/hadoop/emp/`

```shell
bin/sqoop import --connect jdbc:mysql://node03:3306/userdb --password root --username root --table emp --m 1
```

### 导入到指定目录

```shell
# --delete-target-dir 目标目录如果存在，则删除
# --target-dir 指定目标目录
bin/sqoop import  --connect jdbc:mysql://node03:3306/userdb --username root --password root --delete-target-dir --table emp  --target-dir /test/sqoop/userdb/emp --m 1
```

### 指定分隔符

```shell
# --fields-terminated-by 指定字段间分隔符；默认 ,
bin/sqoop import  --connect jdbc:mysql://node03:3306/userdb --username root --password root --delete-target-dir --table emp  --target-dir /test/sqoop/userdb/emp2 --m 1 --fields-terminated-by '\t'
```

### 指定数据子集

```shell
bin/sqoop import \
--connect jdbc:mysql://node03:3306/userdb \
--username root --password root --table emp_add \
--target-dir /test/sqoop/userdb/emp_add -m 1  --delete-target-dir \
--where "city = 'sec-bad'"
```

### 指定SQL语句

```shell
# 必须包含 $CONDITIONS
bin/sqoop import \
--connect jdbc:mysql://node03:3306/userdb \
--username root --password root \
--delete-target-dir -m 1 \
--query 'select phno from emp_conn where 1=1 and $CONDITIONS' \
--target-dir /test/sqoop/userdb/emp_conn
```

### 增量导入

#### 使用参数 --incremental、--check-column、--last-value

```shell
# 增量导入不可有 --delete-target-dir 参数
# 从1202的下一条记录开始导入
bin/sqoop import \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password root \
--table emp \
--incremental append \
--check-column id \
--last-value 1202  \
-m 1 \
--target-dir /test/sqoop/userdb/emp_increment_
```

#### 使用参数 -- where

``` shell
# 参数 --check-column 必须存在
bin/sqoop import \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password root \
--table emp \
--incremental append \
--check-column id \
--where "id > 1202" \
-m 1 \
--target-dir /test/sqoop/userdb/emp_increment2_
```



## 导入表数据到Hive

### 导入到指定Hive表

创建Hive数据库和表

```mysql
create database sqooptohive;
use sqooptohive;

create external table emp_hive(id int,name string,deg string,salary int ,dept string) 
row format delimited fields terminated by '\001';
```

导入

```shell
bin/sqoop import --connect jdbc:mysql://node03:3306/userdb --username root --password root --table emp --fields-terminated-by '\001' --hive-import --hive-table sqooptohive.emp_hive --hive-overwrite --delete-target-dir -m 1
```

### 自动创建Hive表

```shell
bin/sqoop import --connect jdbc:mysql://node03:3306/userdb --username root --password root --table emp_conn --hive-import -m 1 --hive-database sqooptohive;
```



## 导入表数据到HBase

```shell
bin/sqoop import \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password root \
--table emp \
--columns "id,name,deg,salary,dept,create_time,update_time,is_delete" \
--column-family "info" \
--hbase-create-table \
--hbase-row-key "id" \
--hbase-table "emp" \
--num-mappers 1  \
--split-by id
```



# 数据导出

## 导出HDFS到表数据

```shell
# 导出表必须存在
bin/sqoop export \
--connect jdbc:mysql://node03:3306/userdb \
--username root --password root \
--table emp \
--export-dir /test/sqoop/userdb/emp \
--input-fields-terminated-by ","
```



## 导出HBase到表数据

借助Hive表实现。

### HBase表映射到Hive外部表

<font color=red>因为Hive外部表是临时表，被删除不会影响到HBase表。避免如果是Hive内部表意外删除导致相应的HBase表被删除</font>。

```mysql
CREATE EXTERNAL TABLE sqooptohive.emp_ext (id int, name string, deg string, salary int, dept string, create_time string, update_time string, is_delete int)
row format delimited fields terminated by ','
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
   WITH SERDEPROPERTIES (
    "hbase.columns.mapping" =
     ":key,info:name,info:deg,info:salary,info:dept,info:create_time,info:update_time,info:is_delete"
   )
   TBLPROPERTIES("hbase.table.name" = "emp");
```



### 创建Hive内部表，将外部表数据导入到内部表

<font color=red>此步的作用是为了将HBase端的数据拉取到Hive端</font>。

```mysql
# 创建内部表
create table sqooptohive.emp_in (id int, name string, deg string, salary int, dept string, create_time string, update_time string, is_delete int)
row format delimited fields terminated by ',';

# 导入数据
insert overwrite table sqooptohive.emp_in select * from sqooptohive.emp_ext;
```



### Hive内部表数据导出到MySQL

清空表数据 `delete from emp;`

```shell
bin/sqoop export \
--connect jdbc:mysql://node03:3306/userdb \
--username root --password root \
--table emp \
--export-dir /user/hive/warehouse/sqooptohive.db/emp_in \
--input-fields-terminated-by ',' \
--input-null-string '\\N' --input-null-non-string '\\N'
```



# Sqoop作业

将事先定义好的数据导入导出任务按照指定流程运行。

## 创建作业

```shell
# 注意 -- import 中间有一个空格
bin/sqoop job \
--create myjob \
-- import \
--connect jdbc:mysql://node03:3306/userdb --username root --password root \
--table emp \
--delete-target-dir \
--m 1
```

## 验证作业

``` shell
# Sqoop作业列表
bin/sqoop job --list

# 检查或验证特定的工作及其详细信息
bin/sqoop job --show myjob
```

## 执行作业

```shell
# 需要root用户口令
bin/sqoop job --exec myjob
```



# 常用命令

## import

将关系型数据库中的数据导入到HDFS（包括Hive，HBase）中，如果导入的是Hive，那么当Hive中没有对应表时，则自动创建。

- 导入数据

  ```shell
  bin/sqoop import \
  --connect jdbc:mysql://node03:3306/userdb \
  --username root \
  --password root \
  --table emp \
  --hive-import \
  --hive-table sqooptohive.emp \
  -m 1
  ```

- NULL值的使用

  准备测试数据
  
  ```mysql
  -- salary INT(11) 为 NULL
  -- dept VARCHAR(10) 为 NULL
  INSERT INTO `emp`(`id`, `name`, `deg`) VALUES (1201, 'gopal', 'manager');
  ```
  
  测试
  
  ```shell
  # 导入到hive表后
  # salary是NULL，需要通过 is null 查询，在hdfs存储为null字符串
  # dept是“null”，需要通过 dept='null' 查询，在hdfs存储为null字符串
  bin/sqoop import \
  --connect jdbc:mysql://node03:3306/userdb \
  --username root \
  --password root \
  --table emp \
  --hive-import \
  --hive-table sqooptohive.emp \
  -m 1
  
  
  # --null-string '\\N'				mysql字段是null的字符串类型，解释为hive的NULL
  # --null-non-string '\\N'		mysql字段是null的非字符串类型，解释为hive的NULL
  # 使得语义相同，可以通过 is null 查询，在hdfs存储为\N字符串
  bin/sqoop import \
  --connect jdbc:mysql://node03:3306/userdb \
  --username root \
  --password root \
  --table emp \
  --hive-import --hive-overwrite \
  --hive-table sqooptohive.emp \
  --null-string '\\N' --null-non-string '\\N' \
  -m 1
  ```
  
- 增量导入 append

  <font color=red>使用场景：顺序增加，比如日志数据</font>
  
  ```shell
  # 最后一条数据1202
  bin/sqoop import \
  --connect jdbc:mysql://node03:3306/userdb --username root --password root --table emp \
  --num-mappers 1 \
  --fields-terminated-by "\t" --target-dir /test/sqoop/userdb/emp
  
  # --last-value 是上一次导入的最后一条数据；如果不指定会出现数据重复
  # 导入完成后显示
  # Incremental import complete! To run another incremental import of all data following this import, supply the following arguments:
  # 20/04/22 11:26:42 INFO tool.ImportTool:  --incremental append
  # 20/04/22 11:26:42 INFO tool.ImportTool:   --check-column id
  # 20/04/22 11:26:42 INFO tool.ImportTool:   --last-value 1205
  bin/sqoop import \
  --connect jdbc:mysql://node03:3306/userdb --username root --password root --table emp \
  --num-mappers 1 \
  --fields-terminated-by "\t" --target-dir /test/sqoop/userdb/emp \
  --check-column id --incremental append --last-value 1202
  ```
  
- 增量导入 lastmodified

  <font color=red>使用场景：部分业务数据字段更新，更新新增和修改的数据</font>

  ```shell
  # --merge-key 合并修改后的数据
  bin/sqoop import \
  --connect jdbc:mysql://node03:3306/userdb --username root --password root --table emp \
  --num-mappers 1 \
  --fields-terminated-by "\t" --target-dir /test/sqoop/userdb/emp \
  --check-column update_time --incremental lastmodified --last-value "2020-04-22 11:25:55" \
  --merge-key id
  ```

  

## export

从HDFS（包括Hive和HBase）中将数据导出到关系型数据库中。

- 导出数据

  <font color=red>全量导出，如果目标表没有主键或唯一索引，若目标表已有导出的数据则会出现重复数据；如果目标表有主键或唯一索引，若目标表已有导出的数据则会出现Duplicate异常，导出失败。</font>

  <font color=red>因此，要保证目标表是空表，或至少保证导出数据在目标表中不存在。</font>

  ```shell
  bin/sqoop export \
  --connect jdbc:mysql://node03:3306/userdb \
  --username root \
  --password root \
  --table emp \
  --export-dir /user/hive/warehouse/sqooptohive.db/emp/ \
  --num-mappers 1 \
  --input-fields-terminated-by '\001'
  ```

- NULL值的使用

  ``` shell
  # 如果hive表中有字段值是\N，导出时会出现解析数据异常：java.lang.NumberFormatException: For input string: "\N"
  # --input-null-string '\\N'			hive中的\N，字段类型是字符串类型，解析为mysql的null
  # --input-null-non-string '\\N'	hive中的\N，字段类型是非字符串类型，解析为mysql的null
  bin/sqoop export \
  --connect jdbc:mysql://node03:3306/userdb \
  --username root \
  --password root \
  --table emp \
  --export-dir /user/hive/warehouse/sqooptohive.db/emp/ \
  --input-fields-terminated-by '\001' \
  --input-null-string '\\N' --input-null-non-string '\\N' \
  --num-mappers 1
  ```

- 增量导出 updateonly

  ```shell
  # --columns 要导出的mysql数据列，但要同hive字段顺序保持一致；对于由系统控制字段如create_time和update_time，不要包括在内（在设计表时，注意将由系统控制字段放在表的最后）
  
  # --update-key 可以包含多个列，用逗号隔开，用于匹配mysql唯一数据；且指定字段(多个)必须出现在--columns中；不必是主键或唯一索引
  
  # --update-mode updateonly 只对mysql已有数据更新
  
  bin/sqoop export \
  --connect jdbc:mysql://node03:3306/userdb \
  --username root --password root --table emp \
  --export-dir /user/hive/warehouse/sqooptohive.db/emp/ \
  -m 1 --fields-terminated-by '\001' \
  --columns id,name,deg,salary,dept \
  --update-key id --update-mode updateonly
  ```

- 增量导出 allowinsert

  ``` shell
  # --update-key 必须注意，指定的字段必须是主键或唯一索引，否则会出现重复数据。原理是，首先向mysql插入，如果出现重复唯一主键错误时，然后尝试更新
  
  # --columns 是hive中的字段，可以没有id，对应mysql中的字段；而--update-key id 是mysql的字段，主键自增
  
  # --update-mode allowinsert mysql已有数据则更新，没有数据则插入
  
  # alter table emp add primary key(id);
  
  bin/sqoop export \
  --connect jdbc:mysql://node03:3306/userdb \
  --username root --password root --table emp \
  --export-dir /user/hive/warehouse/sqooptohive.db/emp/ \
  -m 1 --fields-terminated-by '\001' \
  --columns id,name,deg,salary,dept \
  --update-key id --update-mode allowinsert
  ```

- 临时表

  由于Sqoop将导出过程分解为多个事务，因此失败的导出作业可能会导致将部分数据提交到数据库。这可能进一步导致后续作业由于某些情况下的插入冲突而失败，或导致其他作业中的重复数据。可以通过 --staging-table 选项指定临时表来解决此问题，该选项充当用于暂存导出数据的辅助表。<font color=red>分阶段数据最终在单个事务中移动到目标表</font>。

  创建临时表

  ```mysql
  -- 创建临时表
  CREATE TABLE `emp_tmp`
  (
      `id`          INT(11)            DEFAULT NULL,
      `name`        VARCHAR(100)       DEFAULT NULL,
      `deg`         VARCHAR(100)       DEFAULT NULL,
      `salary`      INT(11)            DEFAULT NULL,
      `dept`        VARCHAR(10)        DEFAULT NULL,
      `create_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
      `update_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
      `is_delete`   BIGINT(20)         DEFAULT '1'
  ) ENGINE = INNODB
    DEFAULT CHARSET = latin1;
  ```

  导出数据

  ```shell
  # --staging-table 在插入目标表之前将暂存数据的表
  # --clear-staging-table 在数据处理之前删除临时表中所有数据
  # 不可与 --update-key 更新模式共用，更新已存在数据（会出现重复数据或者主键重复）
  bin/sqoop export \
  --connect jdbc:mysql://node03:3306/userdb \
  --username root --password root --table emp \
  --export-dir /user/hive/warehouse/sqooptohive.db/emp/ \
  -m 1 --fields-terminated-by '\001' \
  --columns id,name,deg,salary,dept \
  --update-mode allowinsert \
  --staging-table emp_tmp --clear-staging-table
  ```

  

 ## codegen

将关系型数据库中的表映射为一个Java类，在该类中有各列对应的各个字段，以及相关方法。

```shell
# --bindir 本地目录
bin/sqoop codegen \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password root \
--table emp \
--bindir /home/hadoop/empjar \
--class-name Emp \
--fields-terminated-by "\t"
```



## create-hive-table

生成与关系数据库表结构对应的hive表结构。

```shell
bin/sqoop create-hive-table \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password root \
--table emp \
--hive-table sqooptohive.emp_struc
```



## eval

可以快速的使用SQL语句对关系型数据库进行操作，经常用于在import数据之前，了解一下SQL语句是否正确，数据是否正常，并可以将结果显示在控制台。

```shell
bin/sqoop eval \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password root \
--query "SELECT * FROM emp"
```



## import-all-tables

可以将RDBMS中的所有表导入到HDFS中，每一个表都对应一个HDFS目录。

```shell
bin/sqoop import-all-tables \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password root \
--warehouse-dir /test/sqoop/userdb \
-m 1
```



## job

```shell
# 创建job
# 注意import-all-tables和它左边的--之间有一个空格
# 如果需要连接metastore，则--meta-connect jdbc:hsqldb:hsql://node03:16000/
bin/sqoop job \
--create myjob \
-- import-all-tables \
--connect jdbc:mysql://node03:3306/userdb --username root --password root \
-m 1

# 列出所有job
bin/sqoop job --list

# 执行job
bin/sqoop job --exec myjob

# 删除job
bin/sqoop job --delete myjob

# 列出job的参数，需要输入数据库的密码
bin/sqoop job --show myjob
```

- 在执行一个job时，如果需要手动输入数据库密码，可以做如下优化

  ```xml
  <property> 
  	<name>sqoop.metastore.client.record.password</name> 
    <value>true</value> 
    <description>If true, allow saved passwords in the metastore.</description> 
  </property>
  ```

  

## list-databases

列出所有数据库

```shell
bin/sqoop list-databases \
--connect jdbc:mysql://node03:3306 \
--username root \
--password root
```



## list-tables

列出数据库的所有表

```shell
bin/sqoop list-tables \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password root
```



## merge

准备数据

```shell
# 旧数据
# 1201	gopal	manager	null	null	2020-04-24 16:28:05.0	2020-04-24 16:28:05.0	1
bin/sqoop import \
--connect jdbc:mysql://node03:3306/userdb --username root --password root --table emp \
--num-mappers 1 \
--fields-terminated-by "\t" --target-dir /test/sqoop/userdb/merge/old

# 新数据
# 1201	gopal	manager	50000	TP	2020-04-25 16:38:58.0	2020-04-25 16:38:58.0	1
# 1202	manisha	Proof reader	50000	TP	2020-04-25 16:38:59.0	2020-04-25 16:38:59.0	1
bin/sqoop import \
--connect jdbc:mysql://node03:3306/userdb --username root --password root --table emp \
--num-mappers 1 \
--fields-terminated-by "\t" --target-dir /test/sqoop/userdb/merge/new
```

创建JavaBean

```shell
# --fields-terminated-by "\t" 必须存在，否则分隔字段数据时会出现解析问题
bin/sqoop codegen \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password root \
--table emp \
--bindir /home/hadoop/sqoopdatas/empjar \
--class-name Emp \
--fields-terminated-by "\t"
```

合并

```shell
# 1201	gopal	manager	50000	TP	2020-04-25 16:38:58.0	2020-04-25 16:38:58.0	1
# 1202	manisha	Proof reader	50000	TP	2020-04-25 16:38:59.0	2020-04-25 16:38:59.0	1
# --new-data HDFS 待合并的数据目录，合并后在新的数据集中保留
# --onto HDFS合并后，重复的部分在新的数据集中被覆盖
# --target-dir 合并后的数据在HDFS里存放的目录
bin/sqoop merge \
--new-data /test/sqoop/userdb/merge/new/ \
--onto /test/sqoop/userdb/merge/old/ \
--target-dir /test/sqoop/userdb/merge/result \
--jar-file /home/hadoop/sqoopdatas/empjar/Emp.jar \
--class-name Emp \
--merge-key id
```



## metastore

记录了Sqoop job的元数据信息，如果不启动该服务，那么默认job元数据的存储目录为~/.sqoop，可在sqoop-site.xml中修改。

```shell
# 启动sqoop的metastore服务
bin/sqoop metastore

# 关闭sqoop的metastore服务
bin/sqoop metastore --shutdown
```



## help

打印sqoop帮助信息 

```shell
bin/sqoop help metastore
```



## version

打印sqoop版本信息

```shell
bin/sqoop version
```

