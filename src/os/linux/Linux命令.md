# 文件管理

## cat

```shell
# 1、将file1内容和file2内容输出到file3
# 2、“命令 > 文件”将标准输出重定向到一个文件中（清空原有文件的数据）
# 3、“命令 >> 文件”将标准输出重定向到一个文件中（追加到原有内容的后面）
cat file1 file2 > file3

# 由1开始对所有输出的行编号，包括空行
cat -n file1 file2 > file3

# 由1开始对所有输出的行编号，不包括空行
cat -b file1 file2 > file3

# 拷贝标准输入到标准输出
cat

# 拷贝标准输入到file
cat > file

# 拷贝file1到标准输出
cat file1

# 拷贝file1和file2到标准输出
cat file1 file2

# 1、清空file1的内容
# 2、dev/null：在类Unix系统中，/dev/null称空设备，它丢弃一切写入其中的数据（但报告写入操作成功），
# 读取它则会立即得到一个EOF
cat /dev/null > file1

# 不会得到任何信息，因为将本来该通过标准输出显示的文件信息重定向到了/dev/null中
cat file1 > /dev/null
```



## chown

```shell
# 修改文件的用户及用户组，需要root权限
chown hadoop:test file

# -R 递归设置当前目录和其下子目录、文件
chown -R hadoop:hadoop dir
```



## chgrp

```shell
# 修改文件或目录所属的用户组
# -v 显示指令执行过程
chgrp -v hadoop file
```



## chmod

- Linux/Unix 的文件调用权限分为三级 : 文件拥有者、群组、其他
- <font color=red>设置权限</font>
  - 字串设定 `[ugoa...][[+-=][rwxX]...][,...]`
    - u 表示该文件的拥有者，g 表示与该文件的拥有者属于同一个群组，o 表示其他以外的人，a 表示这三者皆是
    - +表示增加权限、- 表示取消权限、= 表示唯一设定权限
    - r 表示可读取，w 表示可写入，x 表示可执行
  - 数字设定 `chmod abc file`
    - a，b，c各为一个数字，分别表示User，Group，及Other的权限
    - r=4，w=2，x=1

```shell
# 所有人增加写权限
chmod a+w file

# 文件拥有者和群组增加写权限，其他减少写权限
chmod ug+w,o-w file

# -R 递归设置
chmod -R u+rwx,go-rwx dir

# 所有人增加读写执行权限
chmod 777 file
```



## find

``` shell
# 查找当前目录和子目录下所有匹配样式file*的文件或目录，输出路径
find . -name "file*"

# 查找当前目录名称为.lastUpdated后缀的所有文件或目录，然后删除
# xargs传递上一个指令的输出作为参数，一般和 | 联用，使得一些不支持 | 管道符的命令（rm）也可以接受到上一个命令的输出作为参数
find . -name "*.lastUpdated" | xargs rm -rfv
```



## cut

```shell
# 截取每一行的第二个字节
cut -b 2 file

# 截取每一行的第二个字符，汉子可以显示正确
cut -c 2 file

# 截取每一行的第二~三个字符
cut -c 2-3 file

# 默认分隔符tab，若没有分隔符，则输出整行
# -f 指定输出区域，默认从1开始
cut -f 2 file
# -s 没有分割符不打印
cut -s -f 2 file
# -d 指定分隔符为空格
cut -d ' ' -f 2 file
```



## ln

- 建立链接（link），不必重复的占用磁盘空间
- 软链接
  - 以路径的形式存在。类似于Windows操作系统中的快捷方式
  - 可以跨文件系统，硬链接不可以
  - 可以对一个不存在的文件名进行链接
  - 可以对目录进行链接
- 硬链接
  - 以文件副本的形式存在。但不占用实际空间
  - 不允许给目录创建硬链接
  - 硬链接只有在同一个文件系统中才能创建

```shell
# -s 建立软链接
ln -s file linkf
```



## less

- less 与 more 类似，但使用 less 可以随意浏览文件，而 more 仅能向前移动，却不能向后移动
- less 在查看之前不会加载整个文件

```shell
# ps查看进程信息并通过less分页显示
ps -ef | less

# 查看命令历史使用记录并通过less分页显示
history | less

# 1、浏览多个文件
# 2、输入
# 	:n  切换到file2
# 	:p  切换到file1
less file1 file2

# ******************************************************
# 移动到最后一行
G
# 移动到第一行
g
# 向下搜索“字符串”
/字符串
# 向上搜索“字符串”的功能
?字符串
# 重复前一个搜索（与 / 或 ? 有关）
n
# 反向重复前一个搜索（与 / 或 ? 有关）
N
# 默认向下移动一行，输入数字回车则向下移动j行
（j）Enter
# 向上移动一行
k
# 向下翻一页
空格键（d）
# 向上翻一页
b
# 退出less命令
q（ZZ）	
# ******************************************************
```



## more

```shell
# 如有连续两行以上空白行则以一行空白行显示
more -s file

# 从第5行开始显示
more +5 file

# ******************************************************
# 向下翻一页
空格键	
# 向上翻一页
b	
# ******************************************************
```



## mv

```shell
# 移动并改名
mv file1 file

# ok和no都是目录。当no不存在时，则将ok改名为no；当no存在时，将ok目录转移到no目录
mv ok/ no

# ok和no都是目录。当no不存在时，失败；当no存在时，将ok目录所有内容转移到no目录
mv ok/* no
```



## rm

```shell
# rm: remove regular file ‘services’? n
# 删除文件需要询问
rm services

# -r 递归删除目录
rm -r test

# -f 强制删除所有文件和目录（即使只读），不需要询问
rm -rf test
```



## touch

```shell
# -rw-r--r--. 1 root root 670293 Dec 31 17:10 services
# -rw-r--r--. 1 root root 670293 Dec 31 21:07 services
# 文件存在，更新修改时间为当前时间
touch services

# 文件不存在，创建空文件
touch file1

# 创建一个名称为 --name 的文件(同 -name)
# 错误：touch --name 无法识别的选项；-和--后面接option
# 修正：touch -- --name	-- 后面不会再包含option，被认为是 filenames 和 arguments
touch -- --name
# 错误：cat --name 无法识别的选项
cat -- --name
```



## which

```shell
# which指令会在环境变量$PATH设置的目录里查找符合条件的文件
which bash

# 打印所有匹配的路径（不仅只有第一个）
which -a bash
```



## cp

```shell
# -r 递归复制test目录下的内容到test1
cp -r test test1
```



## scp

- scp 是 secure copy 的缩写，scp 是 linux 系统下基于 ssh 登陆进行安全的远程文件拷贝命令
- scp 是加密的，rcp 是不加密的，scp 是 rcp 的加强版

``` shell
# 拷贝 本地文件 -> 远程目录
scp jdk-8u141-linux-x64.tar.gz hadoop@192.168.2.100:/bigdata/soft 

# 拷贝 本地目录 -> 远程目录
# -r 递归拷贝目录
scp -r hadoop-2.6.0-cdh5.14.2/ node02:$PWD

# 拷贝 远程文件 -> 本地目录
scp hadoop@192.168.2.100:/bigdata/soft/jdk-8u141-linux-x64.tar.gz /bigdata/soft 
```



## awk

- <font color=red>格式化文本</font>
- AWK 取了三位创始人 Alfred Aho，Peter Weinberger 和 Brian Kernighan 的 Family Name 的首字符
- BEGIN{ } 行处理前执行语句
- {} 每行处理执行的语句
- END{} 行处理后执行语句

```shell
# ******************************************************
# 测试数据
log.txt
# ******************************************************
2 this is a test
3 Are you like awk
This's a test
10 There are orange,apple,mongo
# ******************************************************

# 行匹配语句 awk '' 只能用单引号
# 每行按空格或TAB分割，输出文本中的1、4项
awk '{print $1,$4}' log.txt
# 格式化输出
awk '{printf "%-8s %-10s\n",$1,$4}' log.txt

# -F 指定分隔符
awk -F, '{print $1,$2}' log.txt
# 使用多个分隔符。先使用空格分割，然后对分割结果再使用,分割
awk -F '[ ,]'  '{print $1,$2,$5}' log.txt

# -v设置变量
# 数字值为+1，字符串值为1
awk -va=1 '{print $1,$1+a}' log.txt
# 设置两个变量
awk -va=1 -vb=s '{print $1,$1+a,$1b}' log.txt

# 过滤第一列大于2的行
awk '$1>2' log.txt
# 过滤第一列等于2的行
awk '$1==2 {print $1,$3}' log.txt
# 过滤第一列大于2并且第二列等于'Are'的行
awk '$1>2 && $2=="Are" {print $1,$2,$3}' log.txt

# 内建变量
# $n	当前记录的第n个字段，字段间由FS分隔
# $0	完整的输入记录
# ARGC	命令行参数的数目
# ARGIND	命令行中当前文件的位置(从0开始算)
# ARGV	包含命令行参数的数组
# CONVFMT	数字转换格式(默认值为%.6g)ENVIRON环境变量关联数组
# ERRNO	最后一个系统错误的描述
# FIELDWIDTHS	字段宽度列表(用空格键分隔)
# FILENAME	当前文件名
# FNR	各文件分别计数的行号
# FS	字段分隔符(默认是任何空格)（Field Separator）
# IGNORECASE	如果为真，则进行忽略大小写的匹配
# NF	一条记录的字段的数目（Number for Field）
# NR	已经读出的记录数，就是行号，从1开始（Number of Record）
# OFMT	数字的输出格式(默认值是%.6g)
# OFS	输出记录分隔符（输出换行符），输出时用指定的符号代替换行符（Out of Field Separator）
# ORS	输出记录分隔符(默认值是一个换行符)（Output Record Separate）
# RLENGTH	由match函数所匹配的字符串的长度
# RS	记录分隔符(默认是一个换行符)（Record Separator）
# RSTART	由match函数所匹配的字符串的第一个位置
# SUBSEP	数组下标分隔符(默认值是/034)
awk 'BEGIN{printf "%4s %4s %4s %4s %4s %4s %4s %4s %4s\n","FILENAME","ARGC","FNR","FS","NF","NR","OFS","ORS","RS";printf "---------------------------------------------\n"}{printf "%4s %4s %4s %4s %4s %4s %4s %4s %4s\n",FILENAME,ARGC,FNR,FS,NF,NR,OFS,ORS,RS}' log.txt
# FS
awk 'BEGIN{FS=","} {print $1,$2}' log.txt
# 指定分隔符
awk -F\' 'BEGIN{printf "%4s %4s %4s %4s %4s %4s %4s %4s %4s\n","FILENAME","ARGC","FNR","FS","NF","NR","OFS","ORS","RS";printf "---------------------------------------------\n"} {printf "%4s %4s %4s %4s %4s %4s %4s %4s %4s\n",FILENAME,ARGC,FNR,FS,NF,NR,OFS,ORS,RS}' log.txt
# 指定输出分隔符
awk '{print $1,$2,$5}' OFS=" $ " log.txt

# 使用正则表达式
# ~ 表示模式开始。// 中是模式。
awk '$2 ~ /th/ {print $2,$4}' log.txt
# 忽略大小写
awk 'BEGIN{IGNORECASE=1} /this/' log.txt
# 模式取反
awk '$2 !~ /th/ {print $2,$4}' log.txt
```



## tail

```shell
# -n 查看文件内容，显示后n行内容；默认后10行
tail -n 10 start-dfs.sh

# -F 监控文件末尾改变情况，实时输出
tail -F start-dfs.sh -n 2

# 显示从第100行到末尾
tail -n +100 start-dfs.sh
```



## dirname

```shell
# 获取当前路径的父路径
# 只有文件名 .
dirname a

# /a
dirname /a/b
```



## basename

```shell
# 获取当前路径的文件名
# b
/a/b
```





# 文档编辑

## grep

- <font color=red>查找或匹配文本</font>

```shell
# 在当前目录文件名为file前缀的文件中，查找包含hadoop文本的文件，并输出行内容
grep hadoop file*

# -r 递归方式查找当前目录和其子目录
# -n 打印行号
grep -rn hadoop .

# 通过上一个命令的结果作为输入进行查找
ls -al | grep file
```



## sed

- <font color=red>编辑匹配到的文本</font>

```shell
# -e 执行脚本命令
# a append，在第三行下面追加\后面的内容，追加后打印到控制台，不影响原来的文本
sed -e 3a\hive file1

# i insert，在第二前面插入内容
sed -e 2i\hive file1

# d delete
nl /etc/passwd | sed '2,5d'

# c replace
nl /etc/passwd | sed '2,5c No 2-5 number'

# p print 配合-n使用，仅显示script处理后的结果
nl /etc/passwd | sed -n '5,7p'

# 搜索显示
nl /etc/passwd | sed -n '/root/p'

# 搜寻并替换
# s/要被取代的字串/新的字串/g	
# s表示正则表达式替换，g表示每一行匹配全部替换，不加的话表示每一行替换第一个匹配
ifconfig ens33 | grep 'inet ' | sed 's/^.*inet //g'

# -i 直接修改源文件
sed -i 's/.*doop$/hive/g' file1

# $表示最后一行，a表示追加
sed -i '$a zookeeper' file1
```



## sort

```shell
# 以默认的方式将文本文件的第一列以ASCII码的次序排列，并将结果输出到标准输出
sort file1
```



## uniq

```shell
# -c 统计重复行，但必须相邻，所以先用sort将第一列排序
sort file3 | uniq -c

# -d 仅显示重复出现的行，即次数大于1
sort file3 | uniq -cd
```



## wc

```shell
# 统计 行数、单词数、字节数
wc file1 file2 file3
```



## let

```shell
# 执行表达式，变量不需要加$
let a=5+4
```



## vi/vim

![vi-vim-cheat-sheet-sch](Linux命令.assets/vi-vim-cheat-sheet-sch.gif)

- vi 是老式的字处理器。vim 具有程序编辑的能力，可以主动的以字体颜色辨别语法的正确性，方便程序设计

``` shell
# ******************************************************
# 【命令模式】
# ******************************************************
# 切换到【输入模式】
i
# 切换到【底线命令模式】
:

# 向后删除光标所在字符
# shift+x 向前删除
# nx 连续删除
x
# 删除游标所在行
dd
# 复制游标所在行
# nyy 向下复制n行
yy
# 复制在游标之后
p
# 撤销前一个动作
u
# 重做前一个动作
control+r

# 移动
# 光标向左移动一个字符
h 或 向左箭头键(←)
# 光标向下移动一个字符
# 向下移动30行，可以使用 "30j" 或 "30↓" 的组合按键
# n<Enter> 向下移动n个字符
j 或 向下箭头键(↓)
# 光标向上移动一个字符
k 或 向上箭头键(↑)
# 光标向右移动一个字符
# n<space> 向右移动n个字符
l 或 向右箭头键(→)
# 屏幕『向下』移动一页，相当于 [Page Down]
[Ctrl] + [f]
# 屏幕『向上』移动一页，相当于 [Page Up]
[Ctrl] + [b]
# 移动到行首，相当于 [Home]
0
# 移动到行尾，相当于[ End ]
$
# 移动到当前屏幕上方的第一个字符
shift+h
# 移动到当前屏幕中央的第一个字符
shift+m
# 移动到当前屏幕下方的第一个字符
shift+l
# 移动到文件最后一行的第一个字符
# n+shift+g 移动到文件的第n行
shift+g
# 移动到文件第一行的第一个字符，相当于 1+shift+g
gg

# 搜索
# 从光标向下搜索
/word
# 从光标向上搜索
?word
# 重复上一次搜索
n
# 反向重复上一次搜索
shift+n

# 替换
# n1 与 n2 为数字。在第 n1 与 n2 行之间寻找 word1 这个字符串，并将该字符串取代为 word2 
:n1,n2s/word1/word2/g
# 从第一行到最后一行寻找 word1 字符串，并将该字符串取代为 word2 
# :%s/word1/word2/g 等价
:1,$s/word1/word2/g
# 替换前确认
# :%s/word1/word2/gc
:1,$s/word1/word2/gc
# 在 10 - 20 行添加 // 注释
# 因为要换中有/，所以分隔符用#代替
:10,20s#^#//#g
# 在 10 - 20 行删除 // 注释
:10,20s#^//##g

# 查找关键字出现次数
:%s/word//gn

# ******************************************************
# 【输入模式】
# ******************************************************
# 换行
enter
# 退格键，删除光标前一个字符
back space / delete
# 切换到【命令模式】
esc

# ******************************************************
# 【底线命令模式】
# ******************************************************
# 退出程序
# q! 不保存退出
q
# 保存文件
w
# 切换到【命令模式】
esc
# 保存退出
wq

# 设置行号
# set nonu 取消行号
set nu
# 取消高亮显示
# set hls 高亮显示
set nohls
```



# 文件传输

## ftp

```shell
# 该服务器是Linux内核的官方服务器
ftp ftp.kernel.org
```



## bye

```shell
# 中断与ftp服务器连接并退出程序
bye
```



# 磁盘管理

## cd

```shell
# 切换当前工作目录为目标目录
cd a/b

# 若目录名称省略，则变换至使用者的 home 目录
cd


```

## df

disk free

```shell
# 显示文件系统的磁盘使用情况统计，显示文件系统、挂载点等信息
# -T 显示文件系统类型
df -Th
```

## du

disk usage

```shell
# 只显示当前目录和子目录的大小
du -h

# 显示当前目录的文件大小，子目录，以及递归子目录的大小（子目录下文件不显示大小）
du -h *

# 仅显示当前目录的子目录和文件的总计大小
du -sh *
```



## mkdir

```shell
# -p 如果父目录不存在，则创建
mkdir -p a/b
```



## pwd

print work directory

```shell
# 输出当前工作路径
pwd
```



## mount

``` shell
# 挂载
# 文件系统（分区） -> 目录（挂载点）
mount /dev/hda1 /mnt

# 只读模式挂载
mount -o ro /dev/hda1 /mnt

# 挂载ios文件
mount -o loop /tmp/image.iso /mnt/cdrom
```



## umount

```shell
# 通过文件系统卸载
# -v 执行时显示详细的信息
umount -v /dev/sda1
# 通过挂载点卸载
umount -v /mnt/mymount/
```



## rmdir

```shell
# 删除目录
rmdir dir

# 删除目录dir2，如果此时dir1为空目录，则一并删除
rmdir -p dir1/dir2
```



## stat

```shell
# 显示inode的内容
stat log.txt
```



## ls

```shell
# 默认当前目录，显示所有文件和目录，包括隐藏档
ls -al
```



## iostat

```shell
# -m 每秒兆字节
# -x 显示扩展信息
# 2 每个2秒刷新一次
iostat -mx 2

# 安装iostat
sudo yum install -y sysstat
```



# 磁盘维护

## lsblk

```shell
# 列出所有可用块设备的信息，并显示他们之间的依赖关系
# -p 打印完整设备路径
lsblk -p

# NAME                        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
# /dev/sda                      8:0    0   40G  0 disk
# ├─/dev/sda1                   8:1    0    1G  0 part /boot
# └─/dev/sda2                   8:2    0   39G  0 part
#   ├─/dev/mapper/centos-root 253:0    0   37G  0 lvm  /
#   └─/dev/mapper/centos-swap 253:1    0    2G  0 lvm  [SWAP]
# /dev/sr0                     11:0    1 1024M  0 rom
```



## parted

```shell
 # 显示分区表
 # MBR(msdos) -> fdisk 分区		
 # 		支持4个主分区 或 3个主分区+1个扩展分区（逻辑分区无限制）
 # 		不支持大于2T的分区
 # GPT -> gdisk 分区					无限制
 parted /dev/sda print
```



## fdisk

Partition table manipulator for Linux

- Linux fdisk是一个创建和维护分区表的程序，它兼容DOS类型的分区表、BSD或者SUN类型的磁盘列表
- 容量
  - 硬盘总容量 = 主分区 + 扩展分区
  - 扩展分区容量 = 逻辑分区总容量

```shell
# 列出所有磁盘分区
fdisk -l

# 对磁盘分区
# 	m 显示菜单和帮助信息
# 	n 增加一个新分区
# 	p 主分区（e 扩展分区）输入分区数（1）
# 	w 保存结果
fdisk /dev/sda
```



## gdisk

```shell
# 查看磁盘信息
gdisk -l /dev/sda

# 对磁盘分区
#		? 显示菜单和帮助信息
# 	n 增加一个新分区
# 	p 查看分区表
# 	w 保存结果
gdisk /dev/sda

# ******************************************************
# LVM Logical Volume Manager
# ******************************************************
# partition -> pv -> vg -pe-> lv -> mkfs -> mount
# ******************************************************
# 分区
# 		gdisk -l /dev/vda		建立文件系统（code 8E00）
# pv阶段
#     1）pvscan     查看有无pv
#     2）pvcreate /dev/vda{5,6,7,8}     建立pv
#     3）pvdisplay /dev/vda5     显示更详细pv信息
# vg阶段
#     1）vgcreate -s 16M vbirdvg /dev/vda{5,6,7}     -s指定pe大小
#     2）vgscan     查看vg
#     3）pvscan     查看pv
#     4）vgdisplay vbirdvg      显示更详细vg信息
#     ----
#     vgextend vbirdvg /dev/vda8     可以给现有vg扩充pv
#     vgreduce     在 VG 內移除 PV
#     vgremove     刪除一個 VG
#    ----
# lv阶段
#     1）lvcreate -L 2G -n vbirdlv vbirdvg     -L容量     -n名称
#     2）lvscan     查看lv
#     3）lvdisplay /dev/vbirdvg/vbirdlv     显示更详细lv信息
#     ----
#     lvextend     在 LV 裡面增加容量     
#						lvextend /dev/vbirdvg/vbirdlv /dev/vda8（vda8必须属于vbirdvg，这个pv全部给vbirdlv）
#     lvreduce     在 LV 裡面減少容量
#     lvremove     刪除一個 LV
#     lvresize     對 LV 進行容量大小的調整
#     ----
# 格式化
#     1）mkfs.xfs /dev/vbirdvg/vbirdlv
#     2）mkdir /srv/lvm     目录
#     3）mount /dev/vbirdvg/vbirdlv /srv/lvm     文件系统挂载到目录
#     4）df -Th /srv/lvm     检查
#     ----
#     xfs_info /srv/lvm     查看文件系统信息
#     xfs_growfs /srv/lvm     lv增大后，要对 目录/文件系统 操作 xfs_growfs ，空间才会相应变化
#     ----
```



## partprobe

```shell
# 查看磁盘分区是否有变化
partprobe /dev/sda

# 不重启使分区生效
partprobe -s
```



## mkfs

```shell
# 分区格式化为xfs
# 等价于 mkfs -t xfs
mkfs.xfs /dev/sda1
```



# 网络通讯

## telnet

```shell
# 登录IP为 192.168.0.5 的远程主机
telnet 192.168.0.5 
```



## ifconfig

```shell
# 显示网络设备信息
ifconfig

# 给eth0网卡配置IP地址，子网掩码，广播地址
ifconfig eth0 192.168.1.56 netmask 255.255.255.0 broadcast 192.168.1.255
```



## netstat

```shell
# -a 列出所有的连接状态，包括 tcp/udp/unix socket 等
# -u udp only 
# -t tcp only
# -l 仅列出有在 Listen 的服务网络状态
# -n 不使用主机名称和服务名称，使用 IP 和 port number
# -P 列出pid
netstat -tulnp

# 安装netstat
yum install -y net-tools
```



## ping

```shell
# 检测是否与主机连通
# -c 指定接收包的次数
ping -c 2 www.baidu.com
```



## ssh

```shell
# 1、登录远程服务器，接着输入密码
# 2、iTerm远程连接centos服务器，显示命令帮助为中文，而centos服务器内部可以显示英文
#    centos语言为US，iTerm语言为CN，因此需要修改iTerm：
#    1）vim ~/.zshrc
#    2）末尾添加
#        export LC_ALL=en_US.UTF-8
#        export LANG=en_US.UTF-8
#    3）source ~/.zshrc
ssh root@172.16.92.132
```



# 系统管理

## useradd

- `/etc/passwd` 账号信息

```shell
# 添加用户并指定组
useradd -g hadoop test
```



## usermod

```shell
# 修改用户所属组
usermod -g test test
```



## userdel

```shell
# -r 删除用户登入目录以及目录中所有文件
# 如组中只有一个用户，则删除此用户时，组一并删除
userdel -r test
```



## groupadd

- `/etc/group` 组账户信息

```shell
# 添加组
groupadd test
```



## groupmod

```shell
# 修改组信息
groupmod -n test1 test
```



## groupdel

```shell
# 删除组
# 若组中存在用户，则不允许删除
groupdel test1
```



## date

```shell
# 显示当前日期时间
date

# + 设定格式标记
# %c 直接显示日期与时间
date '+%c'

# 指定日期加100天
date -d"20200801 +100 days" +"%Y%m%d"
```



## exit

```shell
# 退出shell
exit
```



## sleep

```shell
# number 时间长度，后面可接 s、m、h 或 d，其中 s 为秒，m 为 分钟，h 为小时，d 为日数
date;sleep 30s;date
```



## kill

```shell
# 显示信号
kill -l

# 彻底杀死进程
kill -9 pid

# 正常停止一个进程
kill -15 pid
```



## ps

```shell
# -e 显示所有进程
# -f 全格式列表
# 显示所有命令，连带命令行
ps -ef

# -u 指定用户的进程
ps -u hadoop
ps -u hadoop -f
```



## nice

```shell
# 降低优先级，设置优先级为19
# 范围是[-20, 19]，默认值是10。nice值越大优先级越低，优先级高的会抢占优先级低的进程的时间片
nice -n 19
```



## top

```shell
# 实时显示进程信息
top

# 输入h显示帮助信息
h

# 显示多核CPU使用率
1
```



## shutdown

```shell
# 立即关机
shutdown -h now

# 重新启动
shutdown -r now
```



## sudo

```shell
# 以系统管理者的身份执行指令
sudo ls

# 以指定用户身份执行指令
sudo -u hadoop ls -l
```



## uname

```shell
# 显示系统信息
uname -a

# 显示计算机名
uname -n
```



## who

```shell
# 显示当前登录系统的用户
# -H 显示标题栏
who -H
```



## whoami 

```shell
# 显示用户名
whoami
```



## su

```shell
# 切换到root用户
su root

# 切换用户，改变环境变量
su - root
```



## cpuinfo

```shell
# 物理CPU个数
cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l

# 每个物理cpu中core的个数(即核数)
cat /proc/cpuinfo | grep "cpu cores" | uniq
```



## free

```shell
# 显示内存使用情况
free -h
```



## locale

```shell
# 显示目前所支持的语系
locale
```



## jobs

```shell
# & 在后台运行
vi start-dfs.sh &

# 查看当前bash中有哪些工作
jobs -l

# 从后台调到前台运行
fg %1

# 从前台暂停到后台运行
control+z

# 结束工作任务
control+c
```



## echo

```shell
# -e 解释转义字符
echo -e "You know nothing, Jon Snow.\n\t- Ygritte"

# -E 禁用转义字符的解释（默认）
echo -E  "You know nothing, Jon Snow.\n\t- Ygritte"

# -n 忽略尾部的自动换行
echo -n 'hello'
```



# 系统设置

## clear

```shell
# 清屏
clear
```



## crontab

```shell
# ******************************************************
# *    *    *    *    *
# -    -    -    -    -
# |    |    |    |    |
# |    |    |    |    +----- 星期中星期几 (0 - 7) (星期天 为0)
# |    |    |    +---------- 月份 (1 - 12) 
# |    |    +--------------- 一个月中的第几天 (1 - 31)
# |    +-------------------- 小时 (0 - 23)
# +------------------------- 分钟 (0 - 59)
# ******************************************************
# * 表示每分钟都要执行，以此类推
# a-b 表示从第 a 分钟到第 b 分钟这段时间内要执行，以此类推
# */n 表示每 n 分钟时间间隔执行一次，以此类推
# a, b, c,... 表示第 a, b, c,... 分钟要执行，以此类推
# ******************************************************

# 在12月内，每天的早上6点到12点，每隔3小时0分钟执行一次/usr/bin/backup
0 6-12/3 * 12 * /usr/bin/backup

# 每月每天的午夜0点20分开始，每2小时，直到23点20分为止执行echo "haha"
20 0-23/2 * * * echo "haha"
```



## declare

```shell
# 声明变量，同时赋值
declare -i a=1

# 声明只读变量
declare -r a=1

# 声明环境变量
declare -x a=1
```



## export

```shell
# 列出当前的环境变量值
export -p

# 定义环境变量并赋值
# 仅对当前登录有效
export MYENV=7
```



## rpm

redhat package manager

```shell
# 安装软件
# -h 套件安装时列出标记
# -v 显示指令执行过程
# -i 安装指定的套件档
rpm -hvi dejagnu-1.4.2-10.noarch.rpm

# 显示软件安装信息
# -q 使用询问模式，当遇到任何问题时，rpm指令会先询问用户
rpm -qi dejagnu-1.4.2-10.noarch.rpm
```



## yum

```shell
# 线上安装
yum install -y vim
```



## set

```shell
# 显示环境变量
set
```



## env

```shell
# 显示环境变量
env
```



## passwd

```shell
# 修改用户密码
# 当提示 ’无效的密码： 密码少于 8 个字符‘ 无需管，重新输入密码即可
passwd test
```



# 备份压缩

## zip

``` shell
# -q 不显示指令执行过程
# -r 递归处理，将指定目录下的所有文件和子目录一并处理
zip -qr test.zip *

# -d 从压缩文件内删除指定的文件
zip -d test.zip log.txt
```



## unzip

```shell
# 显示压缩包中的文件
unzip -l test.zip

# 解压缩到指定目录
unzip test.zip -d mytest
```



## zipinfo

```shell
# 显示压缩文件信息
zipinfo test.zip

# 显示压缩文件中每个文件的信息
zipinfo -v test.zip
```



## tar

```shell
# 压缩文件
# -c create
# -z 通过gzip指令处理备份文件
# -v 显示指令执行过程
# -f 指定备份文件
tar -czvf test.tar.gz file*

# 列出压缩文件内容
# -t 列出备份文件的内容
tar -tzvf test.tar.gz

# 解压文件
# -x 从备份文件中还原文件
# -C 解压到指定目的目录
mkdir t1 & tar -xzvf test.tar.gz -C t1
```





















