# VMware

## 安装MySQL

- CentOS 7中切换到**root**用户，安装mysql
- CentOS 7中默认安装有MariaDB，这个是MySQL的分支；但还是要安装MySQL，安装完成之后会直接覆盖掉MariaDB

安装在node03上

```bash
# 切换到root用户
su root

# 安装wget
cd /bigdata/soft/
yum -y install wget

# 使用wget命令下载mysql的rpm包
# -i 指定输入文件
# -c 表示断点续传
wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm

# 安装mysql
yum -y install mysql57-community-release-el7-10.noarch.rpm

# 安装mysql server
yum -y install mysql-community-server
```



## 设置MySQL

启动服务

```bash
# 启动MySQL服务
systemctl start mysqld.service

# 查看mysql启动状态
# active（running）表示mysql服务已启动
systemctl status mysqld.service
```



用临时密码登录

```bash
# 找出临时密码
# G;sZ/.i(G7Gt
grep "password" /var/log/mysqld.log

# 使用临时密码，登陆mysql客户端
mysql -uroot -p
```



修改密码

```mysql
-- 设置密码策略为LOW，此策略只检查密码的长度
set global validate_password_policy=LOW;

-- 设置密码最小长度
set global validate_password_length=4;

-- 修改mysql的root用户，本地登陆的密码为root
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';

-- 开启mysql的远程连接权限
grant all privileges  on  *.* to 'root'@'%' identified by 'root' with grant option;

-- 即时生效
flush privileges;

-- 退出
exit
```



## 卸载MySQL

使用root用户卸载mysql

```bash
# 停止mysql服务
systemctl stop mysqld.service

# 列出已安装的mysql相关的包
# 卸载完成后，用这两个命令再次检查
yum list installed mysql*
# 或
rpm -qa | grep -i mysql

# 卸载，命令后边依次添加上一步列出的包名，包名之间用空格分隔
rpm -e --nodeps
```



删除mysql残留文件

```bash
# 查看mysql相关目录
find / -name mysql

# 删除上一步列出的目录
rm -rf

# 删除文件
rm -rf /root/.mysql_history
rm -f /var/log/mysqld.log
```



# Docker

## 下载镜像

```shell
docker pull mysql:5.7.32
```



## 后台启动

```shell
docker run -itd -v /Users/yangxiaoyu/work/test/mysqldatas/exchange:/exchange -v /Users/yangxiaoyu/work/test/mysqldatas/mysql-test:/var/lib/mysql --name mysql-test -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root mysql:5.7.32

# 查看容器运行情况
docker container ls -a
```



## 进入容器

```shell
# 进入正在运行的容器
docker container exec -it mysql-test /bin/bash

# 登录mysql客户端
mysql -uroot -p
# 退出mysql客户端
quit
```



## 退出容器

```shell
# 退出容器，容器仍处于运行中
control+q+p
# 或
exit
```



## 停止容器

```shell
docker stop mysql-test
```



## 启动容器

```shell
docker start mysql-test
```

