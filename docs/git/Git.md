# Git

![](http://static.laop.cc/images/git.jpg)
workspace：工作区
index: 暂存区
Respository: 本地版本库
Remote: 远程仓库
文件状态： 
untracked：未加入到版本库的文件，比如未执行add的新建文件
modified：文件有变更，未暂存
staged：文件已暂存
commited：文件已提交到本地仓库
unmodified：文件未变更

## clone
拉取远程代码
## checkout
在本地版本库创建或切换分支
> git checkout -h               # 帮助
> git checkout -b [branchName]  # 切换分支；-b 表示branch，分支如果不存在就创建分支
> git checkout [./file]         # 从本地分支检出指定文件，会覆盖未暂存的修改
## branch
分支操作
> git branch -h     # branch帮助文档
> git branch        # 参看本地所有分支
> git branch -a     # 查看所有分支，包括远程仓库分支
> git branch [branchName]       # 创建本地分支（不会切换到创建的分支）
> git branch -d [branchName]    # 删除指定分支
## add / commit
### add
将文件加入暂存区
> git add [file]    # 将指定文件加入暂存区
> git add .         # 将当前目录所有修改加入暂存区
### commit
提交修改到本地仓库
> git commit -m "message"   # 提交修改，-m：提交日志
## push / pull
### push
将本地commit推送到远程仓库
> git push

## reset / restore / revert

## status / log

## diff

## merge




