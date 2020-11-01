# IO模型

|          | 阻塞                         | 非阻塞                         |
| -------- | ---------------------------- | ------------------------------ |
| **同步** | 阻塞I/O模型、I/O多路复用模型 | 非阻塞I/O模型、信号驱动I/O模型 |
| **异步** |                              | 异步I/O模型                    |



<font color=red>阻塞程度：阻塞I/O>非阻塞I/O>I/O多路复用模型>信号驱动I/O>异步I/O ，效率由低到高。</font>



## 阻塞IO（blocking I/O）

![io_blocking](NIO模型与Netty.assets/io_blocking.png)



## 非阻塞IO（noblocking I/O）

![io_noblocking](NIO模型与Netty.assets/io_noblocking.png)



## IO多路复用（I/O multiplexing）

![io_multiplexing](NIO模型与Netty.assets/io_multiplexing.png)



## 信号驱动IO（signal blocking I/O）

![io_signalblocking](NIO模型与Netty.assets/io_signalblocking.png)



## 异步IO（asynchronous I/O）

![io_asynchronous](NIO模型与Netty.assets/io_asynchronous.png)

