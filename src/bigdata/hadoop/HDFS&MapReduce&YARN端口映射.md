# HDFS

| 参数                                | 位置          | 默认  | 描述                        |
| ----------------------------------- | ------------- | ----- | --------------------------- |
| fs.defaultFS                        | core-site.xml | 8020  | NameNode的RPC地址           |
| dfs.namenode.http-address           | hdfs-site.xml | 50070 | NameNode的http地址          |
| dfs.namenode.secondary.http-address | hdfs-site.xml | 50090 | SecondaryNameNode的http地址 |



# MapReduce

| 参数                                | 位置            | 默认  | 描述                 |
| ----------------------------------- | --------------- | ----- | -------------------- |
| mapreduce.jobhistory.address        | mapred-site.xml | 10020 | jobhistory的RPC地址  |
| mapreduce.jobhistory.webapp.address | mapred-site.xml | 19888 | jobhistory的http地址 |



# YARN

| 参数                                | 位置          | 默认 | 描述                     |
| ----------------------------------- | ------------- | ---- | ------------------------ |
| yarn.resourcemanager.webapp.address | yarn-site.xml | 8088 | yarn的http地址           |
|                                     |               | 8032 | ResourceManager的RPC地址 |

