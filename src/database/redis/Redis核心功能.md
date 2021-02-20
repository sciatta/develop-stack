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

  

