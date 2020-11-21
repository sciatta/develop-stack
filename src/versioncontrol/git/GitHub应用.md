# 向开源项目提交 Pull request

## 将开源项目fork到远程origin仓库

`https://github.com/sciatta/shardingsphere`



## 将远程origin仓库clone到本地仓库

- 将远程origin仓库clone到本地仓库 `git clone git@github.com:sciatta/shardingsphere.git`

- 通过 `git status` 查看当前所在分支 On branch master

- 通过 `git remote -v` 查看远程仓库映射关系

  ```shell
  origin	git@github.com:sciatta/shardingsphere.git (fetch)
  origin	git@github.com:sciatta/shardingsphere.git (push)
  ```



## 与上游upstream仓库建立映射关系

- 建立映射 `git remote add upstream https://github.com/apache/shardingsphere.git`

- 通过 `git remote -v` 查看远程仓库映射关系

  ```shell
  origin	git@github.com:sciatta/shardingsphere.git (fetch)
  origin	git@github.com:sciatta/shardingsphere.git (push)
  upstream	https://github.com/apache/shardingsphere.git (fetch)
  upstream	https://github.com/apache/shardingsphere.git (push)
  ```

  

## 创建本地仓库develop分支开发

- 创建develop分支 `git checkout -b develop`
- 在此分支上进行开发 `git add -A`
- 提交本地仓库 `git commit -m` 



## 提交远程origin仓库

- 本地仓库拉取远程upstream仓库的最新内容 `git fetch upstream`
- 切换到本地master分支 `git checkout master`
- 本地仓库和远程upstream仓库的master分支同步 `git rebase upstream/master`
  - <font color=red>rebase</font> 合并为一条时间轴。如在master中执行git rebase develop，找到master和develop的公共祖先，祖先先合并develop的新增提交，然后在后面追加master的新增提交。**注意必须没有待提交的文件**
  - <font color=red>merge</font> 合并路径为分叉时间轴。如在master中执行git merge develop，找到master和develop的公共祖先，然后由公共祖先、master最新提交和develop最新提交，三方合并一个新的提交
- 切换到本地develop分支 `git checkout develop`
- 合并最新master分支 `git rebase master` 
- 将本地develop分支提交到origin仓库 `git push origin develop:develop`



## 提交 Pull Request

在远程origin仓库 `https://github.com/sciatta/shardingsphere` 提交 Pull request