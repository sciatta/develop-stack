# 安装

node03执行

```bash
scp flume-ng-1.6.0-cdh5.14.2.tar.gz hadoop@node03:/bigdata/soft
tar -zxvf flume-ng-1.6.0-cdh5.14.2.tar.gz -C /bigdata/install/
```



# 配置

## flume-env.sh

```bash
cd /bigdata/install/apache-flume-1.6.0-cdh5.14.2-bin/conf
cp flume-env.sh.template flume-env.sh
vi flume-env.sh

# 增加java环境变量
export JAVA_HOME=/bigdata/install/jdk1.8.0_141
```



## /etc/profile

```bash
sudo vi /etc/profile

# 增加flume环境变量
export FLUME_HOME=/bigdata/install/apache-flume-1.6.0-cdh5.14.2-bin
export PATH=$PATH:$FLUME_HOME/bin

# 立即生效
source /etc/profile
```



