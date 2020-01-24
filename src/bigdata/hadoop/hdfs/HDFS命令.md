# hdfs

## hdfs dfs

hdfs有两种命令风格

- hadoop fs
- hdfs dfs

两种命令等价

### help

```bash
hadoop fs -help ls
hdfs dfs -help ls
```



### ls

```bash
# 查看根目录文件列表
hdfs dfs -ls /

# 递归显示目录内容
hdfs dfs -ls -R /

# 显示本地文件系统列表，默认hdfs
# file:// 表示本地文件协议
hdfs dfs -ls file:///bigdata/
```



### touchz

```bash
# 创建空文件
hdfs dfs -touchz /mynote
```



### appendToFile

```bash
# 向文件末尾追加内容
# 注意命令区分大小写
hdfs dfs -appendToFile hello /mynote
```



### cat

```bash
# 查看文件内容
hdfs dfs -cat /mynote
```



### put

```bash
# 上传本地文件到hdfs
hdfs dfs -put hello /h1
```



### copyFromLocal（同put）

```bash
hdfs dfs -copyFromLocal hello /h2
```



### moveFromLocal

```bash
# 上传成功，删除本机文件
hdfs dfs -moveFromLocal hello /h3
```



### get

```bash
# 下载hdfs文件到本地
hdfs dfs -get /h1 hello
```



### copyToLocal（同get）

```bash
hdfs dfs -copyToLocal /h1 hello1
```



### mkdir

```bash
# 创建目录
hdfs dfs -mkdir /shell
```



### rm

```bash
# 删除文件到垃圾桶
# 不能删除目录
hdfs dfs -rm /h1

# 递归删除目录
hdfs dfs -rm -r /shell
```



### rmr

```bash
# 递归删除目录
# 不建议使用，可使用 rm -r 代替
hdfs dfs -rmr /hello
```



### mv

```bash
# 目的文件或目录不存在，修改文件名或目录名
# 目的文件存在，不可移动
hdfs dfs -mv /h2 /h22

# 目的目录存在，将子文件或子目录移动到目录
hdfs dfs -mv /h22 /hello
```



### cp

```bash
# 拷贝文件
# 若目的文件存在，不可复制
hdfs dfs -cp /bigf /bf

# 拷贝文件到目的目录
hdfs dfs -cp /bigf /hello
```



### find

```bash
# 查找文件
hdfs dfs -find / -name "h*"
```



### text

```bash
# 查看文件内容，若为SequenceFile，即使压缩，也可以正常查看压缩前内容
hdfs dfs -text /sequence/none
```



### expunge

```bash
 # 清空回收站，同时创建回收站checkpoint
 hdfs dfs -expunge
```



## hdfs getconf

### namenodes

```bash
# 获取NameNode节点名称，可能有多个
hdfs getconf -namenodes
```



### confKey

```bash
# 用相同的命令可以获得其他属性值
# 获取最小块大小 默认1048576byte（1M）
hdfs getconf -confKey dfs.namenode.fs-limits.min-block-size
```



### nnRpcAddresses

```bash
# 获取NameNode的RPC地址
hdfs getconf -nnRpcAddresses
```



## hdfs dfsadmin

### safemode

```bash
# 查看当前安全模式状态
hdfs dfsadmin -safemode get

# 进入安全模式
# 安全模式只读
# 增删改不可以，查可以
hdfs dfsadmin -safemode enter

# 退出安全模式
hdfs dfsadmin -safemode leave  
```



### allowSnapshot

快照顾名思义，就是相当于对我们的hdfs文件系统做一个备份，我们可以通过快照对我们指定的文件夹设置备份，但是添加快照之后，并不会立即复制所有文件，而是指向同一个文件。当写入发生时，才会产生新文件。

```bash
# 创建快照之前，先要允许该目录创建快照
hdfs dfsadmin -allowSnapshot /main
# 禁用
hdfs dfsadmin -disallowSnapshot /main

# 指定目录创建快照
# Created snapshot /main/.snapshot/s20200113-114345.126
# 可以通过浏览器访问 http://192.168.2.100:50070/explorer.html#/main/.snapshot/s20200113-114345.126
hdfs dfs -createSnapshot /main

# 创建快照指定名称
hdfs dfs -createSnapshot /main snap1

# 快照重命名
hdfs dfs -renameSnapshot /main snap1 snap2

# 列出当前用户下的所有快照目录
hdfs lsSnapshottableDir

# 比较两个快照的不同
hdfs snapshotDiff /main snap1 snap2

# 删除快照
hdfs dfs -deleteSnapshot /main snap1
```



## hdfs fsck

```bash
# 查看文件的文件、块、位置信息
hdfs fsck /h3 -files -blocks -locations
```



## hdfs namenode

### format

```bash
# 格式化NameNode，只在初次搭建集群时使用
hdfs namenode -format
```



## hdfs oiv

```bash
# 查看fsimage内容 offine image view
# -i 输入文件
# -p 处理格式
# -o 输出文件
hdfs oiv -i fsimage_0000000000000000196 -p XML -o test.xml
```



## hdfs oev

```bash
# 查看edits内容 offine edits view
# -i 输入文件
# -p 处理格式
# -o 输出文件
hdfs oev -i edits_0000000000000000001-0000000000000000009 -p XML -o test.xml
```



# hadoop

## hadoop fs（同hdfs dfs）

## hadoop checknative

```bash
# 查看本地库安装状态
hadoop checknative
```



## hadoop jar

```bash
# 在集群上执行自定义的jar
hadoop jar hadoop-mapreduce-wordcount-1.0-SNAPSHOT.jar com.sciatta.hadoop.mapreduce.wordcount.WordCount
```



## hadoop archive

```bash
# -archiveName 档案名称
# -p 父目录
# <src>* 相对于父目录的相对路径
# <dest> 存储档案名称的路径
hadoop archive -archiveName data.har -p /main data data1 data2 /main
```



## hadoop distcp

```bash
# hadoop 集群间数据拷贝
hadoop distcp hdfs://node01:8020/test hdfs://cluster:8020/
```