## Netty

### Netty运行原理



### Netty优化

- 不要阻塞 EventLoop

- 系统参数优化
  `ulimit -a` 增大最大进程数

  `/proc/sys/net/ipv4/tcp_fin_timeout`, `TcpTimedWaitDelay` 缩短TIME_WAIT等待时间

- 缓冲区优化
  `SO_RCVBUF` 接收缓冲区

  `SO_SNDBUF` 发送缓冲区

  `SO_BACKLOG` 保持连接状态

  `REUSEXXX` 重用端口

- 心跳频率周期优化
  心跳机制与断线重连

- 内存 与 ByteBuffer 优化
  DirectBuffer与HeapBuffer 映射堆外内存，零拷贝

- 其他优化

  ioRatio

  Watermark

  TrafficShaping 流控



### 网络程序优化

- 粘包与拆包

  对报文没有指定长度，没有结束符。客户端和服务端要约定报文传递规则。

- 网络拥堵与Nagle算法优化 TCP_NODELAY

  MTU: Maxitum Transmission Unit 最大传输单元 1500Byte

  MSS: Maxitum Segment Size 最大分段大小 1460Byte，其中TCP头20Byte，IP头20Byte

  如果网络上按字节发送，都要带40Byte头，传输不经济划算，其次会带来网络拥堵。

  优化条件

  - 启用Nagle算法（默认），关闭TCP_NODELAY，缓冲区满或达到超时才发送数据包，减少网络传输数据包，适用于并发高、数据量大的场景
  - 禁用Nagle算法，启动TCP_NODELAY。适用于对延迟敏感、数据量小的场景，如SSH会话

- 连接优化

  TCP建立连接（3次握手）

  1. `Client -> Server` SYN（你在吗？）

  2. `Server -> Client` ACK（我在！） + SYN（你在吗？）

     Client收到ACK后，状态变为ESTABLISHED

  3. `Client -> Server` ACK（我在！）

     Server收到ACK后，状态变为ESTABLISHED

  TCP关闭连接（4次挥手）

  1. `Client -> Server` FIN（我要离开！）

  2. `Server -> Client` ACK（第一次确认！）

     Server状态变为CLOSE_WAIT

  3. `Server -> Client` FIN（我要离开！） + ACK（第二次确认！）

     Client状态变为TIME_WAIT，**需要等待2MSL后**，状态变为CLOSED

  4. `Client -> Server` ACK（确认！）

     Server状态变为CLOSED

  优化条件

  1. 缩短2MSL等待周期
  2. 开启端口复用

