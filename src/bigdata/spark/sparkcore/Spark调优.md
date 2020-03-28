# 分配更多的资源

增加资源与性能提升成正比。在资源比较充足的情况下，尽可能的使用更多的计算资源，尽量去调节到最大的大小。

- `--driver-memory 1g` 

  配置driver的内存，使得driver端可以存储更多的数据，避免出现OOM。如果RDD有大量数据，在做collect算子操作时，会把各个Executor的数据转换为数组，拉取到driver端，如果内存不足就会出现OOM异常。

- `--num-executors 3`

  配置Executor的数量，提升并行能力。配合 `--executor-cores` 参数可以提高每个批次最大并行的task数量。

- `--executor-cores 3`

  配置每一个Executor的cpu个数，提升并行能力。

- `--executor-memory 1g`

  配置每一个Executor的内存大小。

  - 对RDD进行cache，可以缓存更多的数据，减少磁盘IO。
  - 进行Shuffle操作，reduce端会拉取数据存放到缓存，缓存不够会刷写磁盘。减少磁盘IO。
  - task执行过程中会创建大量对象，如果内存设置比较小，会导致频繁GC。



# 提高并行度

官方推荐，task数量设置成 spark Application 总cpu core数量的2~3倍 。因为与理想情况不同，有的task运行快，有的运行慢，如果设置同CPU核数相同，则运行快的task对应的CPU就会空闲下来造成资源浪费。如果有后补task，则CPU就不会空闲，并且数据被分摊。

- `spark.defalut.parallelism`

  默认是没有值的，如果设置为10，它会在shuffle的过程起作用。如 `val rdd2 = rdd1.reduceByKey(_+_) `
  此时rdd2的分区数就是10。

- `rdd.repartition`

  该方法会生成一个新的rdd，使其分区数变大。由于一个partition对应一个task，则partition越多，对应的task个数就越多，通过这种方式可以提高并行度。

- `spark.sql.shuffle.partitions`

  默认为200。可以适当增大，来提高并行度。 如 `spark.sql.shuffle.partitions = 500`



# RDD的重用和持久化

默认情况下多次对一个rdd执行算子操作去获取不同的rdd，都会对这个rdd及之前的父rdd全部重新计算一次。
这种情况在实际开发的时候会经常遇到，但是我们一定要避免一个rdd重复计算多次，否则会导致性能急剧降低。**可以把多次使用到的rdd，也就是公共rdd进行持久化**，避免后续需要，再次重新计算，提升效率。

- 可以调用rdd的cache或者persist方法



# 广播变量的使用

当某一个stage的task需要一份共有数据，如果task数量非常大的话，不同的task线程会拉取这份数据，导致大量的网络传输开销，并且所有都会拥有这份数据，导致占用大量的内存空间。如果rdd需要持久化，势必内存不够用需要刷写磁盘，导致磁盘IO性能降低。同时，也会导致内存紧张，频繁GC影响spark作业的运行效率。

- 广播变量的运行机制

  - 广播变量初始的时候，在Drvier上有一份副本。通过在Driver把共享数据转换成广播变量。

  - task在运行的时候，想要使用广播变量中的数据，此时首先会在自己本地的Executor对应的BlockManager中，尝试获取变量副本；如果本地没有，那么就从Driver远程拉取广播变量副本，并保存在本地的BlockManager中；
  - 此后这个executor上的task，都会直接使用本地的BlockManager中的副本。那么这个时候所有该executor中的task都会使用这个广播变量的副本。也就是说一个executor只需要在第一个task启动时，获得一份广播变量数据，之后的task都从本节点的BlockManager中获取相关数据。
  - executor的BlockManager除了从driver上拉取变量，也可**就近从其他节点的BlockManager上拉取变量副本**。

- 注意事项
  - 能不能将一个RDD使用广播变量广播出去？不能，因为RDD是不存储数据的。可以将RDD的结果广播出去。
  - 广播变量只能在Driver端定义，不能在Executor端定义。
  - 在Driver端可以修改广播变量的值，在Executor端无法修改广播变量的值。
  - 如果executor端用到了Driver的变量，如果不使用广播变量，在Executor有多少task就有多少Driver端的变量副本；如果使用广播变量，在每个Executor中只有一份Driver端的变量副本。



# 尽量避免使用shuffle类算子

spark中的shuffle涉及到网络传输，本质就是将满足一定条件的key拉取到相应的节点，然后进行聚合或join运算。spark作业中最耗时的地方就是shuffle操作，应尽量避免。

- 当join运算的某一个rdd数据较少时，可以通过将其设置为广播变量的形式，来代替shuffle
- 如果必须要使用shuffle操作，可以使用map-side预聚合方式来减少网络数据传输量，类似于MapReduce的combiner。如 `reduceByKey` 会在map端使用用户自定义的函数进行预聚合；而 `groupByKey` 不会进行预聚合，会将全量数据分发到reduce端，性能相对来说会比较差。



# 使用高性能的算子

- 使用reduceByKey/aggregateByKey替代groupByKey
  - 利用预聚合特性
- 使用mapPartitions替代map
  - mapPartitions每次函数调用遍历一个分区的数据，而map每次遍历一条数据
  - 如果分区数据过大时，可能会出现OOM问题。因为一次函数调用需要处理一个分区的数据，导致内存不够用，GC又无法回收太多对象
- 使用foreachPartitions替代foreach
  - foreachPartitions一次函数调用处理一个分区的数据，而foreach仅处理一条数据
- 使用filter之后进行coalesce操作
  - 如果filter算子过滤掉rdd的部分数据后，建议使用coalesce减少rdd的分区数量。由于使用filter算子，原分区数据会减少，若仍用原来数量的task处理分区数据，会有一些资源浪费。此时task越多，反而运行越慢。
- 使用repartitionAndSortWithinPartitions替代repartition与sort
  - repartitionAndSortWithinPartitions是Spark官网推荐的一个算子。官方建议如果需要在repartition重分区之后，还要进行排序，建议直接使用repartitionAndSortWithinPartitions算子。因为该算子可以一边进行重分区的shuffle操作，一边进行排序。shuffle与sort两个操作同时进行，比先shuffle再sort来说，性能要高。



# 使用Kryo优化序列化性能

- Spark在进行任务计算的时候，会涉及到数据跨进程的**网络传输**、数据的**持久化**，这个时候就需要对数据进行序列化。Spark默认采用Java的序列化器。
  - 优点：简单，只需要实现Serializble接口
  - 缺点：速度慢，占用内存空间大
- Spark支持使用Kryo序列化机制。
  - 优点：速度快，序列化后的数据要更小，大概是Java序列化机制的1/10
- 开启Kryo序列化机制
  - conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
  - conf.registerKryoClasses(Array(classOf[MyClass1], classOf[MyClass2]))
- Kryo序列化启用后生效的地方
  - 算子函数中使用到的外部变量
  - 持久化RDD时进行序列化
  - 产生shuffle的地方，也就是宽依赖



# 使用fastutil优化数据格式

- fastutil是扩展了Java标准集合框架（Map、List、Set；HashMap、ArrayList、HashSet）的类库，提供了特殊类型的map、set、list和queue，可以提供**更小的内存占用**，**更快的存取速度**。

- 使用场景
  - 算子函数使用的外部变量
    - 首先从源头上就减少内存的占用（fastutil），通过广播变量进一步减少内存占用，再通过Kryo序列化类库进一步减少内存占用。
  - 算子函数里使用了比较大的集合Map/List



# 调节数据本地化等待时长

- Spark的task分配算法，优先会希望每个task正好分配到它要计算的数据所在的节点（移动计算），这样的话就不用在网络间传输数据；但如果数据所在节点不具备分配资源的条件，可以设置让这个task先等待一段时间再分配到数据所在节点。如果超过了这个时间等待阈值，则就近分配资源运行task。
- 本地化级别
  - PROCESS_LOCAL（进程本地化） 数据在Executor的BlockManager中，性能最好
  - NODE_LOCAL（节点本地化） 数据在本节点作为hdfs的一个block存在，或者在其他Executor的BlockManager中，数据需要在不同进程中传输，性能其次
  - RACK_LOCAL（机架本地化） 数据在同一机架的不同节点上，需要网络传输数据。性能比较差
  - ANY（无限制） 数据在集群中的任意一个地方，且不再同一个机架上。性能最差
- 数据本地化等待时长
  - 首先采用最佳的方式，等待3s后降级，还是无法运行，继续降级，最后只能采用最差级别。
    - spark.locality.wait（默认是3s）
    - spark.locality.wait.process
    - spark.locality.wait.node
    - spark.locality.wait.rack
  - 注意如果本地化等待时间设置过大，则会出现等待时间甚至高于不同节点通过网络拉取的时间，这样虽然实现task在数据本地运行，但反而会使得spark作业的运行时间增加。



# 基于Spark内存模型调优

静态内存模型无法借用其他空闲内存区域；

而统一内存模型，Storage和Executor内存区域是可以相互借用的，但是Executor内存使用会优先于Storage，因为Executor需要马上申请内存，而Storage并不是很急迫，因此可以暂时缓存到磁盘。

- Executor申请内存，而Executor不够，则借用Storage内存，如果Storage内存已被使用，则驱逐Storage写到磁盘
- Executor使用内存不多，Storage借用Executor内存，如果此时Executor需要申请内存但已被Storage借用，则驱逐Storage借用的内存写到磁盘

在spark1.6版本以前，spark的executor使用的静态内存模型，但是从spark1.6开始，多增加了一个统一内存模型。通过 `spark.memory.useLegacyMode` 参数配置，默认false，表示用的是新的统一内存模型。



## 内存模型

- 静态内存模型
  - Storage内存区域，由 `spark.storage.memoryFraction` 控制（默认是0.6，占总内存的60%）
    - Reserved 预留，防止OOM（60% * 10% = 6%）
    - 用于<font color =red>缓存RDD数据和广播变量数据</font>，由 `spark.storage.saftyFraction` 控制（60% * 90% = 54%）
      - 用于unroll，缓存iterator形式的block数据，由 `spark.storage.unrollFraction` 控制（60% * 90% * 20% = 10.8%）
  - Executor内存区域，由 `spark.shuffle.memoryFraction` 控制（默认是0.2，占总内存的20%）
    - Reserved 预留，防止OOM（20% * 20% = 4%）
    - 用于<font color=red>缓存shuffle过程中的数据</font>，由 `spark.shuffle.saftyFraction` 控制（20% * 80% =16%）
  - 其他内存区域，由以上两部分内存大小决定（默认是0.2，占总内存的20%），<font color=red>用户定义的数据结构或spark内部元数据</font>
- 统一内存模型
  - 可用内存（usable）= 总内存 - 预留内存
    - 统一内存（Storage + Executor）由 `spark.memory.fraction` 控制（2.x 默认0.6，占可用内存的60%；1.6默认0.75）
      - Storage内存区域，用于<font color =red>缓存RDD数据和广播变量数据</font>，由 `spark.memory.storageFraction` 控制（默认是0.5，usable * 60% * 50%）
      - Executor内存区域，用于<font color=red>缓存shuffle过程中的数据</font>，由 1-  `spark.memory.storageFraction` 控制（默认是0.5，usable * 60% * 50%）
    - 其他内存区域，<font color=red>用户定义的数据结构或spark内部元数据</font> （占可用内存的40%）
  -  预留内存 300M，作用同其他内存区域相同，防止OOM

