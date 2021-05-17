# Netty运行原理



# Netty优化

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



# 网络程序优化

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

  - 缩短2MSL等待周期

  - 开启端口复用

