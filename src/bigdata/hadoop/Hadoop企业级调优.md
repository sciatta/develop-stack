# hdfs参数调优

## hdfs-site.xml

- dfs.namenode.handler.count 默认10

  NameNode有一个工作线程池，用来处理不同DataNode并发心跳以及Client并发元数据操作。对于大规模集群或者有大量客户端的集群来说，通常需要增大参数。设置该值的一般原则是将其设置为集群大小的自然对数乘以20，即 `20logN`，N为集群大小。如集群规模为8台时，此参数设置为60。

  此参数不可过小，否则导致请求超时；也不可过大，否则，队列任务积压过多，请求处理时间过慢。

- dfs.namenode.edits.dir 和 dfs.namenode.name.dir

  - 两个目录尽量分开，达到最低写入延迟
  - dfs.namenode.name.dir 多目录冗余存储配置



# yarn参数调优

## yarn-site.xml

- yarn.nodemanager.resource.memory-mb 默认 8192MB

  可分配给容器的物理内存总量。当此值设置 -1 和 `yarn.nodemanager.resource.detect-hardware-capabilities` 设置为 true时, 此值自动计算。如果内存资源不够8G，则需要调小此值。

- yarn.scheduler.minimum-allocation-mb 默认 1024MB

  向RM发起的容器请求（单个任务）最小可分配内存。

- yarn.scheduler.maximum-allocation-mb  默认 8192MB

  向RM发起的容器请求（单个任务）最大可分配内存。

- yarn.scheduler.minimum-allocation-vcores 默认1

  每个Container申请的最小CPU核数

- yarn.scheduler.maximum-allocation-vcores 默认4

  每个Container申请的最大CPU核数



# MapReduce 参数调优

## mapred-site.xml

- mapreduce.map.memory.mb

  MapTask使用内存上限

- mapreduce.reduce.memory.mb

  ReduceTask使用内存上限

- mapreduce.map.cpu.vcores

  MapTask使用cpu核数

- mapreduce.reduce.cpu.vcores

  ReduceTask使用cpu核数

- mapreduce.reduce.shuffle.parallelcopies

  在Shuffle阶段，ReduceTask到MapTask获取数据的并行数

- mapreduce.reduce.shuffle.merge.percent

  Buffer中的数据达到多少比例开始写入磁盘

- mapreduce.reduce.shuffle.input.buffer.percent

  Buffer占Reduce可用内存的比例

- mapreduce.reduce.input.buffer.percent

  指定多少比例的内存用来存放Buffer中的数据

- mapreduce.task.io.sort.mb 默认100MB

  Shuffle的环形缓冲区大小

- mapreduce.map.sort.spill.percent 默认0.8

  环形缓冲区溢出的阈值

- mapreduce.map.maxattempts 默认4

  MapTask最大重试次数，一旦重试参数超过该值，则认为MapTask运行失败

- mapreduce.reduce.maxattempts 默认4

  ReduceTask最大重试次数，一旦重试参数超过该值，则认为ReduceTask运行失败

- mapreduce.task.timeout 默认600000ms（10m）

  当一个Task即不读取数据或写出数据，也不更新状态，则该Task被强制终止前的超时时间



# MapReduce效率优化

效率问题主要从两方面着手：

1. 计算机性能

   内存、cpu、磁盘健康、网络

2. I/O操作优化

   - 数据倾斜
   - MapTask和ReduceTask数量设置是否合理
   - MapTask运行时间过长，导致ReduceTask等待时间过长
   - 小文件过多
   - 大量不可切分的超大文件
   - spill溢写磁盘次数过多
   - merge合并次数过多



## 数据输入阶段

- 合并小文件

  大量小文件，会产生大量的MapTask，加载MapTask比较耗时。采用CombineTextInputFormat解决大量小文件输入的场景。



## MapTask运行阶段

- 减少spill溢写磁盘次数

  调整 `mapreduce.task.io.sort.mb 默认100MB` 和 `mapreduce.map.sort.spill.percent 默认0.8` ，增大触发spill的内存上限，减少spill次数，从而减少磁盘io。

- 减少merge合并次数

  调整 `mapreduce.task.io.sort.factor 默认10` ，增大一次合并文件数量，减少merge次数。

- 在不影响业务逻辑前提下，进行Combine规约处理

  减少网络io



## ReduceTask运行阶段

- 合理设置MapTask和ReduceTask数量

  太少，会造成任务等待；太多，会导致任务间竞争资源，任务处理超时等问题。

- 设置MapTask和ReduceTask共存

  调整 `mapreduce.job.reduce.slowstart.completedmaps 默认0.05` MapTask运行一定程度后，ReduceTask开始运行，减少ReduceTask等待时间。 

- 规避使用ReduceTask

  ReduceTask在拉取MapTask数据集时会产生大量网络io消耗。

- 合理设置ReduceTask端的buffer

  buffer内存数据达到阈值时会写磁盘，导致Reducer处理数据时需要读磁盘。调整 `mapreduce.reduce.input.buffer.percent 默认0` 使得buffer中的一部分数据可以直接输出到Reducer，从而较少磁盘io开销。但需注意会增大内存使用量。



## IO传输阶段

- 采用数据压缩方式，减少网络io

  snappy 或 lzo

- 采用SequenceFile二进制文件



## 数据倾斜

- 抽样和范围分区

  通过对原始数据抽样得到的结果来预设分区边界值。

- 自定义分区

  通过key的背景知识自定义分区。

- Combine

  聚合精简数据，减小数据倾斜。

