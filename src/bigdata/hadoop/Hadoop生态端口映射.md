# HDFS

| 参数                                 | 位置          | 默认  | 描述                         |
| ------------------------------------ | ------------- | ----- | ---------------------------- |
| fs.defaultFS                         | core-site.xml | 8020  | NameNode的RPC地址            |
| dfs.namenode.http-address            | hdfs-site.xml | 50070 | NameNode的http地址           |
| dfs.namenode.secondary.http-address  | hdfs-site.xml | 50090 | SecondaryNameNode的http地址  |
| dfs.namenode.secondary.https-address | hdfs-site.xml | 50091 | SecondaryNameNode的https地址 |
| dfs.datanode.address                 | hdfs-site.xml | 50010 | DataNode的RPC地址            |
| dfs.datanode.http.address            | hdfs-site.xml | 50075 | DataNode的http地址           |



# MapReduce

| 参数                                | 位置            | 默认  | 描述                 |
| ----------------------------------- | --------------- | ----- | -------------------- |
| mapreduce.jobhistory.address        | mapred-site.xml | 10020 | jobhistory的RPC地址  |
| mapreduce.jobhistory.webapp.address | mapred-site.xml | 19888 | jobhistory的http地址 |



# YARN

| 参数                                | 位置          | 默认 | 描述                     |
| ----------------------------------- | ------------- | ---- | ------------------------ |
| yarn.resourcemanager.webapp.address | yarn-site.xml | 8088 | yarn的http地址           |
| yarn.resourcemanager.address        | yarn-site.xml | 8032 | ResourceManager的RPC地址 |



# HBase

| 参数                                | 位置           | 默认  | 描述                                                |
| ----------------------------------- | -------------- | ----- | --------------------------------------------------- |
| hbase.master.port                   | hbase-site.xml | 16000 | HMaster绑定的端口                                   |
| hbase.zookeeper.property.clientPort | hbase-site.xml | 2181  | 客户端连接Zookeeper的端口，来自于Zookeeper的zoo.cfg |

