# 关系数据库

## 数据库设计范式

- 第一范式（1NF）关系 R 属于第一范式，当且仅当R中的每一个属性A的值域只包含原子项

  **消除重复数据**，即每一列都是不可再分的基本数据项；每个列都是原子的。

- 第二范式（2NF）在满足 1NF 的基础上，消除非主属性对码的部分函数依赖

  **消除部分依赖**，表中没有列只与主键的部分相关，即每一行都被主键唯一标识；每个列都有主键。

- 第三范式（3NF）在满足 2NF 的基础上，消除非主属性对码的传递函数依赖

  **消除传递依赖**，消除表中列不依赖主键，而是依赖表中的非主键列的情况，即没有列是与主键不相关的。从表只引用主表的主键，即表中每列都和主键相关。

- BC 范式（BCNF）在满足 3NF 的基础上，消除主属性对码的部分和传递函数依赖

- 第四范式（4NF）消除非平凡的多值依赖

- 第五范式（5NF）消除一些不合适的连接依赖



## 结构化查询语言

- 数据查询语言（DQL Data Query Language）其语句也称为“数据检索语句”，用以从表中获得数据，确定数据怎样在应用程序给出。保留字 SELECT 是 DQL（也是所有 SQL）用得最多的动词，其他 DQL 常用的保留字有 WHERE，ORDER BY，GROUP BY 和 HAVING。这些 DQL 保留字常与其它类型的 SQL 语句一起使用。
- 数据操作语言（DML Data Manipulation Language）其语句包括动词 INSERT、UPDATE 和 DELETE。它们分别用于添加、修改和删除。
- 事务控制语言（TCL）它的语句能确保被 DML 语句影响的表的所有行及时得以更新。包括COMMIT（提交）命令、SAVEPOINT（保存点）命令、ROLLBACK（回滚）命令。
- 数据控制语言（DCL）它的语句通过 GRANT 或 REVOKE 实现权限控制，确定单个用户和用户组对数据库对象的访问。某些 RDBMS 可用 GRANT 或 REVOKE 控制对表单个列的访问。
- 数据定义语言（DDL）其语句包括动词 CREATE, ALTER 和 DROP。在数据库中创建新表或修改、删除表（CREAT TABLE 或 DROP TABLE）；为表加入索引等。
- 指针控制语言（CCL）它的语句像 DECLARE CURSOR，FETCH INTO 和 UPDATE WHERE CURRENT 用于对一个或多个表单独行的操作。



## 数据库设计优化

- 如何恰当选择引擎

  myisam 适用于对读写性能要求高，但对数据一致性要求低的场景

  innodb 支持行锁，事务

  memory 临时表

- 库表如何命名

  有意义

- 如何选择恰当数据类型

  **明确、尽量小**

-  文件、图片是否要存入到数据库

  建议存放结构化数据；读多写少的数据也可

- 时间日期的存储问题

  存放时间戳，int或long类型或string类型

- 数值的精度问题

  字符串存放，应用负责转换

- 是否使用外键、触发器

  不建议，应用保证。性能高

- 是否可以冗余字段

  高效查询可以适当冗余

- 是否使用游标、变量、视图、自定义函数、存储过程

  不建议，维护、调试成本高，移植性差

- 自增主键的使用问题

  分布式不建议

- 能够在线修改表结构（DDL 操作）

  不建议，会锁表。备份也会锁表，尽量在无业务处理时操作

- 逻辑删除还是物理删除

  重要数据需要审计，增加字段表示逻辑删除

- 要不要加 create_time，update_time 时间戳

  建议，增量复制

- 数据库碎片问题

  插入和删除都会导致碎片，优化压缩

- 如何快速导入导出、备份数据

  批量导入导出内建命令

  批量导入时先删除索引、外键，导入数据后，再添加



## SQL语句的执行顺序

### 语法

```mysql
SELECT
    [ALL | DISTINCT | DISTINCTROW ]
    [HIGH_PRIORITY]
    [STRAIGHT_JOIN]
    [SQL_SMALL_RESULT] [SQL_BIG_RESULT] [SQL_BUFFER_RESULT]
    [SQL_CACHE | SQL_NO_CACHE] [SQL_CALC_FOUND_ROWS]
    select_expr [, select_expr] ...
    [into_option]
    [FROM table_references
      [PARTITION partition_list]]
    [WHERE where_condition]
    [GROUP BY {col_name | expr | position}
      [ASC | DESC], ... [WITH ROLLUP]]
    [HAVING where_condition]
    [ORDER BY {col_name | expr | position}
      [ASC | DESC], ...]
    [LIMIT {[offset,] row_count | row_count OFFSET offset}]
    [PROCEDURE procedure_name(argument_list)]
    [into_option]
    [FOR UPDATE | LOCK IN SHARE MODE]

into_option: {
    INTO OUTFILE 'file_name'
        [CHARACTER SET charset_name]
        export_options
  | INTO DUMPFILE 'file_name'
  | INTO var_name [, var_name] ...
}
```

### 执行顺序

1. from
2. on
   - 先on过滤条件之后，才会join生成临时表
3. join
4. where
   - 临时表生成之后，根据限制条件从临时表中筛选
   - 在分组（聚集函数）之前筛选数据
5. group by
   - 分组之后，执行聚集函数
6. having
   - 聚合函数执行之后对分组数据进一步筛选，同group by一起使用，不可单独使用
7. select
   - 如果有group by使用的话，select查询的字段可能是group by后跟的分组字段，也有可能是对字段聚合函数计算的结果
8. distinct
9. order by
   - group by和orderby可以实现组内排序，即 `group by A,B order by A,B`
10. limit



# MySQL

## 存储

### 独占模式

公共数据文件

- 日志组文件：ib_logfile0和ib_logfile1（redo log）
- 二进制日志文件，记录主数据库服务器的 DDL 和 DML 操作：默认文件名为 “主机名-bin.num”

- 二进制日志索引文件：主机名-bin.index

在MySQL数据目录下，每一个数据库都是一个文件目录

- 表结构文件：*.frm

- 独占表空间文件：*.ibd

- 字符集和排序规则文件：db.opt

### 共享模式

innodb_file_per_table=1

- 数据都在 ibdata1中



## 执行流程

update记录执行流程

![mysql_update_table_flow](MySQL实践.assets/mysql_update_table_flow.png)



- 连接器：管理连接，权限验证
- 分析器：词法分析、语法分析
- 优化器：索引确定，执行计划生成
- 执行器：和引擎交互，返回结果

- undo log：用于回滚，崩溃恢复，MVCC；存储回滚段指针和事务id，通过回滚段指针找到对应undo log记录，通过事务id判断记录的可见性
- 记录所在页存在于内存中
  - 存在
    - 唯一索引：找到数据，判断数据冲突与否，更新内存
    - 普通索引：找到数据，更新内存
  - 不存在
    - 唯一索引：将数据页从磁盘读入内存，判断数据冲突与否，更新内存
    - 普通索引：在change buffer更新记录，change buffer异步将更新同步到磁盘，通过change buffer降低磁盘IO次数
- redo log：WAL 用于事务崩溃恢复，以及将随机写变成顺序写，提过性能
- binlog：用于备份，主从同步
- 刷写 redo log：处于commit-prepare阶段
- 刷写 binlog：处于commit-commit阶段



## 执行引擎

| 存储引擎 | 存储限制 | 事务 | 索引 | 锁的粒度 | 数据压缩 | 外键 |
| -------- | -------- | ---- | ---- | -------- | -------- | ---- |
| myisam   | 256TB    | -    | 支持 | 表锁     | 支持     | -    |
| innodb   | 64TB     | 支持 | 支持 | 行锁     | -        | 支持 |
| memory   | 有       | -    | 支持 | 表锁     | -        | -    |
| archive  | 无       | -    | -    | 行锁     | 支持     | -    |

- myisam 适用于对读写性能要求高，但对数据一致性要求低的场景。



## 索引

### 语法

```mysql
CREATE [UNIQUE|FULLTEXT|SPATIAL] INDEX index_name [USING index_type] ON tbl_name (index_col_name,...)
-- index_col_name：col_name [(length)] [ASC | DESC]
-- index_type：存储引擎MyISAM允许的索引类型BTREE，存储引擎InnoDB允许的索引类型BTREE，存储引擎MEMORY/HEAP允许的索引类型HASH,BTREE
-- 普通索引：创建索引时，不附加任何限制条件(唯一、非空等限制)。该类型可以创建在任何数据类型的字段上
-- UNIQUE唯一索引：创建索引时限制索引的值必须是唯一的，通过该类型的索引可以更快速的查询某条记录，唯一索引字段不可加长度
-- FULLTEXT全文索引：主要关联在数据类型为char、varchar、text的字段上，以便能够更加快速的查询数据量较大的字符串类型的字段，必须加上长度
-- 多列索引：是指在创建索引时，所关联的字段不是一个字段，而是多个字段。虽然可以通过所关联的字段进行查询，但是只有查询条件中使用了所关联字段中的第一个字段，多列索引才会被使用
```



### 原理

数据是**按页来分块**的，假设当一个数据被用到时，其附近的数据通常也会被马上使用。

InnoDB使用B+Tree实现聚簇索引。

B+Tree 所有叶子节点才有指向数据的指针。非叶子节点就是纯索引数据和主键。每个叶子节点都有指向下一个叶子节点的连接。<font color=red>非叶子节点存放在内存中</font>，B-Tree中每个节点包含的数据的指针会带来额外的内存占用，减少了放入内存的非叶子节点数；B+Tree则尽可能多地将非叶子节点放入内存。

- 主键索引（聚簇索引）：<font color=red>叶子节点存放的是key值和数据，存放在磁盘上</font>，叶子节点加载到内存后，数据一起加载，即找到叶子节点的key，就找到了数据；
  - innodb一定存在聚簇索引，默认以主键作为聚簇索引
  - 没有主键时，会用一个唯一且不为空的索引列做为主键，成为此表的聚簇索引；如果没有这样的索引，InnoDB会隐式定义一个主键来作为聚簇索引
- 辅助索引（非聚簇索引）：叶子节点存放的是key值（索引字段的值）和对应记录的主键值，<font color=red>辅助索引的叶子节点仍然是索引</font>，使用辅助索引查询，首先检索辅助索引获取主键，然后用主键在主键索引中检索获取记录。
  - 每建立一个非聚簇索引，会根据索引字段生成一颗新的B+树。因此每加一个索引，就会增加表的体积， 占用磁盘存储空间
  - 当执行 `select col from table where col = ?` col上有索引的时候，效率比执行 `select * from table where col = ?` 速度要快很多，因为只需要在非聚簇索引检索即可
  - 多加一个索引，就会多生成一颗非聚簇索引树。因此在做插入时，需要同时维护N颗树的变化，如果索引太多，插入性能就会下降



**单表不建议超过2000万数据。**

InnoDB存储引擎最小存储单元页（Page），一个页的大小是16K。idb文件大小始终是16的倍数。

- 存放数据：记录按主键排序存放到不同的页中
- 存放<font color=red>键值+指针</font>

在B+Tree中叶子节点存放数据，非叶子节点存放键值+指针。每个节点是一页。索引组织表通过非叶子节点的二分查找法以及指针确定数据在哪个页中，进而在去数据页中查找到需要的数据。每张表的**根页**位置在表空间文件中是固定的，即page number=3的页。

假设：每页大小16K，每条记录占1K大小，主键为Bigint类型占8字节，指针占6字节。

- 每一页约存放 ( 16 * 1024 ) / ( 8 + 6 ) = 1170 指向数据页的指针。
- 数据页存放16/1=16条记录

B+Tree高为2，数据记录条数 1170 * 16 = 18720

B+Tree高为3，数据记录条数 1170 * 1170 * 16 = 21902400

所以在InnoDB中B+树高度一般为1-3层，它就能满足千万级的数据存储。在查找数据时一次页的查找代表一次IO， 所以通过主键索引查询通常只需要1-3次IO操作即可查找到数据。



## 事务

<font color=red>解决并发问题。</font>

- 问题

  - 脏读(dirty read)：使用到从未被确认的数据(例如：早期版本、回滚)
  - 不可重复读：其他事务 update 或 delete 会对结果集有影响
  - 幻读(Phantom)：相同的查询语句，在不同的时间点执行时产生不同的结果集

- 解决

  提高隔离级别、使用间隙锁或临键锁

### 事务可靠性模型 ACID

- Atomicity

  原子性，一次事务中的操作要么全部成功，要么全部失败

- Consistency

  一致性，跨表、跨行、跨事务，数据库始终保持一致状态

- Isolation

  隔离性，可见性，保护事务不会互相干扰，包含4种隔离级别

- Durability

  持久性，事务提交成功后，不会丢数据。如电源故障，系统崩溃

### 锁

加锁采用当前读，不加锁采用快照读。

- myisam 表级锁

  - 共享锁(S锁)：假设<font color=red>事务T1</font>对数据A加上共享锁，那么<font color=red>事务T2</font>可以读数据A，不能修改数据A
  - 排他锁(X锁)：假设事务T1对数据A加上排他锁，那么事务T2不能读数据A，不能修改数据A
  - 意向共享锁(IS锁)：一个事务在获取（任何一行/或者全表）S锁之前，一定会先在所在的表上加IS锁
  - 意向排他锁(IX锁)：一个事务在获取（任何一行/或者全表）X锁之前，一定会先在所在的表上加IX锁

  意向锁的目的是 `LOCK TABLE xxx WRITE/READ` 表级锁的请求直接根据意向锁来判断是否存在表冲突。

  - LOCK TABLES

    1. DDL

    2. dump

    3. LOCK TABLES xxx WRITE/READ;（锁表）、UNLOCK TABLES;（解锁表）、LOCK TABLES xxx WRITE/READ, xxx WRITE/READ;（锁多张表）

       ```mysql
       drop table if exists lockdb.t1;
       create table if not exists lockdb.t1(id int unsigned not null);
       
       drop table if exists lockdb.t2;
       create table if not exists lockdb.t2(id int unsigned not null);
       ```

       - session1：lock tables t1 read;
         - session1 对 t1（范围内） 只能 select，异常 insert/delete/update
         - session1 对 t2（范围外） 异常 insert/delete/update/select
         - session2 对 t1 只能 select，阻塞 insert/delete/update
         - session2 对 t2 正常 insert/delete/update/select
       - session1：lock tables t1 write;
         - session1 对 t1（范围内） 正常 insert/delete/update/select 
         - session1 对 t2（范围外） 异常 insert/delete/update/select
         - session2 对 t1 阻塞 insert/delete/update/select
         - session2 对 t2 正常 insert/delete/update/select

- innodb 行级锁

  - <font color=red>记录锁(Record)</font>：始终锁定索引记录，**锁加在索引上，而不是行上**，innodb一定存在聚簇索引，因此行锁会落在聚簇索引上。
    - `select * from table where id = ? lock in share mode;` 读取记录加S锁
    - `select * from table where id = ? for update` 读取记录加X锁
  - <font color=red>间隙锁(Gap)</font>：对索引的间隙加锁，**目的是为了防止其他事务插入数据**；其中，READ UNCOMMITTED 和 READ COMMITTED 不会出现间隙锁；REPEATABLE READ 和 SERIALIZABLE 会出现间隙锁。
  - <font color=red>临键锁(Next-Key)</font>：记录锁+间隙锁的组合
  - 谓词锁(Predicat)：空间索引

### 隔离级别

<font  color=red>平衡一致性和性能。</font>

- 读未提交：READ UNCOMMITTED

  - <font color=blue>很少使用</font>，不能保证一致性

  - 脏读(dirty read) ：使用到从未被确认的数据。例如：早期版本、回滚

  - 锁

    - 以非锁定方式执行
    - 可能的问题：脏读、不可重复读、幻读

  - 演示

    ```mysql
    -- 创建数据库
    create database if not exists lockdb;
    use lockdb;
    
    -- 创建表
    drop table if exists lockdb.locks;
    create table if not exists lockdb.locks
    (
      id int unsigned not null,
      name int unsigned,
      num int unsigned,
      primary key(id)
    ) ENGINE = InnoDB
      DEFAULT CHARSET = utf8;
    create index idx_name on locks(name); -- 普通索引
    
    insert into locks(id, name, num) values(1,100,10);
    insert into locks(id, name, num) values(2,400,20);
    insert into locks(id, name, num) values(3,400,30);
    insert into locks(id, name, num) values(8,800,20);
    
    -- 显示表结构
    -- 或 show columns from locks;
    desc locks;
    
    -- 显示表数据，按列显示
    select * from locks\G;
    
    -- 查询全局的事务隔离级别
    SELECT @@global.tx_isolation;
    
    -- 查询当前会话的事务级别
    SELECT @@session.tx_isolation;
    
    -- 设置隔离级别
    -- SET [SESSION | GLOBAL] TRANSACTION ISOLATION LEVEL {READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE}
    ```

    - 没有间隙锁，只有记录锁

    ```mysql
    -- 设置隔离级别
    set session transaction isolation level read uncommitted;
    
    -- 所有测试需要加事务 begin; commit; rollback;
    
    -- 聚簇索引	
    -- 快照读
    begin; select * from locks where id=1;
    -- 快照读
    begin; select * from locks where id>2;
    
    -- 当前读
    -- t1 1 获得S锁
    -- t2 1 可以获得S锁，写会阻塞（本质是要获得X锁）
    begin; select * from locks where id=1 lock in share mode;
    
    -- 当前读
    -- t1 3 8 获得S锁
    -- t2 3 8 可以获得S锁，写会阻塞，其他记录没有限制
    begin; select * from locks where id>2 lock in share mode;
    
    -- 当前读
    -- t1 1 获得X锁
    -- t2 1 不可以获得S锁，不可以获得X锁
    begin; select * from locks where id=1 for update;
    
    -- t1 3 8 获得X锁
    -- t2 3 8 不可以获得S锁，不可以获得X锁
    begin; select * from locks where id>2 for update;
    
    
    -- 非聚簇索引：锁 非聚簇索引+聚簇索引
    
    
    -- 非索引：通过聚簇索引进行全表扫描，全表加锁，不符合条件的立即释放锁
    ```

    

- 读已提交：READ COMMITTED

  - 每次查询都会设置和读取自己的新快照

  - 仅支持基于行的 bin-log

  - UPDATE 优化：半一致读(semi-consistent read)

  - 不可重复读：不加锁的情况下，其他事务 UPDATE 或 DELETE 会对查询结果有影响

  - 幻读(Phantom)：加锁后，不锁定间隙，其他事务可以 INSERT

  - 锁

    - 锁定索引记录，而不锁定记录之间的间隙
    - 可能的问题：不可重复读、幻读

  - 演示

    ```mysql
    -- 设置隔离级别
    set session transaction isolation level read committed;
    
    -- 可以解决脏读
    ```

    

- 可重复读：REPEATABLE READ

  - <font color=red>InnoDB 的默认隔离级别</font>

  - 使用事务第一次读取时创建的快照

  - 多版本技术

  - 锁

    - 间隙锁因为可以防止插入数据，因此可以部分解决幻读问题
    - 使用唯一索引的唯一查询条件时，只锁定查找到的索引记录，不锁定间隙
    - 其他查询条件，会锁定扫描到的索引范围，通过间隙锁或临键锁来阻止其他会话在这个范围中插入值
    - 可能的问题：幻读

  - 演示

    - 聚簇索引：当锁定精确记录时，索引上只有记录锁；当锁定范围时，索引上除了记录锁，索引间还会有间隙锁
    - 非聚簇索引
      - 唯一索引：聚簇索引和非聚簇索引上都会有记录锁，对于锁定范围时，非聚簇索引上会有间隙锁
      - 普通索引：聚簇索引和非聚簇索引上都会有记录锁，对于精确记录或范围，非聚簇索引上会有间隙锁
    - 非索引：对于精确记录或范围，所有记录的聚簇索引会有记录锁，所有范围会有间隙锁，**相当于锁表**
    - 在一个事务内对表作update/delete**范围**操作，查询条件是索引，会加记录锁+间隙锁（<font color=red>~~禁止~~</font>），<font color=red>要尽量缩小范围并在索引上查询</font>

    ```mysql
    # 设置隔离级别
    set session transaction isolation level REPEATABLE READ;
    
    -- 聚簇索引	
    -- 快照读
    begin; select * from locks where id=1;
    -- 快照读
    begin; select * from locks where id>2;
    
    -- 当前读
    -- t1 1 获得S锁
    -- t2 1 可以获得S锁，不可以获得X锁
    begin; select * from locks where id=1 lock in share mode;
    
    -- 当前读
    -- t1 3 8 获得S锁
    -- t2 3 8 可以获得S锁，不可以获得X锁(update/delete)，在(2,3) (3,8) (8,+∞)存在间隙锁(insert)  ==》  解决不可重复读
    begin; select * from locks where id>2 lock in share mode;
    
    -- 当前读
    -- t1 1 获得X锁
    -- t2 1 不可以获得S锁，不可以获得X锁
    begin; select * from locks where id=1 for update;
    
    -- t1 3 8 获得X锁
    -- t2 3 8 不可以获得S锁，不可以获得X锁(update/delete)，在(2,3) (3,8) (8,+∞)存在间隙锁(insert)
    begin; select * from locks where id>2 for update;
    
    -- 对不存在记录加X锁，(3,8) 之间存在间隙锁
    begin;select * from locks where id=6 for update;
    -- (8,+∞)存在间隙锁
    begin;select * from locks where id>10 for update;
    
    
    -- 非聚簇索引
    -- 1、唯一索引：锁 非聚簇索引+聚簇索引
    -- 2、普通索引：
    -- 400和2、3 +X锁， (100,400) (400,800) +间隙锁
    begin;select * from locks where name=400 for update;
    -- 不存在记录 (400,800) +间隙锁
    begin;select * from locks where name=600 for update;
    -- 不存在记录 (800,+∞) +间隙锁
    begin;select * from locks where name>900 for update;
    
    
    -- 非索引
    -- 1 2 3 8 +S锁
    -- (-∞,1) (1,2) (2,3) (3,8) (8,+∞) +间隙锁
    begin;select * from locks where num=20 for update;
    ```

    

- 可串行化：SERIALIZABLE

  - 最严格的级别，事务串行执行，资源消耗最大

### 支撑

- undo log 撤销日志
  - <font color=red>保证事务的原子性</font>
  - 用处：事务回滚，一致性读、崩溃恢复
  - 记录事务回滚时所需的撤消操作
  - 一条 INSERT 语句，对应一条 DELETE 的 undo log
  - 每个 UPDATE 语句，对应一条相反 UPDATE 的 undo log
  - 保存位置
    - system tablespace (MySQL 5.7默认)
    - undo tablespaces (MySQL 8.0默认)

- redo log 重做日志
  - <font color=red>确保事务的持久性</font>，防止事务提交后数据未刷新到磁盘就掉电或崩溃
  - 事务执行过程中写入 redo log，记录事务对数据页做了哪些修改
  - 提升性能：WAL(Write-Ahead Logging) 技术，先写日志（顺序写提供性能），再写磁盘
  - 日志文件：ib_logfile0，ib_logfile1
  - 日志缓冲：innodb_log_buffer_size
  - 强刷：fsync()

- MVCC 多版本并发控制
  - 使 InnoDB 支持一致性读：READ COMMITTED 和 REPEATABLE READ
  - 让查询不被阻塞、无需等待被其他事务持有的锁，这种技术手段可以增加并发性能
  - InnoDB 保留被修改行的旧版本
  - 查询正在被其他事务更新的数据时，会读取更新之前的版本
  - 每行数据都存在一个版本号，每次更新时都更新该版本
  - 这种技术在数据库领域的使用并不普遍。 某些数据库，以及某些 MySQL 存储引擎都不支持
  - 实现机制
    - 隐藏列
      - DB_TRX_ID | 6-byte | 指示最后插入或更新该行的事务 ID
      - DB_ROLL_PTR | 7-byte | 回滚指针。指向回滚段中写入的undo log 记录
      - DB_ROW_ID | 6-byte | 聚簇 row ID/聚簇索引
    - 事务链表，保存还未提交的事务，事务提交则会从链表中摘除
    - Read view：每个 SQL 一个，包括 rw_trx_ids，low_limit_id，up_limit_id，low_limit_no 等
    - 回滚段：通过 undo log 动态构建旧版本数据


