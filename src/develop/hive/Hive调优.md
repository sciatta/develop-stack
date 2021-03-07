# Fetch抓取

Fetch抓取是指Hive中对某些情况的查询可以不必使用MapReduce计算。这些情况包括：

- 全局查找 `select * from score;`
- 字段查找 `select s_id from score;`
- 限制查找 `select s_id from score limit 3;`

由参数 `hive.fetch.task.conversion` 控制，老版本默认是 `minimal`，新版本默认是 `more`。当参数值为 `more`时，以上查询不使用MapReduce，直接读取存储目录下的文件。

而当 `set hive.fetch.task.conversion=none;` 时，以上查询会使用MapReduce。



# 本地模式

在Hive客户端测试时，默认情况下是启用hadoop的job模式把任务提交到集群中运行，这样会导致计算非常缓慢；可以通过本地模式在单台机器上处理任务。对于小数据集，执行时间可以明显被缩短。

```mysql
-- 开启本地运行模式，并执行查询语句
set hive.exec.mode.local.auto=true;

-- 设置local mr的最大输入数据量，当输入数据量小于这个值时采用 local mr 的方式
-- 默认为134217728，即128M
set hive.exec.mode.local.auto.inputbytes.max=50000000;

-- 设置 local mr 的最大输入文件个数，当输入文件个数小于这个值时采用 local mr 的方式
-- 默认为4
set hive.exec.mode.local.auto.input.files.max=5;

-- 执行查询的sql语句
select * from teacher cluster by t_id;

-- 关闭本地运行模式
set hive.exec.mode.local.auto=false;
```



# 表优化

## 小表和大表 join

- 将key相对分散，并且数据量小的表放在join的左边，这样可以有效减少内存溢出错误发生的几率。新版的hive已经对小表 join 大表和大表 join 小表进行了优化。小表放在左边和右边已经没有明显区别

- 可以使用map join让小的维度表（1000条以下的记录条数）先加载内存，在map端完成join操作

  ```mysql
  -- 开启mapjoin参数（默认是true）
  set hive.auto.convert.join = true;
  
  -- 大小表阈值
  set hive.mapjoin.smalltable.filesize=26214400;
  ```

  

- 多个表关联时，最好分拆成小段，避免大sql（无法控制中间Job）

  

## 大表 join 大表

- 空 key 过滤
  - 有时join超时是因为某些key对应的数据太多，而相同key对应的数据都会发送到相同的reducer上，从而导致内存不够
  - 如果是由于异常数据key造成，可以过滤异常数据

- 空 key 转换
  - 有时虽然某个key为空对应的数据很多，但是相应的数据不是异常数据，必须要包含在join的结果中，此时可以为表中key为空的字段赋一个随机值，使得数据随机均匀地分不到不同的reducer上



## group by

- 默认情况下，Map阶段同一Key数据分发给一个reduce，当一个key数据过大时就会发生数据倾斜

- 并不是所有的聚合操作都需要在Reduce端完成，很多聚合操作都可以先在Map端进行部分聚合，最后在Reduce端得出最终结果

- 开启Map端聚合参数设置

  ```mysql
  -- 是否在Map端进行聚合，默认为True
  set hive.map.aggr = true;
  
  -- 在Map端进行聚合操作的条目数目
  set hive.groupby.mapaggr.checkinterval = 100000;
  
  -- 有数据倾斜的时候进行负载均衡（默认是false）
  -- 当选项设定为true，生成的查询计划会有两个MR Job。第一个MR Job中，Map的输出结果会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是相同的Group By Key有可能被分发到不同的Reduce中，从而达到负载均衡的目的；第二个MR Job再根据预处理的数据结果按照Group By Key分布到Reduce中（这个过程可以保证相同的Group By Key被分布到同一个Reduce中），最后完成最终的聚合操作。
  set hive.groupby.skewindata = true;
  ```

  

## count(distinct) 

- 数据量小的时候无所谓，数据量大的情况下，由于count distinct 操作需要用一个reduce Task来完成，这一个Reduce需要处理的数据量太大，就会导致整个Job很难完成，一般count distinct使用，先group by再count的方式替换

  ```mysql
  select count(ip) from (select ip from log_text group by ip) t;
  ```



## 笛卡尔积

- 尽量避免笛卡尔积，即避免join的时候不加on条件，或者无效的on条件
- Hive只能使用1个reducer来完成笛卡尔积



# 使用分区剪裁、列剪裁

尽可能早地过滤掉尽可能多的数据量，避免大量数据流入外层SQL。

- 列剪裁
  - 只获取需要的列的数据，减少数据输入。

- 分区裁剪

  - 分区在hive实质上是目录，分区裁剪可以方便直接地过滤掉大部分数据。

  - 尽量使用**分区过滤**



# 并行执行

把一个sql语句中没有相互依赖的阶段并行去运行。提高集群资源利用率。

```mysql
-- 开启并行执行
set hive.exec.parallel=true;

-- 同一个sql允许最大并行度，默认为8
set hive.exec.parallel.thread.number=16;
```



# 严格模式

Hive提供了严格模式，可以防止用户执行效率极差的查询。

```mysql
-- 设置非严格模式（默认）
set hive.mapred.mode=nonstrict;

-- 设置严格模式
set hive.mapred.mode=strict;
```

开启严格模式可以禁止3种类型的查询：

1. 对于分区表，除非where语句中含有分区字段过滤条件来限制范围，否则不允许执行
2. 对于使用了order by语句的查询，要求必须使用limit语句
3. 限制笛卡尔积的查询



# JVM重用

JVM重用是Hadoop调优参数的内容，其对Hive的性能具有非常大的影响，特别是对于很难避免小文件的场景或task特别多的场景，这类场景大多数执行时间都很短。

Hadoop的默认配置通常是使用派生JVM来执行map和Reduce任务的。这时JVM的启动过程可能会造成相当大的开销，尤其是执行的job包含有成百上千task任务的情况。JVM重用可以使得JVM实例在同一个job中重新使用N次。

可以在Hadoop的mapred-site.xml文件中进行配置。通常在10-20之间，具体多少需要根据具体业务场景测试得出。

```xml
<property>
  <name>mapreduce.job.jvm.numtasks</name>
  <value>10</value>
  <description>How many tasks to run per jvm. If set to -1, there is no limit. 
  </description>
</property>
```

也可以在hive当中设置

```mysql
set mapred.job.reuse.jvm.num.tasks=10;
```

这个功能的缺点是，开启JVM重用将一直占用使用到的task插槽，以便进行重用，直到任务完成后才能释放。如果某个“不平衡的”job中有某几个Reduce task执行的时间要比其他Reduce task消耗的时间多的多的话，那么保留的插槽就会一直空闲着却无法被其他的job使用，直到所有的task都结束了才会释放。



# 推测执行

在分布式集群环境下，因为程序Bug（包括Hadoop本身的bug），负载不均衡或者资源分布不均等原因，会造成同一个作业的多个任务之间运行速度不一致，有些任务的运行速度可能明显慢于其他任务（比如一个作业的某个任务进度只有50%，而其他所有任务已经运行完毕），则这些任务会拖慢作业的整体执行进度。为了避免这种情况发生，Hadoop采用了推测执行（Speculative Execution）机制，它根据一定的法则推测出“拖后腿”的任务，并为这样的任务启动一个备份任务，让该任务与原始任务同时处理同一份数据，并最终选用最先成功运行完成任务的计算结果作为最终结果。

可以在Hadoop的mapred-site.xml文件中进行配置。

```xml
<property>
  <name>mapreduce.map.speculative</name>
  <value>true</value>
  <description>If true, then multiple instances of some map tasks may be executed in parallel.</description>
</property>

<property>
  <name>mapreduce.reduce.speculative</name>
  <value>true</value>
  <description>If true, then multiple instances of some reduce tasks may be executed in parallel.</description>
</property>
```

注意，如果用户输入数据本身很大，需要长时间执行MapTask或者ReduceTask。如果开启此参数，可能会造成更多资源的浪费。



# 压缩

使用压缩的优势是可以最小化所需要的磁盘存储空间，以及减少磁盘和网络io操作

- Hive表中间数据压缩

  ```mysql
  -- 开启hive中间传输数据压缩功能
  set hive.exec.compress.intermediate=true;
  
  -- 开启mapreduce中map输出压缩功能
  set mapreduce.map.output.compress=true;
  
  -- 设置mapreduce中map输出数据的压缩方式
  set mapreduce.map.output.compress.codec=org.apache.hadoop.io.compress.SnappyCodec;
  ```

- Hive表最终输出结果压缩

  ```mysql
  -- 开启hive最终输出数据压缩功能
  set hive.exec.compress.output=true;
  
  -- 开启mapreduce最终输出数据压缩
  set mapreduce.output.fileoutputformat.compress=true;
  
  -- 设置mapreduce最终数据输出压缩方式
  set mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.SnappyCodec;
  
  -- 设置mapreduce最终数据输出压缩为块压缩
  set mapreduce.output.fileoutputformat.compress.type=BLOCK;
  ```



# 使用EXPLAIN（执行计划）

hive将sql解释为多个MapReduce，可以通过explain查看执行计划，了解MapReduce的执行顺序。大致顺序是 `from... where... select... group by... having... order by...`

```mysql
EXPLAIN [EXTENDED|DEPENDENCY|AUTHORIZATION] query
```



# 合理设置MapTask数量

- MapTask数量的决定因素

  - 文件个数
- 文件大小
  
  可以通过 `computeSliteSize(Math.max(minSize, Math.min(maxSize, blocksize)))` 来调整切片大小，继而调整MapTask数量。
  
  ```mysql
  -- minsize（切片最小值）参数调的比blockSize大，则可以让切片变大，MapTask数量变少
  set mapreduce.input.fileinputformat.split.minsize=1;
  
  -- maxsize（切片最大值）参数如果调到比blocksize小，则可以让切片变小，MapTask数量变多
  set mapreduce.input.fileinputformat.split.maxsize=256000000;
  ```
  
  

- MapTask数越多越好？

  - 如果一个job有大量小文件，则每个小文件也会被当做一个块，用一个MapTask来完成，而一个MapTask启动和初始化的时间远远大于逻辑处理的时间，就会造成很大的资源浪费。而且，同时可执行的MapTask数是受限的。

  - 解决

    - 减少MapTask数

    - JVM重用

    - 小文件合并
    
      ```mysql
      set mapred.max.split.size=112345600;
      set mapred.min.split.size.per.node=112345600;
      set mapred.min.split.size.per.rack=112345600;
      set hive.input.format= org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;
      ```

  

- 每个MapTask处理接近128m的文件块，就是最优的？

  - 如果字段少，记录多，而且Map逻辑复杂，用一个MapTask处理肯定是比较耗时的
  - 解决
    - 增大MapTask数



# 合理设置ReduceTask数量

- 调整ReduceTask数量的方法

  - 通过公式调整

    ```mysql
    -- 参数1：每个Reduce处理的数据量默认是256MB
    set hive.exec.reducers.bytes.per.reducer=256000000;
    
    -- 参数2：每个任务最大的reduce数，默认为1009
    set hive.exec.reducers.max=1009;
    
    -- N 为ReduceTask数量
    N=min(参数2，总输入数据量/参数1)
    ```

  - 设置ReduceTask数量

    ```mysql
    -- 设置每一个job中reduce个数
    set mapreduce.job.reduces=3;
    ```

- ReduceTask数越多越好？

  - 过多的启动和初始化reduce会消耗时间和资源
  - 同时过多的reduce会生成很多个文件，也有可能出现小文件问题



# 合并小文件

在Map-only的任务结束时合并小文件

```mysql
set hive.merge.mapfiles = true
```

在Map-Reduce的任务结束时合并小文件

```mysql
set hive.merge.mapredfiles = true
```

合并文件大小

```mysql
set hive.merge.size.per.task = 256*1000*1000
```

输出文件的平均大小小于该值时，启动一个独立的map-reduce任务进行文件合并

```mysql
set hive.merge.smallfiles.avgsize=16000000
```



# 排序

对于排序问题，能够不用全局排序就一定不要使用全局排序order by（只有一个ReduceTask，效率低），如果一定要使用order by一定要加上limit。



# MapTask尽量多处理数据

能够在MapTask处理完成的任务，尽量在MapTask多处理任务，避免数据通过shuffle到ReduceTask，通过网络拷贝导致性能低下。

- map aggr
- map join



 # 尽量减少IO操作

- 多表插入

  Hive支持多表插入，可以在同一个查询中使用多个insert子句，这样的好处是我们只需要扫描一遍源表就可以生成多个不相交的输出。可以减少表的扫描，从而减少 JOB 中 MR的 STAGE 数量，达到优化的目的。

  ```mysql
  -- 多表插入的关键点在于将所要执行查询的表语句 "from 表名"，放在最开头位置
  from test1
  insert overwrite table test2 partition (age) select name,address,school,age
  insert overwrite table test3 select name,address
  ```

- 一次计算，多次使用

  - 提高代码可读性，简化SQL
  - 一次分析，多次使用

  ```mysql
  -- with as就类似于一个视图或临时表，可以用来存储一部分的sql语句作为别名，不同的是 with as 属于一次性的，而且必须要和其他sql一起使用才可以
  WITH t1 AS (
          SELECT * FROM carinfo
      ), 
       t2 AS (
          SELECT * FROM car_blacklist
      )
  SELECT * FROM t1, t2
  ```

