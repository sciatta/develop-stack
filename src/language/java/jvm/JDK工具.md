# JDK内置命令行工具

## jps

Lists the instrumented Java Virtual Machines (JVMs) on the target system. This command is experimental and unsupported.

```shell
# -m Displays the arguments passed to the main method. The output may be null for embedded JVMs.
# -l Displays the full package name for the application's main class or the full path name to the application's JAR file.
# -v Displays the arguments passed to the JVM.
jps -mlv
```



## jinfo

Generates configuration information. This command is experimental and unsupported.

```shell
jinfo <pid>
```



## jstat*

Monitors Java Virtual Machine (JVM) statistics. This command is experimental and unsupported.

`jstat -gc` 需要重点关注OU（老年代的使用量）、YGCT（年轻代GC消耗的总时间）、FGCT（Full GC 消耗的时间）

```shell
# -gcutil Displays a summary about garbage collection statistics.
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
         

# -gc Garbage-collected heap statistics.
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

# 内存参数
# 堆内存 -Xms 30m -Xmx 1000m
# 新生代 -XX:NewSize 10m -XX:MaxNewSize 333m
# 老生代 -XX:OldSize 20m
# -XX:SurvivorRatio=8（默认8)
# -XX:NewRatio=2（默认2）

# -XX:CICompilerCount=2 -XX:InitialHeapSize=31457280 -XX:MaxHeapSize=1048576000 -XX:MaxNewSize=349503488 -XX:MinHeapDeltaBytes=196608 -XX:NewSize=10485760 -XX:OldSize=20971520 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps
```



## jmap

Prints shared object memory maps or heap memory details for a process, core file, or remote debug server. This command is experimental and unsupported.

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



# JDk内置图形化工具

## jconsole

Starts a graphical console that lets you monitor and manage Java applications.

## jvisualvm

Visually monitors, troubleshoots, and profiles Java applications.

## jmc

Java Mission Control is a Profiling, Monitoring, and Diagnostics Tools Suite.

