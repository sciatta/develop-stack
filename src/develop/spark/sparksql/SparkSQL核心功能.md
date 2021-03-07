# 概述

SparkSQL是Apache Spark用来处理**结构化数据**的一个模块。

- 易集成
  - 将SQL查询与Spark程序无缝混合
  - 可以使用不同的语言进行代码开发，如Java，Scala，Python 和 R
- 统一数据访问
  - 以相同的方式连接到任何数据源
- 兼容Hive
  - 运行 SQL 或者 HiveQL 查询已存在的数据仓库
- 支持标准的数据库连接

# DataFrame

## 概述

- 在Spark中，DataFrame是一种以RDD为基础的分布式数据集，类似于传统数据库的二维表格
- DataFrame带有Schema元信息，即DataFrame所表示的二维表数据集的每一列都带有名称和类型，但底层做了更多的优化
- DataFrame可以从很多数据源构建。如：已经存在的RDD、结构化文件、外部数据库、Hive表
- RDD可以把内部元素看作是java对象；而DataFrame可以把内部元素看作是Row对象，表示一行一行的数据

## DataFrame和RDD的优缺点

### RDD

- 优点
  - 编译时类型安全
  - 具有面向对象的编程风格
- 缺点
  - 构建大量的java对象占用了大量heap堆空间，导致频繁的GC（程序在进行垃圾回收的过程中，所有的任务都是暂停，影响程序执行的效率）
  - 数据的序列化和反序列化性能开销很大（包括对象内容和结构）

### DataFrame

DataFrame引入了schema元信息和off-heap

- 优点
  - DataFrame引入off-heap，大量的对象构建直接使用操作系统层面上的内存，不在使用heap堆中的内存，这样一来heap堆中的内存空间就比较充足，不会导致频繁GC，程序的运行效率比较高，它是解决了RDD构建大量的java对象占用了大量heap堆空间，导致频繁的GC这个缺点
  - DataFrame引入schema元信息（数据结构的描述信息），spark程序中的大量对象在进行网络传输的时候，只需要把数据的内容本身进行序列化就可以，数据结构信息可以省略掉。这样一来数据网络传输的数据量是有所减少，数据的序列化和反序列性能开销就不是很大了。它是解决了RDD数据的序列化和反序列性能开销很大这个缺点
- 缺点
  - 编译时类型不安全
  - 不再具有面向对象的编程风格

## 常用操作

### DSL（domain-specific language）风格语法

```scala
// Michael, 29
// Andy, 30
// Justin, 19
case class People(name:String,age:Int)

// RDD[String]
val rdd1 = sc.textFile("/test/spark/people.txt")
// RDD[People]
val rdd2 = rdd1.map(_.split(", ")).map(x=>People(x(0),x(1).toInt))
// DataFrame
val df = rdd2.toDF

// 打印schema
df.printSchema
// 展示数据
df.show
df.select("name")

// $ 是一种方法调用，$"name" 相当于 new ColumnName("name") 的缩写
// 查询某一列
df.select($"name")

// 按年龄分组
df.groupBy("age").count.show

// 按年龄分组，统计结果排序
df.groupBy("age").count.sort($"count".desc).show
```

### SQL（Standard Query Language）风格语法

```scala
// 创建临时表
df.createTempView("people")

// Spark session 调用 sql 方法
spark.sql("select * from people").show
spark.sql("select age, count(*) as count from people group by age").show
```

# DataSet

## 概述

- DataSet是分布式的数据集合，DataSet提供了强类型支持，也是在RDD的每行数据加了类型约束
- DataSet是在Spark1.6中添加的新的接口。它集中了RDD的优点（强类型和可以用强大lambda函数）以及使用了Spark SQL优化的执行引擎
- DataSet包含了DataFrame的功能，Spark2.0中两者统一，DataFrame表示为DataSet[Row]，即DataSet的子集
  - DataSet可以在编译时检查类型
  - 面向对象的编程接口

## 常用操作

```scala
val ds = spark.createDataset(sc.textFile("/test/spark/people.txt"))
// 展示数据
ds.show
```

# 系统集成

##Hive集成

### 本地支持HiveSQL

- Maven依赖

  ```xml
  <dependency>
  	<groupId>org.apache.spark</groupId>
  	<artifactId>spark-hive_2.11</artifactId>
  	<version>2.3.3</version>
  </dependency>
  ```

  

- 自定义设置derby.log、数据仓库数据、元数据位置

  ```scala
  object Config {
    def getLocalConfig(appName: String): SparkConf = {
      // 自定义derby.log位置
      System.setProperty("derby.system.home", "/Users/yangxiaoyu/work/test/sparkdatas/hivelocal")
  
      val sparkConf = new SparkConf()
  
      sparkConf
        .setMaster("local[2]")
  
        .setAppName(appName)
  
        // 自定义数据仓库数据位置
        .set("spark.sql.warehouse.dir", "/Users/yangxiaoyu/work/test/sparkdatas/hivelocal/spark-warehouse")
  
        // 自定义元数据位置
        .set("javax.jdo.option.ConnectionURL", "jdbc:derby:;databaseName=/Users/yangxiaoyu/work/test/sparkdatas/hivelocal/metastore_db;create=true")
  
      sparkConf
    }
  
    def getServerConfig(appName: String): SparkConf = {
      val sparkConf = new SparkConf()
  
      sparkConf.setAppName(appName)
  
      sparkConf
    }
  }
  
  object HiveLocalSupport {
    def main(args: Array[String]): Unit = {
      val config = Config.getLocalConfig(getClass.getName)
  
      val sparkSession = SparkSession.builder().config(config).enableHiveSupport().getOrCreate()
  
      sparkSession.sql("create database if not exists test")
      sparkSession.sql("use test")
      sparkSession.sql("create table if not exists people(id string, name string, age int) row format delimited fields terminated by ' '")
      sparkSession.sql("load data local inpath '/Users/yangxiaoyu/work/test/sparkdatas/people' into table people")
      sparkSession.sql("select * from people").show()
  
      sparkSession.stop()
    }
  }
  ```

### 联动Hive

- 初始化环境

  <font color=red>配置读取Hive元数据</font>

  - node03执行

    将hive的hive-site.xml分发到所有spark的conf目录下（mysql连接信息）

    ```bash
    cp hive-site.xml /bigdata/install/spark-2.3.3-bin-hadoop2.7/conf
    
    scp hive-site.xml node02:$PWD
    scp hive-site.xml node01:$PWD
    ```

  - node01执行

    将mysql驱动分发到所有spark的jars目录下

    ```bash
    cp mysql-connector-java-5.1.38.jar /bigdata/install/spark-2.3.3-bin-hadoop2.7/jars
    
    scp mysql-connector-java-5.1.38.jar node02:$PWD
    scp mysql-connector-java-5.1.38.jar node03:$PWD
    ```

- 启动交互式环境

  <font color=red>配置数据仓库位置 `spark.sql.warehouse.dir` </font>

  ```bash
  spark-sql \
  --master spark://node01:7077 \
  --executor-memory 1g \
  --total-executor-cores 2 \
  --conf spark.sql.warehouse.dir=hdfs://node01:8020/user/hive/warehouse
  ```

  运行Hive语句

  ```mysql
  select * from myhive.score;
  ```

- 提交脚本

  ```bash
  spark-sql \
  --master spark://node01:7077 \
  --executor-memory 1g \
  --total-executor-cores 2 \
  --conf spark.sql.warehouse.dir=hdfs://node01:8020/user/hive/warehouse \
  -e 'select * from myhive.score;'
  ```

  

## MySQL集成

- Maven依赖 MySQL JDBC 驱动

  ```xml
  <dependency>
  	<groupId>mysql</groupId>
  	<artifactId>mysql-connector-java</artifactId>
  	<version>5.1.38</version>
    <scope>runtime</scope>
  </dependency>
  ```

  

- 登录Mysql创建测试数据

  - 启动mysql服务

    ```bash
    # 启动服务
    systemctl start mysqld.service
    
    # 登录
    mysql -uroot -proot
    ```

    

  - 创建mysql测试数据

    ```mysql
    -- 创建数据库
    create database spark;
    
    -- 创建表
    use spark;
    create table user(id int, name varchar(40), age int);
    
    -- 插入数据
    insert into user(id, name, age) values(1,'yoyo',37);
    insert into user(id, name, age) values(2,'lucky',3);
    insert into user(id, name, age) values(2,'rain',38);
    ```

    

  - SparkSQL通过JDBC加载MySQL数据库表数据到DataFrame，利用Spark分布式计算框架的优势，对数据计算之后将结果写回到MySQL数据库表中

    ```scala
    object ReadWrite {
      def main(args: Array[String]): Unit = {
        val url = "jdbc:mysql://node03:3306/spark"
    
        val readTableName = "user"
        val writeTableName = "newuser"
    
        val properties = new Properties()
        properties.setProperty("user", "root")
        properties.setProperty("password", "root")
    
        // val config = Config.getLocalConfig(getClass.getName)
        val config = Config.getServerConfig(getClass.getName)
    
        val sparkSession = SparkSession.builder().config(config).getOrCreate()
    
        // 读取数据
        val readDF = sparkSession.read.jdbc(url, readTableName, properties)
        // readDF.printSchema()
        // readDF.show()
    
        // 创建临时表
        readDF.createTempView("user")
    
        // 处理数据
        val writeDF = sparkSession.sql("select id, name, age, age-10 as newhope from user")
    
        // 写入数据
        // mode
        // overwrite  表示覆盖，如果表不存在，创建
        // append     表示追加，如果表不存在，创建
        // ignore     表示忽略，如果表存在，不进行任何操作
        // error      如果表存在，报错（默认选项）
        writeDF.write.mode("overwrite").jdbc(url, writeTableName, properties)
    
        sparkSession.stop()
      }
    }
    ```

    

  - 集群运行

    ```bash
    # --driver-class-path 指定 Driver 端所需要的jar
    # --jars 指定 Executor 端所需要的jar
    spark-submit --master spark://node01:7077 \
    --executor-memory 1g --total-executor-cores 2 \
    --class com.sciatta.hadoop.spark.example.sql.mysql.ReadWrite \
    --driver-class-path /bigdata/soft/mysql-connector-java-5.1.38.jar \
    --jars /bigdata/soft/mysql-connector-java-5.1.38.jar \
    hadoop-spark-example-1.0-SNAPSHOT.jar
    ```





