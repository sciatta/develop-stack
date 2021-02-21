# 缓存技术

缓存的本质，系统各级处理速度不匹配，导致利用<font color=red>空间换时间</font>。缓存是提升系统性能的一个简单有效的办法。



## 数据分类和使用频率

数据分类

- 静态数据：一般不变，类似于字典表 

- 准静态数据：变化频率很低，部门结构设置，全国行政区划数据等 

- 中间状态数据：一些计算的可复用中间数据，变量副本，配置中心的本地副本 



这些数据适合于使用缓存的方式访问 

- 热数据：使用频率高
- 读写比较大：读的频率 >> 写的频率

广义上来说，为了加速数据处理，让业务更快访问的**临时存放冗余数据**，都是缓存。狭义上，现在我们一般在分布式系统里把缓存到内存的数据叫做内存缓存。



## 缓存加载时机

- 启动全量加载

  全局有效，使用简单 

- 懒加载
  - 同步使用加载
    - 先看缓存是否有数据，有的话直接返回，没有的话从数据库读取 
    - 读取的数据，先放到内存，然后返回给调用方 
  - 延迟异步加载
    - 从缓存获取数据，不管是否为空直接返回
    - 策略一（异步）：如果为空，则主动发起一个异步加载的线程，负责加载数据 
    - 策略二（解耦）：后台异步线程负责维护缓存的数据，定期或根据条件触发更新



## 缓存的有效性

- 读写比

  对数据的**写操作**导致数据变动，意味着维护成本。N : 1

- 命中率

  命中缓存意味着缓存数据被使用，意味着有价值。90%+



## 缓存使用不当导致的问题

- 系统预热导致启动慢

  试想一下，一个系统启动需要预热半个小时。 导致系统不能做到快速应对故障宕机等问题。 

- 系统内存资源耗尽 

  只加入数据，不能清理旧数据。旧数据处理不及时，或者不能有效识别无用数据。




## 缓存常见问题

- 缓存穿透（恶意）

  大量并发查询不存在的key，导致直接将压力透传到数据库

  解决办法

  1. 缓存空值的key
  2. Bloom过滤或RoaringBitmap判断key是否存在
  3. 完全以缓存为准，使用延迟异步加载策略

- 缓存击穿（偶然）

  某个key失效的时候，正好有大量并发请求访问这个key

  解决办法

  1. key的更新操作添加全局互斥锁
  2. 完全以缓存为准，使用延迟异步加载策略

- 缓存雪崩

  当某一时刻发生大规模缓存失效的情况，会有大量的请求直接打到数据库，导致数据库压力过大甚至宕机。

  - 更新策略
- 数据热点
  - 缓存服务宕机
  
  解决办法
  
  1. 更新策略在时间上做到比较均匀
  2. 使用的热数据尽量分散到不同的机器上
  3. 多台机器做主从复制或多副本，实现高可用
  4. 实现熔断限流机制，对系统进行负载能力控制



# Redis

## 安装

```shell
# 下载最新镜像
docker pull redis

# 查看redis版本 -i 忽略大小写
# docker image inspect redis | grep -i 'version'

# 创建容器并运行
docker run -itd --name redis-test -p 6379:6379 redis

# 显示容器运行时的完整命令
# 	docker-entrypoint.sh redis-server
#		默认行为 Entrypoint + Cmd，其中Cmd可以被docker run命令中image后的CMD参数替换
# docker ps --no-trunc

# 进入容器
docker exec -it redis-test /bin/bash

# 进入redis命令交互模式
redis-cli

# 性能测试
# redis-benchmark -n 100000 -c 32 -t SET,GET,INCR,HSET,LPUSH,MSET -q
```



## 常用命令

- ttl key

  以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)

- flushdb

  清空当前数据库中的所有 key

- dbsize

  返回当前数据库的 key 的数量

- info

  获取 Redis 服务器的各种信息和统计数值

- bgsave

  在后台异步保存当前数据库的数据到磁盘

- save

  同步保存数据到硬盘

- shutdown [nosave] [save]

  异步保存数据到硬盘，并关闭服务器

- time

  返回当前服务器时间

- config get parameter

  获取指定配置参数的值

- config rewrite

  对启动 Redis 服务器时所指定的 redis.conf 配置文件进行改写

- config set parameter value

  修改 redis 配置参数，无需重启

- role

  返回主从实例所属的角色

- slaveof host port

  将当前服务器转变为指定服务器的从属服务器(slave server)



## 基本数据结构

### 字符串（string）

可表示三种类型：int、string、byte[] 

字符串类型是Redis中最为基础的数据存储类型，它在Redis中是**二进制**安全的，这便意味着该类型可以接受任何格式的数据，如JPEG图像数据或json对象描述信息等。在Redis中字符串类型的value最多可以容纳的数据长度是512M

`select 0 选择0数据库，默认；共有 0~15 16个数据库`

- set 
- mset：同时设置多个key value
- get
- mget：同时获取多个key
- <font color=red>keys</font>：`keys *` 可以查看所有key；* 代表任意个字符，? 代表任意一个字符
- getset：原子操作，设置新值，返回原始值
- del
- exists：存在key返回1，不存在返回0
- append：追加新值到原值的末尾

- incr：integer类型自增1
- decr
- incrby：integer类型增加x，如 `INCRBY test_i 3`
- decrby

注意

- 字符串append：会使用更多的内存，为了提高效率会开辟一块额外未使用的内存空间

- 整数共享：如能使用整数，就尽量使用整数，整数会共享；当限制了redis内存+LRU时，可能会使整数共享失效

- 整数精度问题：redis大概能保证16位精度，当17-18位的大整数时就会丢失精度



### 散列（hash）

可表示：Map、Pojo Class 

Redis中的Hash类型可以看成具有String key 和String value的map容器。所以该类型非常适合于存储对象的信息。如Username、password和age。如果Hash中包含少量的字段，那么该类型的数据也将仅占用很少的磁盘空间。 Redis 中每个 hash 可以存储 2^32 - 1 键值对（40多亿）。

- hset
- hget：需要指定key
- hmset
- hmget
- hgetall：获取所有key和value
- hdel
- hincrby 
- hexists
- hlen
- hkeys：获取所有key
- hvals：获取所有value



### 列表（list）

可表示：LinkedList 

在Redis中，List类型是按照插入顺序排序的字符串链表。和数据结构中的普通链表一样，我们可以在其头部(Left)和尾部(Right)添加新的元素。

- lpush：在左侧插入元素
- lpop：弹出最左侧元素
- rpush：在右侧插入元素
- rpop：弹出最右侧元素
- lrange：获取范围元素，从最左侧开始编号，0为开始元素，后续元素序号递增1；如果需要获取所有元素可以使用 `0 -1` 作为范围查询条件



### 集合（set）

可表示：set，不重复的list 

在redis中，可以将Set类型看作是没有排序的字符集合，和List类型一样，我们也可以在该类型的数值上执行添加、删除和判断某一元素是否存在等操作。Redis 中集合是通过哈希表实现的，这些操作的时间复杂度为O(1)，即常量时间内完成依次操作。 和List类型不同的是，Set集合中不允许出现重复的元素。 

- sadd：向集合中插入元素，成功返回1，失败返回0
- srem：删除集合中的第一个元素
- smembers：查询所有元素
- SPOP：随机返回元素并删除
- SRANDMEMBER：随机返回元素，可以指定个数，如 `SRANDMEMBER sk 2` 随机返回集合中的两个元素
- sismember：判断元素是否在集合中存在
- sinter：求多个集合的**交集**
- sunion：求多个集合的**并集**
- sdiff：求多个集合的**差集**



### 有序集合（sorted set） 

sortedset和set极为相似，他们都是字符串的集合，都不允许重复的成员出现在一个 set中。他们之间的主要差别是sortedset中每一个成员都会有一个分数与之关联。redis正是通过分数来为集合的成员进行从小到大的排序。sortedset中分数是可以重复的。 

- zadd `zadd ss 99 a 98 b`
- ZRANGE `zrange ss 0 -1 withscores` 范围查询，按score从小到大排序，withscores携带权重
- ZREVRANGE 范围查询，按score从大到小排序
- zscore 获取集合中某一个成员的score
- zcard  获取集合中成员数量 
- zrem 移除集合中指定的成员，可以指定多个成员 
- ZREMRANGEBYRANK 按照排名范围删除元素（从小到大）



## 高级数据结构

### Bitmaps

setbit/getbit/bitop/bitcount/bitpos 

bitmaps不是一个真实的数据结构。而是String类型上的一组面向bit操作的集合。由于 strings是二进制安全的blob，并且它们的最大长度是512m，所以bitmaps能最大设置2^32个不同的bit。 



### Hyperloglogs

pfadd/pfcount/pfmerge 

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定的、并且是很小的。在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

- 基数是一种算法。举个例子，一本英文著作由数百万个单词组成，你的内存却不足以存储它们，那么我们先分析一下业务。英文单词本身是有限的，在 这本书的几百万个单词中有许多重复单词，扣去重复的单词，这本书中也就是几千到一万多个单词而已，那么内存就足够存储它们了。比如数字集合{1，2，5，7，9，1，5，9}的基数集合为{1，2，5，7，9}，那么基数（不重复元素）就是5，基数的作用是评估大约需要准备多少个存储单元去存储数据， 但是基数的算法一般会存在一定的误差（一般是可控的）。Redis对基数数据结构的支持是从版本2.8.9开始的。



###GEO

geoadd/geohash/geopos/geodist/georadius/georadiusbymember 

Redis的GEO特性在 Redis3.2版本中推出，这个功能可以将用户给定的地理位置（经度和纬度）信息储存起来，并对这些信息进行操作。



## 线程

- IO线程
  - redis 6之前（2020年5月），单线程 
  - redis 6之后，多线程，NIO模型（主要的性能提升点）

- 内存处理线程
  - 单线程（高性能的核心）



## 事务

Redis 事务可以一次执行多个命令， 并且带有以下三个重要的保证：

- 批量操作在发送 EXEC 命令前被放入队列缓存。
- 收到 EXEC 命令后进入事务执行，**事务中任意命令执行失败，其余的命令依然被执行**。
- 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。

一个事务从开始到执行会经历以下三个阶段：

- 开始事务。
- 命令入队。
- 执行事务。

单个 Redis 命令的执行是原子性的，但 Redis 没有在事务上增加任何维持原子性的机制，所以 <font color=red>Redis 事务的执行并不是原子性的</font>。事务可以理解为一个打包的批量执行脚本，但批量指令并非原子化的操作，中间某条指令的失败不会导致前面已做指令的回滚，也不会造成后续的指令不做。

- 执行事务

  ```shell
  # 开启事务 
  multi
  sadd a 1
  sadd a 2
  # 失败，但不影响其他语句的执行
  get a
  sadd a 3
  # 提交事务
  exec
  
  # DISCARD 取消事务，放弃执行事务块内的所有命令
  ```

- 监听

  在 Redis 中使用 watch 命令可以决定事务是执行还是回滚。一般而言，可以在 multi 命令之前使用 watch 命令监控某些键值对，然后使用 multi 命令开启事务，执行各类对数据结构进行操作的命令，这个时候这些命令就会进入队列。当 Redis 使用 exec 命令执行事务的时候，它首先会去比对被 watch 命令所监控的键值对，如果没有发生变化，那么它会执行事务队列中的命令，提交事务；如果发生变化，那么它不会执行任何事务中的命令，而去事务回滚。无论事务是否回滚，Redis 都会去取消执行事务前的 watch 命令。

  Redis 参考了多线程中使用的 CAS（比较与交换，Compare And Swap）去执行的。在数据高并发环境的操作中，我们把这样的一个机制称为**乐观锁**。

  ```shell
  # 客户端1
  set lock 1
  # 执行事务前监控lock
  watch lock
  
  multi
  set a 1
  
  
  # 客户端2
  # 修改监控lock
  set lock 0
  
  
  # 客户端1
  # 回滚
  exec
  ```



## 管道技术

Redis 管道技术可以在服务端未响应时，客户端可以继续向服务端发送请求，并最终一次性读取所有服务端的响应。合并操作批量处理，且不阻塞前序命令。

```shell
(echo -en "PING\r\nSET pkey redis\r\nGET pkey\r\nINCR visitor\r\nINCR visitor\r\nINCR visitor\r\n"; sleep 1) | nc localhost 6379
```



## 使用场景

### 业务数据缓存（*）

1. 通用数据缓存，string，int，list，map等。 
2. 实时热数据，最新500条数据。 
3. 会话缓存，token缓存等。



### 业务数据处理

1. 非严格一致性要求的数据：评论，点击等。 
2. 业务数据去重：订单处理的幂等校验等。 
3. 业务数据排序：排名，排行榜等。



### 全局一致计数（*）

1. 全局流控计数 
2. 秒杀的库存计算 
3. 抢红包 
4. 全局ID生成



### 高效统计计数

1. id去重，记录访问ip等全局bitmap操作 
2. UV、PV等访问量（非严格一致性要求）



### 发布订阅与Stream

1. Pub-Sub 模拟队列
   - SUBSCRIBE / PUBLISH
2. Redis Stream 是 Redis 5.0 版本新增加的数据结构。Redis Stream 主要用于消息队列（MQ，Message Queue）。 



### 分布式锁（*）

1. 获取锁：单个原子性操作 

   ```lua
   -- SET key value [EX seconds] [PX milliseconds] [NX|XX]
   -- ex 缓存过期时间 秒
   -- px 缓存过期时间 毫秒
   -- nx 只在键不存在时，才对键进行设置操作
   -- xx 只在键已经存在时，才对键进行设置操作
   SET dlock my_random_value NX PX 30000 
   ```

   

2. 释放锁：lua脚本，其保证原子性，因为内存处理是单线程，从而具有事务性 

   ```lua
   if redis.call("get",KEYS[1]) == ARGV[1] then 
   	return redis.call("del",KEYS[1]) 
   else
   	return 0 
   end 
   ```

   

   - Redis 脚本使用 Lua 解释器来执行脚本。 Redis 2.6 版本内嵌支持 Lua 环境。执行脚本的常用命令为 **EVAL**。

   - `EVAL script numkeys key [key ...] arg [arg ...] `

     - numkeys 键的个数
     - key [key ...] 可以在script中引用，起始索引为1，key的个数要和numkeys保持一致
     - arg [arg ...] 非key参数

     ```lua
     -- 直接执行
     eval "return 'hello java'" 0
     
     -- 传递参数
     eval "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
     
     -- 调用redis命令
     eval "return redis.call('set',KEYS[1],ARGV[1])" 1 foo bar
     
     -- 在服务器上缓存脚本，不会立即执行
     SCRIPT load "return redis.call('set',KEYS[1],ARGV[1])"
     -- 执行缓存脚本
     EVALSHA c686f316aaf1eb01d5a4de1b0b63cd233010e63d 1 t 1
     ```



# 从单机到集群

- 主从复制

  为了分担读压力，Redis支持主从复制，读写分离。一个Master可以有多个Slaves。

  优点

  - 数据备份
  - 读写分离，提高服务器性能

  缺点

  - 不能自动故障恢复
  - 无法实现动态扩容

- 哨兵机制

  Redis Sentinel是社区版本推出的原生高可用解决方案，其部署架构主要包括两部分：Redis Sentinel集群和Redis数据集群。其中Redis Sentinel集群是由若干Sentinel节点组成的分布式集群，可以实现故障发现、故障自动转移、配置中心和客户端通知。Redis Sentinel的节点数量要满足2n+1（n>=1）奇数个。

  优点

  - 自动化故障恢复

  缺点

  - Redis 数据节点中 slave 节点作为备份节点不提供“写”服务
  - 无法实现动态扩容

- redis-Cluster

  Redis Cluster是社区版推出的Redis分布式集群解决方案，主要解决Redis分布式方面的需求，比如，当遇到单机内存，并发和流量等瓶颈的时候，Redis Cluster能起到很好的负载均衡的目的。Redis Cluster着眼于**提高并发量**。

  群集至少需要3主3从，且每个实例使用不同的配置文件。在redis-cluster架构中，redis-master节点一般用于接收读写，而redis-slave节点则一般只用于备份， 其与对应的master拥有相同的slot集合，若某个redis-master意外失效，则再将其对应的slave进行升级为临时redis-master。

  在redis的官方文档中，对redis-cluster架构上，有这样的说明：在cluster架构下，默认的，一般redis-master用于接收读写，而redis-slave则用于备份，当有请求是在向slave发起时，会直接重定向到对应key所在的master来处理。 但如果不介意读取的是redis-cluster中有可能过期的数据并且对写请求不感兴趣时，则亦可通过readonly命令，将slave设置成可读，然后通过slave获取相关的key，达到读写分离。

  优点

  - 解决分布式负载均衡的问题。具体解决方案是分片/虚拟槽slot。
  - 可实现动态扩容
  - P2P模式，无中心化

  缺点

  - 为了性能提升，客户端需要缓存路由表信息
  - Slave在集群中充当“冷备”，不能缓解读压力



## 主从复制

### 配置网络

- 使用 `tunnelblick` + `mac-network` 方案使得宿主机可以直接访问docker容器，docker容器通过bridge组网。因为宿主机可以独立访问容器的IP和端口，所以使用此种方式，可以不需要向宿主机映射端口

```shell
# 创建专用网络
docker network create --driver=bridge --subnet=172.82.0.0/24 redisnet
```



### 配置redis.conf

```properties
# redis01 主
port 6379
logfile "/log/redis.log"

# redis02 从
port 6379
replicaof 172.82.0.100 6379
logfile "/log/redis.log"

# redis03 从
port 6379
replicaof 172.82.0.100 6379
logfile "/log/redis.log"
```



### 启动redis服务

```shell
# 主
docker run -p 6379:6379 --name redis01 --hostname redis01 --net=redisnet --ip=172.82.0.100 \
-v /Users/yangxiaoyu/work/test/redisdatas/redis01/redis.conf:/etc/redis/redis.conf \
-v /Users/yangxiaoyu/work/test/redisdatas/redis01/data:/data \
-v /Users/yangxiaoyu/work/test/redisdatas/redis01/log:/log \
-v /Users/yangxiaoyu/work/test/redisdatas/exchange:/exchange \
-d redis redis-server /etc/redis/redis.conf

# 从
docker run -p 6380:6379 --name redis02 --hostname redis02 --net=redisnet --ip=172.82.0.101 \
-v /Users/yangxiaoyu/work/test/redisdatas/redis02/redis.conf:/etc/redis/redis.conf \
-v /Users/yangxiaoyu/work/test/redisdatas/redis02/data:/data \
-v /Users/yangxiaoyu/work/test/redisdatas/redis02/log:/log \
-v /Users/yangxiaoyu/work/test/redisdatas/exchange:/exchange \
-d redis redis-server /etc/redis/redis.conf

# 从
docker run -p 6381:6379 --name redis03 --hostname redis03 --net=redisnet --ip=172.82.0.102 \
-v /Users/yangxiaoyu/work/test/redisdatas/redis03/redis.conf:/etc/redis/redis.conf \
-v /Users/yangxiaoyu/work/test/redisdatas/redis03/data:/data \
-v /Users/yangxiaoyu/work/test/redisdatas/redis03/log:/log \
-v /Users/yangxiaoyu/work/test/redisdatas/exchange:/exchange \
-d redis redis-server /etc/redis/redis.conf
```



### 验证

- 参数

  ```shell
  # redis01
  info replication
  # role:master
  # connected_slaves:2
  # slave0:ip=172.82.0.101,port=6379,state=online,offset=809,lag=1
  # slave1:ip=172.82.0.102,port=6379,state=online,offset=809,lag=1
  
  # redis02
  info replication
  # role:slave
  # master_host:172.82.0.100
  # master_port:6379
  
  # redis03
  info replication
  # role:slave
  # master_host:172.82.0.100
  # master_port:6379
  ```

- 日志

  ```shell
  1:M 18 Feb 2021 09:12:41.091 * Synchronization with replica 172.82.0.101:6379 succeeded
  1:M 18 Feb 2021 09:21:18.626 * Synchronization with replica 172.82.0.102:6379 succeeded
  ```

- 命令

  - redis01（主）对key 增 / 删 / 改 会同步到redis02（从）和redis03（从）上。redis01可以查询数据
  - redis02（从）和redis03（从）只读



## 高可用

以主从复制的三个主从节点为基础，搭建sentinel。

### 配置sentinel.conf

```shell
# sentinel01
port 26379
logfile "/log/sentinel.log"
# 配置主，不需要配置从，可以通过主获取到需要监控的从
# 最后一个2表示两台sentinel判定主被动下线后，就进行failover(故障转移)
sentinel monitor mymaster 172.82.0.100 6379 2
# 3s内mymaster无响应，则认为mymaster宕机了
sentinel down-after-milliseconds mymaster 3000
# 如果10秒后mysater仍没启动过来，则启动failover  
sentinel failover-timeout mymaster 10000
# 限制同时同新主同步的从数量，即执行故障转移时最多有1个从同新的主进行同步，此时这个从不可用；之后其他从轮询同主进行同步；因此值越小，同步的时间就越久，但值越大，也会造成一段时间内多个从不可用的问题
sentinel parallel-syncs mymaster 1

# sentinel02
port 26379
logfile "/log/sentinel.log"
sentinel monitor mymaster 172.82.0.100 6379 2
sentinel down-after-milliseconds mymaster 3000
sentinel failover-timeout mymaster 10000
sentinel parallel-syncs mymaster 1

# sentinel03
port 26379
logfile "/log/sentinel.log"
sentinel monitor mymaster 172.82.0.100 6379 2
sentinel down-after-milliseconds mymaster 3000
sentinel failover-timeout mymaster 10000
sentinel parallel-syncs mymaster 1
```



### 启动sentinel服务

```shell
# sentinel01
docker run -p 26379:26379 --name sentinel01 --hostname sentinel01 --net=redisnet --ip=172.82.0.200 \
-v /Users/yangxiaoyu/work/test/redisdatas/sentinel01/sentinel.conf:/etc/redis/sentinel.conf \
-v /Users/yangxiaoyu/work/test/redisdatas/sentinel01/data:/data \
-v /Users/yangxiaoyu/work/test/redisdatas/sentinel01/log:/log \
-v /Users/yangxiaoyu/work/test/redisdatas/exchange:/exchange \
-d redis redis-sentinel /etc/redis/sentinel.conf

# sentinel02
docker run -p 26380:26379 --name sentinel02 --hostname sentinel02 --net=redisnet --ip=172.82.0.201 \
-v /Users/yangxiaoyu/work/test/redisdatas/sentinel02/sentinel.conf:/etc/redis/sentinel.conf \
-v /Users/yangxiaoyu/work/test/redisdatas/sentinel02/data:/data \
-v /Users/yangxiaoyu/work/test/redisdatas/sentinel02/log:/log \
-v /Users/yangxiaoyu/work/test/redisdatas/exchange:/exchange \
-d redis redis-sentinel /etc/redis/sentinel.conf

# sentinel03
docker run -p 26381:26379 --name sentinel03 --hostname sentinel03 --net=redisnet --ip=172.82.0.202 \
-v /Users/yangxiaoyu/work/test/redisdatas/sentinel03/sentinel.conf:/etc/redis/sentinel.conf \
-v /Users/yangxiaoyu/work/test/redisdatas/sentinel03/data:/data \
-v /Users/yangxiaoyu/work/test/redisdatas/sentinel03/log:/log \
-v /Users/yangxiaoyu/work/test/redisdatas/exchange:/exchange \
-d redis redis-sentinel /etc/redis/sentinel.conf
```



### 验证

- 日志

  ```shell
  1:X 18 Feb 2021 12:56:14.055 # +monitor master mymaster 172.82.0.100 6379 quorum 2
  # 两从
  1:X 18 Feb 2021 12:56:14.060 * +slave slave 172.82.0.101:6379 172.82.0.101 6379 @ mymaster 172.82.0.100 6379
  1:X 18 Feb 2021 12:56:14.072 * +slave slave 172.82.0.102:6379 172.82.0.102 6379 @ mymaster 172.82.0.100 6379
  # 另外两个sentinel
  1:X 18 Feb 2021 12:56:23.955 * +sentinel sentinel cf2c4ed31c1d46f68bc2fd73ece3b7af96043bd0 172.82.0.201 26379 @ mymaster 172.82.0.100 6379
  1:X 18 Feb 2021 12:56:30.776 * +sentinel sentinel 6f2cd400137500c1a248e6a7074e568c082f02a9 172.82.0.202 26379 @ mymaster 172.82.0.100 6379
  ```

- 命令

  - 主redis01下线
    - redis03被选举为主，redis02为从，符合主从复制约定限制
    - 主redis01上线，成为从，同步主数据
  - 从redis02下线
    - 从redis02上线，仍为从，同步主数据

- 参数

  ```shell
  # 连接到sentinel服务
  redis-cli -h localhost -p 26379
  
  # master清单
  sentinel masters
  
  # 指定master的slave清单
  sentinel slaves mymaster
  
  # 指定master的sentinel清单
  sentinel sentinels mymaster
  ```

  

## 集群

**数据分布**

- 全量数据，单机Redis节点无法满足要求，按照分区规则把数据分到若干个子集当中

- 常用数据分布方式

  - 顺序分布

  - 哈希分布

    - 节点取余分区

      举例：如有100个数据，对每个数据进行hash运算之后，与节点数进行取余运算，根据余数不同保存在不同的节点上

      优点：客户端分片配置简单，对数据进行哈希，然后取余

      缺点：数据节点伸缩时，导致数据迁移。迁移数量和添加节点数据有关，建议翻倍扩容

    - 一致性哈希分区

      原理：将所有的数据当做一个token环，token环中的数据范围是0到2的32次方。然后为每一个数据节点分配一个token范围值，这个节点就负责保存这个范围内的数据。对每一个key进行hash运算，被哈希后的结果在哪个token的范围内，则按顺时针去找最近的节点，这个key将会被保存在这个节点上。

      优点：采用客户端分片方式：哈希 + 顺时针（优化取余）节点伸缩时，只影响邻近节点，但是还是有数据迁移

      缺点：翻倍伸缩，保证最小迁移数据和负载均衡

    - 虚拟槽分区

      <font color=red>虚拟槽分区是Redis Cluster采用的分区方式</font>。

      预设虚拟槽，每个槽就相当于一个数字，有一定范围。每个槽映射一个数据子集，一般比节点数大。Redis Cluster中预设虚拟槽的范围为0到16383。

      - 把16384槽按照节点数量进行平均分配，由节点进行管理

      - 对每个key按照CRC16规则进行hash运算

      - 把hash结果对16384进行取余

      - 把余数发送给Redis节点

      - 节点接收到数据，验证是否在自己管理的槽编号的范围

        - 如果在自己管理的槽编号范围内，则把数据保存到数据槽中，然后返回执行结果

        - 如果在自己管理的槽编号范围外，则会把数据发送给正确的节点，由正确的节点来把数据保存在对应的槽中（Redis Cluster的节点之间会共享消息，每个节点都会知道是哪个节点负责哪个范围内的数据槽）

      优点：虚拟槽分布方式中，由于每个节点管理一部分数据槽，数据保存到数据槽中。当节点扩容或者缩容时，对数据槽进行重新分配迁移即可，数据不会丢失



**redis-cluster**

采用无中心结构，每个节点保存数据和整个集群状态，每个节点都和其他所有节点连接。

- 主从复制不能实现高可用
- 随着公司发展，用户数量增多，并发越来越多，业务需要更高的QPS，而主从复制中单机的QPS可能无法满足业务需求
- 数据量的考虑，现有服务器内存不能满足业务数据的需要时，单纯向服务器添加内存不能达到要求，此时需要考虑分布式需求，把数据分布到不同服务器上
- 网络流量需求，业务的流量已经超过服务器的网卡的上限值，可以考虑使用分布式来进行分流
- 离线计算，需要中间环节缓冲等别的需求

基本架构

- Redis Cluster中有多个节点，每个节点都负责进行数据读写操作
- 节点之间会相互通信，meet操作是节点之间完成相互通信的基础，meet操作有一定的频率和规则
- 把16384个槽平均分配给节点进行管理，<font color=red>每个节点只能对自己负责的槽进行读写操作</font>。由于每个节点之间都彼此通信，每个节点都知道另外节点负责管理的槽范围
- 客户端访问任意节点时，对数据key按照CRC16规则进行hash运算，然后对运算结果对16384进行取余，如果余数在当前访问的节点管理的槽范围内，则直接返回对应的数据。如果不在当前节点负责管理的槽范围内，则会告诉客户端去哪个节点获取数据，由客户端去正确的节点获取数据（多一跳）
- 保证高可用，每个主节点都有一个从节点，当主节点故障，Cluster会按照规则实现主备的高可用性。对于节点来说，有一个配置项`cluster-enabled`，即是否以集群模式启动



### 配置redis.conf

```shell
# redis7000 | redis7001 | redis7002 | redis7003 | redis7004 | redis7005
port 7000
cluster-enabled yes	# 启用集群支持
cluster-config-file nodes.conf # 由redis集群维护，记录集群节点状态等信息
cluster-node-timeout 5000 # 集群不可用最长时间
appendonly yes
daemonize no
protected-mode no
pidfile  /data/redis.pid
```



### 启动redis服务

```shell
# redis7000
docker run --name redis7000 --net=redisnet --ip=172.82.0.70 -v /Users/yangxiaoyu/work/test/redisdatas/cluster/7000:/data -d redis redis-server /data/redis.conf

# redis7001
docker run --name redis7001 --net=redisnet --ip=172.82.0.71 -v /Users/yangxiaoyu/work/test/redisdatas/cluster/7001:/data -d redis redis-server /data/redis.conf

# redis7002
docker run --name redis7002 --net=redisnet --ip=172.82.0.72 -v /Users/yangxiaoyu/work/test/redisdatas/cluster/7002:/data -d redis redis-server /data/redis.conf

# redis7003
docker run --name redis7003 --net=redisnet --ip=172.82.0.73 -v /Users/yangxiaoyu/work/test/redisdatas/cluster/7003:/data -d redis redis-server /data/redis.conf

# redis7004
docker run --name redis7004 --net=redisnet --ip=172.82.0.74 -v /Users/yangxiaoyu/work/test/redisdatas/cluster/7004:/data -d redis redis-server /data/redis.conf

# redis7005
docker run --name redis7005 --net=redisnet --ip=172.82.0.75 -v /Users/yangxiaoyu/work/test/redisdatas/cluster/7005:/data -d redis redis-server /data/redis.conf
```



### 创建集群

```shell
# -p 指定连接 Redis 的端口
# --cluster 使用 Redis 集群模式命令
# create 创建 Redis 集群
# --cluster-replicas 指定副本数（slave 数量）
docker exec -it redis7000 \
redis-cli -p 7000 --cluster create \
172.82.0.70:7000 172.82.0.71:7000 172.82.0.72:7000 \
172.82.0.73:7000 172.82.0.74:7000 172.82.0.75:7000 \
--cluster-replicas 1
```

输出关键信息

```shell
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 172.82.0.74:7000 to 172.82.0.70:7000
Adding replica 172.82.0.75:7000 to 172.82.0.71:7000
Adding replica 172.82.0.73:7000 to 172.82.0.72:7000
```



###验证

- 查看集群信息

  登录任意节点

  ```shell
  # -p：指定连接 Redis 的端口；
  # -c：使用集群模式；
  docker exec -it redis7000 redis-cli -p 7000 -c
  ```

  cluster info

  ```shell
  cluster_state:ok
  cluster_slots_assigned:16384
  cluster_slots_ok:16384
  cluster_slots_pfail:0
  cluster_slots_fail:0
  cluster_known_nodes:6
  cluster_size:3
  cluster_current_epoch:6
  cluster_my_epoch:1
  cluster_stats_messages_ping_sent:1014
  cluster_stats_messages_pong_sent:1031
  cluster_stats_messages_sent:2045
  cluster_stats_messages_ping_received:1026
  cluster_stats_messages_pong_received:1014
  cluster_stats_messages_meet_received:5
  cluster_stats_messages_received:2045
  ```

  cluster nodes 查看主从关联情况

  ```shell
  1f1bfcb5d7f8da84b6b2ce418260b19edb4b4731 172.82.0.73:7000@17000 slave a223227673365bab768d52bce81fe57c74a70c78 0 1613826416777 3 connected
  a1a0e1df50d7b217f0a6e81de7d549168cd158d3 172.82.0.71:7000@17000 master - 0 1613826417000 2 connected 5461-10922
  a223227673365bab768d52bce81fe57c74a70c78 172.82.0.72:7000@17000 master - 0 1613826417593 3 connected 10923-16383
  c9cb4de4f0980694190f8096fe12e21503dfa7ea 172.82.0.75:7000@17000 slave a1a0e1df50d7b217f0a6e81de7d549168cd158d3 0 1613826417000 2 connected
  f31b9855b0314eca9c4b6a431b19a1be5f953b74 172.82.0.74:7000@17000 slave bb49e8c8ff1c7c8c5b58f6bfae09d16ad5a249ab 0 1613826417801 1 connected
  bb49e8c8ff1c7c8c5b58f6bfae09d16ad5a249ab 172.82.0.70:7000@17000 myself,master - 0 1613826415000 1 connected 0-5460
  ```

- 命令

  以单机模式运行master客户端

  ```shell
  docker exec -it redis7000 redis-cli -p 7000
  
  # 提示 (error) MOVED 6257 172.82.0.71:7000
  # 需要在redis7001执行该命令才会成功；同样，也只有在redis7001才可以查询
  set msg helloworld
  ```

  以集群模式运行master客户端

  - <font color=red>在集群模式下任意客户端可以写入，也可以读取；redis客户端会自动重定向，不需要切换客户端</font>

  ```shell
  docker exec -it redis7000 redis-cli -p 7000 -c
  
  # 在redis7000执行，请求被重定向到redis7001
  # redis7000可以查询，也可以读取
  # -> Redirected to slot [6257] located at 172.82.0.71:7000
  # OK
  set msg helloworld
  ```

  以单机模式运行slave客户端

  - <font color=red>当设置为 READONLY 模式时，可以只读查询</font>

  ```shell
  # redis7005是redis7001的slave
  docker exec -it redis7005 redis-cli -p 7000
  
  # 无法查询
  get msg
  ```

  以集群模式运行slave客户端

  - <font color=red>注意当重定向后，keys * 只能查询到重定向节点的数据</font>

  ```shell
  docker exec -it redis7005 redis-cli -p 7000 -c
  
  # 请求被重定向
  # -> Redirected to slot [6257] located at 172.82.0.71:7000
  # "helloworld"
  get msg
  ```



- 测试高可用

  ```shell
  # 目前关联情况
  # master -> slave
  172.82.0.70 -> 172.82.0.74
  172.82.0.71 -> 172.82.0.75
  172.82.0.72 -> 172.82.0.73
  
  # 172.82.0.70下线，172.82.0.74升级为master，slots保持不变
  # 172.82.0.74下线，(error) CLUSTERDOWN The cluster is down 导致集群不可用
  
  # 172.82.0.74上线为master，处理数据正常
  # 172.82.0.70上线为slave，成为备份节点
  
  # 目前关联情况
  # master -> slave
  172.82.0.74 -> 172.82.0.70
  172.82.0.71 -> 172.82.0.75
  172.82.0.72 -> 172.82.0.73
  ```

  

