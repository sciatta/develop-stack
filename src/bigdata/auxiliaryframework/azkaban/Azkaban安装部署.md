# 编译

需要使用jdk1.8

``` shell
sudo yum -y install wget
sudo yum -y install git
sudo yum -y install gcc-c++

cd /bigdata/soft/
wget https://github.com/azkaban/azkaban/archive/3.51.0.tar.gz
mv 3.51.0.tar.gz azkaban-3.51.0.tar.gz
tar -zxvf azkaban-3.51.0.tar.gz -C ../install/

cd /bigdata/install/azkaban-3.51.0
./gradlew build installDist -x test
```

经过漫长的编译后安装文件列表如下

- azkaban-exec-server-0.1.0-SNAPSHOT.tar.gz

  `/bigdata/install/azkaban-3.51.0/azkaban-exec-server/build/distributions`

- azkaban-web-server-0.1.0-SNAPSHOT.tar.gz

  `/bigdata/install/azkaban-3.51.0/azkaban-web-server/build/distributions`

- azkaban-solo-server-0.1.0-SNAPSHOT.tar.gz

  `/bigdata/install/azkaban-3.51.0/azkaban-solo-server/build/distributions`

- execute-as-user.c（two server mode 需要的C程序）

  `/bigdata/install/azkaban-3.51.0/az-exec-util/src/main/c` 

- create-all-sql-0.1.0-SNAPSHOT.sql（数据库脚本）

  /bigdata/install/azkaban-3.51.0/azkaban-db/build/install/azkaban-db



# 服务安装

## 数据库准备

登录mysql客户端 `mysql -uroot -proot`

执行命令

```mysql
CREATE DATABASE azkaban;

-- %	可以在任意远程主机登录
-- SHOW VARIABLES LIKE 'validate_password%'; 							密码有效期
-- SET GLOBAL validate_password_length = 7;								密码长度
-- SET GLOBAL validate_password_number_count = 0;					密码中数字个数
-- SET GLOBAL validate_password_mixed_case_count = 0;			混合大小写个数
-- SET GLOBAL validate_password_special_char_count = 0;		特殊字符个数
CREATE USER 'azkaban'@'%' IDENTIFIED BY 'azkaban';

GRANT all privileges ON azkaban.* to 'azkaban'@'%' identified by 'azkaban' WITH GRANT OPTION;

flush privileges;

use azkaban;

source /bigdata/soft/azkaban/create-all-sql-0.1.0-SNAPSHOT.sql
```



## 解压安装包

```shell
tar -xzvf azkaban-web-server-0.1.0-SNAPSHOT.tar.gz -C ../../install/

tar -xzvf azkaban-exec-server-0.1.0-SNAPSHOT.tar.gz -C ../../install/

mv azkaban-web-server-0.1.0-SNAPSHOT/ azkaban-web-server-3.51.0

mv azkaban-exec-server-0.1.0-SNAPSHOT/ azkaban-exec-server-3.51.0
```



## azkaban-web-server 安装

### 安装SSL安全认证

允许使用https的方式访问azkaban的web服务

```shell
cd /bigdata/install/azkaban-web-server-3.51.0
# 密码 azkaban
keytool -keystore keystore -alias jetty -genkey -keyalg RSA
```

### 修改配置文件

#### azkaban.properties

```shell
vi azkaban.properties

azkaban.name=Azkaban
azkaban.label=My Azkaban
default.timezone.id=Asia/Shanghai

jetty.use.ssl=true

jetty.ssl.port=8443
jetty.keystore=/bigdata/install/azkaban-web-server-3.51.0/keystore
jetty.password=azkaban
jetty.keypassword=azkaban
jetty.truststore=/bigdata/install/azkaban-web-server-3.51.0/keystore
jetty.trustpassword=azkaban

mysql.host=node03

# azkaban.executorselector.filters=StaticRemainingFlowSize,MinimumFreeMemory,CpuStatus

azkaban.activeexecutor.refresh.milisecinterval=10000
azkaban.queueprocessing.enabled=true
azkaban.activeexecutor.refresh.flowinterval=10
azkaban.executorinfo.refresh.maxThreads=10
```



## azkaban-exec-server 安装

### 修改配置文件

#### azkaban.properties

```shell
azkaban.name=Azkaban
azkaban.label=My Azkaban
default.timezone.id=Asia/Shanghai

jetty.use.ssl=true

jetty.keystore=/bigdata/install/azkaban-web-server-3.51.0/keystore
jetty.password=azkaban
jetty.keypassword=azkaban
jetty.truststore=/bigdata/install/azkaban-web-server-3.51.0/keystore
jetty.trustpassword=azkaban

azkaban.webserver.url=https://node03:8443

mysql.host=node03
```

### 插件

#### 添加插件

```shell
sudo yum -y install gcc-c++

cp execute-as-user.c /bigdata/install/azkaban-exec-server-3.51.0/plugins/jobtypes
cd /bigdata/install/azkaban-exec-server-3.51.0/plugins/jobtypes

gcc execute-as-user.c -o execute-as-user
sudo chown root execute-as-user
sudo chmod 6050 execute-as-user
```

#### 修改配置文件 commonprivate.properties

```shell
cd /bigdata/install/azkaban-exec-server-3.51.0/plugins/jobtypes
vi commonprivate.properties

memCheck.enabled=false
azkaban.native.lib=/bigdata/install/azkaban-exec-server-3.51.0/plugins/jobtypes
```



# 启动服务

```shell
# AzkabanExecutorServer
cd /bigdata/install/azkaban-exec-server-3.51.0
bin/start-exec.sh
# 激活
# {"status":"success"}
curl -G "node03:$(<./executor.port)/executor?action=activate" && echo

# AzkabanWebServer
cd /bigdata/install/azkaban-web-server-3.51.0
bin/start-web.sh
```



# 停止服务

```shell
# AzkabanExecutorServer
cd /bigdata/install/azkaban-exec-server-3.51.0
bin/shutdown-exec.sh

# AzkabanWebServer
cd /bigdata/install/azkaban-web-server-3.51.0
bin/shutdown-web.sh
```



# Web访问

```  shell
# AzkabanWebServer地址
# 用户名和密码：azkaban
https://node03:8443
```

