# Git配置

Git 提供 git config 工具，专门用来配置或读取相应的工作环境变量。

- /etc/gitconfig 对所有用户有效，`git config --system` 读取此文件
- ~/.gitconfig 对当前用户有效，`git config --global` 读取此文件
- 工作目录中的 .git/config 对当前项目有效，`git config` 读取此文件

## 用户信息

```shell
git config --global user.name "rain"
git config --global user.email 15058553030@163.com
```



## 文本编辑器

```shell
git config --global core.editor emacs
```



## 差异分析工具

```shell
git config --global merge.tool vimdiff
```



## 查看配置信息

```shell
# 查看全部
git config --list

# 查看某一个
git config user.name
```



# 仓库管理

## git init

在执行完成 git init 命令后，Git 仓库会生成一个 .git 目录，该目录包含了资源的所有元数据，其他的项目目录保持不变。默认创建master分支。

```shell
# 在当前目录生成.git目录
git init

# 指定目录作为仓库
git init newrepo
```



## git clone

```shell
# 会在当前目录创建hadoop-main目录
git clone https://github.com/sciatta/hadoop-main.git

# 指定本地目录
git clone https://github.com/sciatta/hadoop-main.git hadoop
```



## git remote

git本地仓库同github仓库映射有两种方式：

1. 先建立本地仓库，然后再同github仓库关联；适用于将已有项目托管到github。

   ```shell
   # 创建本地仓库
   git init
   # 关联本地仓库和远程仓库
   git remote add origin https://github.com/sciatta/hadoop-main.git
   # 推送更新
   git push origin master
   ```

   

2. 先建立github仓库，然后再同本地仓库关联；适用于在开发之前，先在github托管项目。

   ```shell
   # 创建本地仓库，同时关联
   git clone https://github.com/sciatta/hadoop-main.git
   ```

   

``` shell
#  查看远程仓库
git remote

# 别名对应的实际链接地址
# 如：
# origin	https://github.com/sciatta/hadoop-main.git (fetch)
# origin	https://github.com/sciatta/hadoop-main.git (push)
git remote -v

# 关联远程仓库，别名同实际链接地址做映射
git remote add github https://github.com/sciatta/hadoop-main.git
# 取消关联
git remote rm github
```



## git fetch

<font color=red>fetch和pull的区别</font>

- head指向本地分支master；remotes指向远程分支orgin/master
- fetch只更新remotes；需要手动merge
- pull更新remotes和head

```shell
# 从远程仓库下载最新分支与数据
# 此命令执行完后需要执行 git merge origin/master 远程分支到你所在的 master 分支
git fetch origin master
```



## git pull

```shell
# 拉取远程仓库origin的master分支到本地
git pull orgin master
```



## git push

```shell
# 将master分支推送到远程仓库origin的master分支
git push origin master
```



# 快照操作

## git add

将文件添加到缓存区

```shell
# 将README文件添加到缓存
git add README

# 添加当前目录的所有文件
git add -A
```



## git status

```shell
# 查看项目的当前状态
git status

# 显示简要状态
git status -s
```



## git diff

查看执行 git status 的结果的详细信息

```shell
# 未缓存的改动
git diff

# 已缓存的改动
git diff --cached

# 查看已缓存的与未缓存的所有改动
git diff HEAD

# 显示摘要
git diff --stat
```



## git commit

将缓存区内容添加到仓库中

```shell
# 添加到本地仓库，-m 后跟注释
# 如果没有 -m 注释的话，git会打开一个文本编辑器
git commit -m '第一次版本提交'

# 跳过 add
git commit -a

# 修改上一次提交，可用于修改误写注释
git commit --amend
```



## git reset HEAD

取消已缓存的内容

```shell
git reset HEAD README
```

取消push的内容

```shell
# 误提交远程版本
git checkout 5.0.x
# 查看待回滚的版本号
git log
# 本地回滚
git reset --soft d159cb65915840b2dc2297ee6b40309b8aa5d2aa
# 提交远程更新
git push –-force 



## git rm

从已跟踪文件清单中移除

​```shell
# 从已跟踪文件清单中移除，同时清除工作目录文件
git rm README

# 从已跟踪文件清单中移除，工作目录文件仍然存在
git rm --cached README

# 递归删除
git rm –r * 
```



## git mv

```shell
# 移动或重命名一个文件、目录、软连接
git mv README README.md
```



## git log

```shell
# 回顾提交历史
git log

# 简洁
git log --oneline

# 开启拓扑图选项
git log --graph

# 逆向显示所有日志
git log --reverse
```



# 分支管理

## git branch

```shell
# 创建分支
git branch abc

# 没有参数时，列出当前的本地分支
git branch

# 列出本地和远程分支
git branch -a

# 删除分支
git branch -d abc

# 设置默认push分支
git branch --set-upstream-to=origin/5.0.x-notes 5.0.x-notes
```



## git checkout

```shell
# 切换分支
git checkout abc

# 创建分支，之后立即切换到此分支
git checkout -b abc

# 以5.0.x分支为基础，创建5.0.x-notes分支并切换
git checkout -b 5.0.x-notes 5.0.x
```



## git merge 

``` shell
# 合并分支
# 假设当前分支是abc，则将test分支内容合并到当前abc分支
git merge test

# 合并时发生冲突
<<<<<<< HEAD
当前分支代码
=======
合并分支代码
>>>>>>> test

# 修改后 git add 通知 git 冲突解决完毕
```



# 标签管理

如果达到一个重要的阶段，并希望永远记住那个特别的提交快照，可以使用 git tag 给它打上标签。

## git tag

```shell
# 创建带注释的标签
git tag -a v1.0

# 为历史提交版本创建标签
git tag -a v0.9 85fc7e7

# 查看所有标签
git tag
```

