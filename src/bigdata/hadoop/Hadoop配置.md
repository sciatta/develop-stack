# 端口映射

## core-site.xml

### fs.defaultFS

`8020`

NameNode的RPC地址  

## hdfs-site.xml

###dfs.namenode.http-address

`50070`

NameNode的http地址

### dfs.namenode.secondary.http-address  

`50090`

SecondaryNameNode的http地址  



# 数据目录

## hdfs-site.xml

### dfs.namenode.name.dir

`file:///bigdata/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/namenodeDatas/`

- NameNode fsimage

- 可以配置多个目录，不同的目录存放相同的fsimage

### dfs.namenode.edits.dir

`file:///bigdata/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/dfs/nn/edits`

NameNode edits

### dfs.namenode.checkpoint.dir

`file:///bigdata/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/dfs/snn/name`

SecondNameNode fsimage

### dfs.namenode.checkpoint.edits.dir

`file:///bigdata/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/dfs/nn/snn/edits`

SecondNameNode edits

### dfs.datanode.data.dir

`file:///bigdata/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/datanodeDatas`

- DataNode
- 可以配置多个目录，不同的目录存放不同的数据文件