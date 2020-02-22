# HDFS

| 参数                                 | 位置          | 默认  | 描述                         |
| ------------------------------------ | ------------- | ----- | ---------------------------- |
| fs.defaultFS                         | core-site.xml | 8020  | NameNode的RPC端口            |
| dfs.namenode.http-address            | hdfs-site.xml | 50070 | NameNode的WebUI端口          |
| dfs.namenode.secondary.http-address  | hdfs-site.xml | 50090 | SecondaryNameNode的http端口  |
| dfs.namenode.secondary.https-address | hdfs-site.xml | 50091 | SecondaryNameNode的https端口 |
| dfs.datanode.address                 | hdfs-site.xml | 50010 | DataNode的RPC端口            |
| dfs.datanode.http.address            | hdfs-site.xml | 50075 | DataNode的http端口           |



# MapReduce

| 参数                                | 位置            | 默认  | 描述                  |
| ----------------------------------- | --------------- | ----- | --------------------- |
| mapreduce.jobhistory.address        | mapred-site.xml | 10020 | jobhistory的RPC端口   |
| mapreduce.jobhistory.webapp.address | mapred-site.xml | 19888 | jobhistory的WebUI端口 |



# YARN

| 参数                                | 位置          | 默认 | 描述                     |
| ----------------------------------- | ------------- | ---- | ------------------------ |
| yarn.resourcemanager.webapp.address | yarn-site.xml | 8088 | yarn的WebUI端口          |
| yarn.resourcemanager.address        | yarn-site.xml | 8032 | ResourceManager的RPC端口 |



# HBase

| 参数                                | 位置           | 默认  | 描述                                                |
| ----------------------------------- | -------------- | ----- | --------------------------------------------------- |
| hbase.master.port                   | hbase-site.xml | 16000 | HMaster绑定的端口                                   |
| hbase.zookeeper.property.clientPort | hbase-site.xml | 2181  | 客户端连接Zookeeper的端口，来自于Zookeeper的zoo.cfg |
| hbase.regionserver.info.port        | hbase-site.xml | 60030 | HBase的WebUI端口                                    |

