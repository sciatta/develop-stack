# Linux命令

# A



# B

## bye



# C

## cat

Concatenate FILE(s), or standard input, to standard output.

```
cat file1 file2 > file3
1、将file1内容和file2内容输出到file3
2、“命令 > 文件”将标准输出重定向到一个文件中（清空原有文件的数据）
3、“命令 >> 文件”将标准输出重定向到一个文件中（追加到原有内容的后面）

cat -n file1 file2 > file3
由1开始对所有输出的行编号，包括空行

cat -b file1 file2 > file3
由1开始对所有输出的行编号，不包括空行

cat
拷贝标准输入到标准输出

cat file1
拷贝file1到标准输出

cat file1 file2
拷贝file1和file2到标准输出

cat /dev/null > file1
1、清空file1的内容
2、dev/null：在类Unix系统中，/dev/null称空设备，它丢弃一切写入其中的数据（但报告写入操作成功），
读取它则会立即得到一个EOF

cat file1 > /dev/null
不会得到任何信息，因为将本来该通过标准输出显示的文件信息重定向到了/dev/null中
```

## chgrp

## chmod

## chown

## cp

Copy SOURCE to DEST, or multiple SOURCE(s) to DIRECTORY.

```
-a：此选项通常在复制目录时使用，它保留链接、文件属性，并复制目录下的所有内容。其作用等于dpR参数组合。
-d：复制时保留链接。这里所说的链接相当于Windows系统中的快捷方式。
-f：覆盖已经存在的目标文件而不给出提示。
-i：与-f选项相反，在覆盖目标文件之前给出提示，要求用户确认是否覆盖，回答"y"时目标文件将被覆盖。
-p：除复制文件的内容外，还把修改时间和访问权限也复制到新文件中。
-r：若给出的源文件是一个目录文件，此时将复制该目录下所有的子目录和文件。
-l：不复制文件，只是生成链接文件。

cp -r test test1
复制test目录下的内容到test1
```

## cd

dirName：要切换的目标目录。可为绝对路径或相对路径。"~" 表示为 home 目录，"." 则是表示目前所在的目录，".." 则表示目前目录位置的上一层目录。

```
cd a/b
切换当前工作目录为目标目录

cd
若目录名称省略，则变换至使用者的 home 目录
```

# D

## df

Show information about the file system on which each FILE resides, or all file systems by default.

```
df -h
显示文件系统的磁盘使用情况统计
Filesystem               Size  Used Avail Use% Mounted on
显示文件系统、挂载点等信息
```

## du

Summarize disk usage of each FILE, recursively for directories.

```
du -h
只显示当前目录和子目录的大小

du -h *
显示当前目录的文件大小，子目录，以及递归子目录的大小（子目录下文件不显示大小）

du -sh *
仅显示当前目录的子目录和文件的总计大小
```

# E



# F

## find

search for files in a directory hierarchy

```
find . -name "file*"
查找当前目录和子目录下所有匹配样式file*的文件或目录，输出路径

find . -name "*.lastUpdated" | xargs rm -rf
```

## tp

# G

## grep

Search for PATTERN in each FILE or standard input.

```
grep java file*
在当前目录文件名为file前缀的文件中，查找包含java的文件，并输出行内容

grep -rn java .
递归方式查找当前目录和其子目录

grep -n '2019-10-24 00:01:11' *.log
```

# H



# I



# J



# K



# L

## less

Less is a program similar to more, but which allows backward movement in the file as well as forward movement.  Also, less does not have to read the entire input file before starting, so with large input files it starts up faster than text editors like vi.

G	移动到最后一行

g	移动到第一行

/字符串	向下搜索“字符串”

?字符串	向上搜索“字符串”的功能

n	重复前一个搜索（与 / 或 ? 有关）

N	反向重复前一个搜索（与 / 或 ? 有关）

Enter（j）	向下移动一行

k	向上移动一行

空格键（d）	向下翻一页

b	向上翻一页

q（ZZ）	退出less命令

```
ps -ef |less
ps查看进程信息并通过less分页显示

history | less
查看命令历史使用记录并通过less分页显示

less file1 file2
1、浏览多个文件
2、输入
:n  切换到file2
:p  切换到file1
```

## ls

```
ls -al
默认当前目录，显示所有文件和目录，包括隐藏档
```

# M

## more

空格键	向下翻一页

b	向上翻一页



```
more -s file1
如有连续两行以上空白行则以一行空白行显示

more +5 file1
从第5行开始显示
```

## mv

```
mv file1 file
移动并改名

mv ok/ no
ok和no都是目录。当no不存在时，则将ok改名为no；当no存在时，将ok目录转移到no目录。

mv ok/* no
ok和no都是目录。当no不存在时，失败；当no存在时，将ok目录所有内容转移到no目录。
```

## mkdir

Create the DIRECTORY(ies), if they do not already exist.



mkdir [-p] dirName



-p 确保目录名称存在，不存在的就建一个。



```
mkdir -p a/b
在当前工作目录下创建目录，父目录不存在，则一起创建
```



# N



# O

# P

## pwd

Print the full filename of the current working directory.



pwd [--help][--version]



--help 在线帮助。

--version 显示版本信息。



```
pwd
输出当前工作路径
```



# Q



# R

## rcp

## rm

Remove (unlink) the FILE(s).



rm [options] name...



-i 删除前逐一询问确认。

-f 即使原档案属性设为唯读，亦直接删除，无需逐一确认。

-r 将目录及以下之档案亦逐一删除。



```
rm services
rm: remove regular file ‘services’? n
删除文件需要询问

rm -r test
删除目录

rm -rf test
强制删除所有文件和目录（即使只读），不需要询问
```

# S

## ssh

```
ssh root@172.16.92.132
1、登录远程服务器，接着输入密码
2、iTerm远程连接centos服务器，显示命令帮助为中文，而centos服务器内部可以显示英文。
    centos语言为US，iTerm语言为CN，因此需要修改iTerm。
    1）vim ~/.zshrc
    2）末尾添加
        export LC_ALL=en_US.UTF-8
        export LANG=en_US.UTF-8
    3）source ~/.zshrc
```



## scp



# T

## touch

```
touch services
-rw-r--r--. 1 root root 670293 Dec 31 17:10 services
-rw-r--r--. 1 root root 670293 Dec 31 21:07 services
文件存在，更新修改时间为当前时间

touch file1
文件不存在，创建空文件
```

# U



# V



# W

## which

Write the full path of COMMAND(s) to standard output.



which [文件...]



-n<文件名长度>  指定文件名长度，指定的长度必须大于或等于所有文件中最长的文件名。

-p<文件名长度>  与-n参数相同，但此处的<文件名长度>包括了文件的路径。

-w  指定输出时栏位的宽度。

-V  显示版本信息。



```
which bash
which指令会在环境变量$PATH设置的目录里查找符合条件的文件

which -a bash
打印所有匹配的路径（不仅只有第一个）
```

## wc

Print newline, word, and byte counts for each FILE, and a total line ifmore than one FILE is specified.  With no FILE, or when FILE is -,read standard input.  A word is a non-zero-length sequence of charactersdelimited by white space.



wc [-clw][--help][--version][文件...]



-c或--bytes 只显示Bytes数。

-m或--chars 只显示字符数。

-l或--lines 只显示行数。

-w或--words 只显示单词数。

--help 在线帮助。

--version 显示版本信息。



```
wc file1 file2
 2  3 17 file1
 3  4 18 file2
 5  7 35 total
 行数 单词数 字节数，以及总统计值
```



# X



# Y



# Z