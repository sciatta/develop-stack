# Mac安装VMware

## 安装VMware_Fusion_Pro_11.5

注册码 `7HYY8-Z8WWY-F1MAN-ECKNY-LUXYX` 



## 设置网络

偏好设置 | 网络 | 新建vmnet2自定义网络连接

- 选择：使用NAT

- 选择：将MAC主机连接到该网络

- 取消选择：通过DHCP在该网络上提供地址



修改子网

```bash
# 空格需用\转义
cd /Library/Preferences/VMware\ Fusion
sudo vi networking

answer VNET_2_HOSTONLY_SUBNET 192.168.2.0
```



修改网关

```bash
cd vmnet2
sudo vi nat.conf

# NAT gateway address
ip = 192.168.2.2
```



偏好设置 | 网络 | vmnet2

- 取消选择：将MAC主机连接到该网络 | 应用

- 选择：将MAC主机连接到该网络 | 应用

目的是为了使配置生效。



## 创建自定义虚拟机

选择操作系统

- Linux | Centos 7 64



创建完成后设置

- 内存：2048 MB
- 硬盘：40 GB
- 网络适配器：vmnet2
- <font color=red>启动磁盘：CD/DVD（即设置BIOS）</font>



## 安装Centos7

设置中选择 CD/DVD（IDE）

- <font color=red>选中连接CD/DVD驱动器</font>

- 选择镜像CentOS-7-x86_64-DVD-1810.iso



启动

- install Centos 7

- language：english

- date & time：Asia/shanghai

- installation destination：automatic partitioning selected

- network & hostname：ens33 | <font color=red>on</font>

- root password：root（太短，双击确认即可）

