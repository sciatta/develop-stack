# JDK内置命令行工具

基于centos7

```shell
# 下载镜像
docker pull centos:7

# 启动镜像
# 添加 --cap-add=SYS_PTRACE =解决=》 Can't attach to the process: ptrace(PTRACE_ATTACH, ..) failed for 253: Operation not permitted
docker run --cap-add=SYS_PTRACE --name develop -it \
-v /Users/yangxiaoyu/work/test/javadatas/exchange:/exchange \
-d centos:7 /bin/bash

# 进入环境
docker exec -it develop /bin/bash

# 运行测试程序
java -cp dev-java-jvm-1.0-SNAPSHOT.jar com.sciatta.dev.java.jvm.gc.GCLogAnalysis 3600 1000 1000
```



## jps

Jvm process status tool

Lists the instrumented Java Virtual Machines (JVMs) on the target system. This command is experimental and unsupported.

- jps -mlv

```shell
# -m Displays the arguments passed to the main method. The output may be null for embedded JVMs.
# -l Displays the full package name for the application's main class or the full path name to the application's JAR file.
# -v Displays the arguments passed to the JVM.
jps -mlv
```



## jinfo

Configuration info for java

Generates configuration information. This command is experimental and unsupported.

- `jinfo -flag [+|-]name vmid` 开启或者关闭对应名称的参数。不重启虚拟机，动态调整jvm参数。`jinfo -flag +PrintGC pid`

```shell
jinfo <pid>
```



## jstat*

Jvm statistics monitoring tool

Monitors Java Virtual Machine (JVM) statistics. This command is experimental and unsupported.

`-gc` 需要重点关注OU（老年代的使用量）、YGCT（年轻代GC消耗的总时间）、FGCT（Full GC 消耗的时间）

- jstat -gc
- jstat -gcutil

```shell
# -gcutil Displays a summary about garbage collection statistics. 已使用空间占比
# 1000 间隔时间 默认毫秒
# 3 显示次数
# -t 第一列显示自启动以来的时间戳，单位秒
jstat -gcutil -t <pid> 1000 3

# S0: Survivor space 0 utilization as a percentage of the space's current capacity.
# S1: Survivor space 1 utilization as a percentage of the space's current capacity.
# E: Eden space utilization as a percentage of the space's current capacity.
# O: Old space utilization as a percentage of the space's current capacity.
# M: Metaspace utilization as a percentage of the space's current capacity.
# CCS: Compressed class space utilization as a percentage.
# YGC: Number of young generation GC events.
# YGCT: Young generation garbage collection time.
# FGC: Number of full GC events.
# FGCT: Full garbage collection time.
# GCT: Total garbage collection time.
  Timestamp         S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
         2538.2   1.75   0.00  35.64  60.57  98.26  97.02     38    0.222     2    0.199    0.422
         2539.3   1.75   0.00  35.64  60.57  98.26  97.02     38    0.222     2    0.199    0.422
         2540.3   1.75   0.00  35.78  60.57  98.26  97.02     38    0.222     2    0.199    0.422
         

# -gc Garbage-collected heap statistics. 监视java堆
# jstat -gc -t 7534  1000 3
jstat -gc -t <pid> 1000 3

# S0C: Current survivor space 0 capacity (kB).
# S1C: Current survivor space 1 capacity (kB).
# S0U: Survivor space 0 utilization (kB).
# S1U: Survivor space 1 utilization (kB).
# EC: Current eden space capacity (kB).
# EU: Eden space utilization (kB).
# OC: Current old space capacity (kB).
# OU: Old space utilization (kB).
# MC: Metaspace capacity (kB).
# MU: Metacspace utilization (kB).
# CCSC: Compressed class space capacity (kB).
# CCSU: Compressed class space used (kB).
# YGC: Number of young generation garbage collection events.
# YGCT: Young generation garbage collection time.
# FGC: Number of full GC events.
# FGCT: Full garbage collection time.
# GCT: Total garbage collection time.
Timestamp        S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
         2894.6 2112.0 2112.0  36.9   0.0   17472.0  11941.0   43300.0    26227.7   37632.0 36976.8 4352.0 4222.2     38    0.222   2      0.199    0.422
         2895.6 2112.0 2112.0  36.9   0.0   17472.0  11941.0   43300.0    26227.7   37632.0 36976.8 4352.0 4222.2     38    0.222   2      0.199    0.422
         2896.7 2112.0 2112.0  36.9   0.0   17472.0  11941.0   43300.0    26227.7   37632.0 36976.8 4352.0 4222.2     38    0.222   2      0.199    0.422
```



## jmap

Memory map for java

Prints shared object memory maps or heap memory details for a process, core file, or remote debug server. This command is experimental and unsupported.

- jmap -heap
- jmap -histo
- jmap -dump

```shell
# -heap 打印堆内存的配置和使用信息
jmap -heap 7534

Heap Configuration:
   MinHeapFreeRatio         = 40	# 空余堆小于40%，增加到Xmx
   MaxHeapFreeRatio         = 70	# 空余堆大于70%，减小到Xms
   MaxHeapSize              = 1048576000 (1000.0MB)
   NewSize                  = 10485760 (10.0MB)
   MaxNewSize               = 349503488 (333.3125MB)
   OldSize                  = 20971520 (20.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)
   
Heap Usage:
New Generation (Eden + 1 Survivor Space):
   capacity = 20054016 (19.125MB)
   used     = 363760 (0.3469085693359375MB)
   free     = 19690256 (18.778091430664062MB)
   1.8139010161356208% used
Eden Space:
   capacity = 17891328 (17.0625MB)
   used     = 342360 (0.32649993896484375MB)
   free     = 17548968 (16.736000061035156MB)
   1.9135527558379122% used
From Space:
   capacity = 2162688 (2.0625MB)
   used     = 21400 (0.02040863037109375MB)
   free     = 2141288 (2.0420913696289062MB)
   0.9895093513257576% used
To Space:
   capacity = 2162688 (2.0625MB)
   used     = 0 (0.0MB)
   free     = 2162688 (2.0625MB)
   0.0% used
tenured generation:
   capacity = 44339200 (42.28515625MB)
   used     = 30048248 (28.65624237060547MB)
   free     = 14290952 (13.628913879394531MB)
   67.76903507505774% used

16028 interned Strings occupying 1479120 bytes.

# 查看堆中类的实例占用空间的histogram
jmap -histo 7534 > jmap_hdfs

# dump堆内存
jmap -dump:format=b,file=7534.hprof 7534
```



## jstack

Prints Java thread stack traces for a Java process, core file, or remote debug server. This command is experimental and unsupported.

- jstack -l

```shell
# 长列表模式，将线程相关的locks信息一起输出，比如持有的锁，等待的锁
jstack -l 7534
```



## jcmd*

Sends diagnostic command requests to a running Java Virtual Machine (JVM).

```shell
jcmd 7534 help
```



## jrunscript/jjs

Runs a command-line script shell that supports interactive and batch modes. This command is experimental and unsupported.

```shell
# -e 计算指定的脚步
jrunscript -e "cat('https://www.baidu.com')"

# 启动nashorn引擎交互
jrunscript
```



## jhat*

Java Heap Analysis Tool 内存Dump分析工具

Field Type对应表

| BaseType Character | Type       | 解释     |
| ------------------ | ---------- | -------- |
| B                  | byte       |          |
| C                  | char       |          |
| D                  | double     |          |
| F                  | float      |          |
| I                  | int        |          |
| J                  | long       |          |
| L                  | ClassName; | 类的实例 |
| S                  | short      |          |
| Z                  | boolean    |          |
| [                  | reference  | 一维数组 |
| [[                 | reference  | 二维数组 |



# JDk内置图形化工具

## jconsole

Starts a graphical console that lets you monitor and manage Java applications.



## jvisualvm

Visually monitors, troubleshoots, and profiles Java applications.



## jmc

Java Mission Control is a Profiling, Monitoring, and Diagnostics Tools Suite.



# 第三方工具

## arthas

### 启动

```shell
java -jar arthas-boot.jar

# [1]: 1196 com.sciatta.dev.java.jvm.gc.leak.FullGCLeak
# 选择要分析的程序：1
```



### 命令

- help 

  Display Arthas Help

  ```shell
  # 查看具体命令的帮助信息
  help sc
  ```

  

- sc

  Search all the classes loaded by JVM 

  - **查找具有某一特征的完全限定类名**
  - **可以查找动态生成的java类，然后反编译查看源码**

  ```shell
  # 查找指定类
  # -d 显示类的详细信息
  sc -d *.FullGCLeak
  ```

  

- jad

  Decompile class

  - **反编译**

  ```shell
  jad com.sciatta.dev.java.jvm.gc.leak.FullGCLeak
  
  # 输出反编译的源码到本地目录
  jad --source-only com.sciatta.dev.java.jvm.gc.leak.FullGCLeak > /exchange/test/FullGCLeak.java
  ```



- thread

  Display thread info, thread stack

  - **分析死锁**

  ```shell
  # 显示死锁线程
  thread -b
  ```

  

- jvm

  Display the target JVM information

  ```shell
  jvm
  ```



- mc

  Memory compiler, compiles java files into bytecode and class files in memory.

  - **编译java文件**

  ```shell
  # -c The hash code of the special ClassLoader
  # -d Sets the destination directory for class files
  mc -c 7852e922 /exchange/test/FullGCLeak.java -d /exchange/test
  ```

  

- redefine 

  Redefine classes.

  - **热加载**

  ```shell
  # 反编译class将源码输出到指定文件
  jad --source-only com.sciatta.dev.java.jvm.gc.leak.FullGCLeak > /exchange/test/FullGCLeak.java
  
  # 编辑修改
  vi FullGCLeak.java
  
  # 获取指定类的类加载器的hashcode
  # classLoaderHash   7852e922
  sc -d *.FullGCLeak | grep classLoaderHash
  
  # 内存编译java->class
  mc -c 7852e922 /exchange/test/FullGCLeak.java -d /exchange/test
  
  # 热加载
  redefine /exchange/test/com/sciatta/dev/java/jvm/gc/leak/FullGCLeak$CardInfo.class
  ```

  

### 场景

- 阅读动态代理生成类源码，如dubbo
- 线上不停机更新系统补丁
- 分析死锁
- 系统监控诊断

