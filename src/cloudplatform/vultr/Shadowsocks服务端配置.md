# 注册

## 登录

https://www.vultr.com/

## 注册用户



# 新建Server

Products | Deploy new server

## Server Location

Australia | Sydney

## Server Type

64 bit OS | CentOS | 7 x64



# 管理Server

## 登录

- `ping -c 3 IP Address` 查看延时情况
- ssh root@IP Address
- UI界面 `copy password` 复制到控制台登录 （**不需要修改root密码**）



## 安装wget

视情况安装

```bash
yum -y install wget
```



## 初始化目录

```bash
mkdir -p /export/ && cd /export/
```



## 下载脚本

```bash
# 下载
wget https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh

# 修改权限
chmod +x shadowsocks.sh
```



## 运行脚本

```bash
cd /export/
# 当出现 Enjoy it! 表示安装成功
./shadowsocks.sh 2>&1 | tee shadowsocks.log

# ====
# 密码
# 端口号
# 加密（可以使用默认aes-256-gcm）
# ====
```



## 检查服务

```bash
ps -ef | grep ssserver
```



## 停止服务

```bash
ssserver -c /etc/shadowsocks.json -d stop
```



## 启动服务

```bash
ssserver -c /etc/shadowsocks.json -d start
```



