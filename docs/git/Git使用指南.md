# Git 使用指南

## Git基础

![](https://static.laop.cc/images/git.jpg)

Git数据存储区域

1. workspace（工作区）
2. index（暂存区）
3. Respository（本地版本库）
4. Remote（远程仓库）

文件状态： 

1. untracked：未加入到版本库的文件，比如未执行add的新建文件
2. modified：文件有变更，未暂存
3. staged：文件已暂存
4. commited：文件已提交到本地仓库
5. unmodified：文件未变更

## 如何创建Git项目

创建Git项目有两种方式，本地创建和Clone
1. 本地创建
```shell
git init gittest2 	 	# 初始化Git项目，命令会创建目录gittest2
git remote add origin [git@gitee.com:childepeng/gittest2.git]     # 设置项目的远程仓库地址
```
2. 从远程仓库clone已有的项目，（github\gitee等）

```shell
git clone git@gitee.com:childepeng/gittest2.git
git clone git@gitee.com:childepeng/gittest2.git  gittest3 	# 重命名	
```

## 创建一次commit
```shell
touch README      # 创建README文件
git add README    # 将文件加入到暂存区； 可以指定文件和目录，也可以使用 . 或者 *
git commit -m 'add readme'    # 提交修改；也可以不加参数，使用编辑器输入提交日志；
```

git使用nano作为默认编辑器，可用于输入提交日志、编辑文件冲突等；

可执行如下代码修改vim为默认编辑器：

```shell
git config -global core.editor vim
```

## 撤销

1. 提交完成之后，发现有未提交文件，或者又有修改必须要与前面的修改合并提交，可以使用`--amend`， 

```shell
git commit -m "first commit"		# 第一次提交
git add .							
git commit --amend					# 第二次提交，两次的提交会合并，提交日志会覆盖前一次
```

2. 执行 `add` 暂存文件之后，发现有文件不需要提交，可以使用 `reset` 或者 `restore` 撤销暂存

```shell
git  add *			
git  reset  HEAD  reset_file	# 取消暂存指定文件，执行之后 reset_file 会被移出暂存，状态改为 modified
git  restore  --staged  reset_file  # 同上，可以取消暂存指定文件，git最新版本建议使用 restore 进行取消暂存；
```

3. 还原文件的修改

```shell
git  checkout  reset_file	# checkout执行之后，工作区的文件会被还原到最近一次的提交，也就是修改会丢失
```

4. 已经 commit 的文件无法撤销

## 版本回退

1. 回退到指定版本

```shell
# --soft 
# 回退到指定版本，不改变暂存区和工作区，仅将本地仓库回退到指定版本
#（实际上是将HEAD指针移动到指定版本，因为只是移动了指针，因此还能将指针移动回最新版本）
git reset --soft <commit-id>
# 'HEAD^' 可以表示当前上一个提交版本
git reset --soft HEAD^	

# --mixed
# 回退到指定版本，将本地仓库和暂存区回退到指定版本，工作区不变
# mixed是reset命令的默认选项
git reset --mixed <commit-id>

# --hard 
# 本地仓库、暂存区、工作区都将回退到指定版本，操作会移除工作区的修改，慎用！
git reset --hard <commit-id>
```

## 强制更新

如果本地项目改崩了，需要强制恢复到远程仓库的版本

```shell
git fetch --all		# 拉取远程仓库的版本
git reset --hard origin/main	# 重置本地版本（本地仓库、暂存区、工作区全部重置到远程仓库的版本）
git pull			
```

fetch: 是将远程仓库的最新内容拉到本地，之后用户再检查是否合并到本地分支；pull: 是将远程仓库的最新内容拉取到本地并直接合并，git pull = git fetch + git merge；

## 远程仓库

一个项目可以设置多个远程仓库，每个远程仓库都可以设置一个别名，默认为 `origin`，并且fetch/push可以指定不同的仓库地址，默认是一样的；

1. 远程仓库查看与设置

```shell
git remote -v 	# 查看远程仓库
git remote add origin git@gitee.com:childepeng/gittest2.git		# 添加远程仓库
git remote remove origin  # 移除远程仓库
```

2. 代码同步

```shell
git pull origin master		# 拉取远程分支代码到本地，并执行合并
git push --set-upstream master	# 将本地代码提交到远程分支
```

## Tag标签

Git可以给仓库历史某个提交上设置标签，比如标记软件的发布节点等；

1. 创建标签

```shell
git tag 	# 查看标签
git tag v0.9 [commitid]	# 创建轻量级标签，v0.9为标签名，可以在指定的commit上设置标签
git tag -a v1.0 [-m 'message'] [commitid]	# 创建附注标签， -m设置标签日志
```

Git 将标签分为轻量标签（lightweight）和附注标签（annotated）；

轻量标签是指定commit的引用，通常用于创建一些不重要的标签、临时标签等；

附注标签在Git数据库中单独存储，包含更多信息，比如提交者、时间、描述信息、commitid等等；

2. 查看标签

```shell
git tag		# 查看所有标签，列表展示
git show  v0.9	# 查看指定标签，如果是轻量标签，这里展示的是对应的Commit信息；
```

3. 标签共享

本地创建的标签，需要通过 push 操作，推送到远程仓库

```shell
git push origin v0.9  # 推送指定标签
git push origin --tags # 推送所有标签
```

4. 删除标签

```shell
git tag -d v0.9
```

5. 根据标签检出代码

```shell
git checkout <tagName>
```

一般通过标签检出的代码不允许直接修改，如果要修改指定标签的代码，需要创建新的分支

```shell
git checkout -b <branch> <tag>
```

## 分支

Git 的分支，其实本质上仅仅是指向提交对象的可变指针。 Git 的默认分支名字是 master。 在多次提交操作之 后，你其实已经有一个指向最后那个提交对象的 master 分支。 master 分支会在每次提交时自动向前移动。

**HEAD** 是一个特殊的指针，指向当前所在分支；

1. 创建和切换分支（branch、checkout）

```shell
git branch <branch-name> 	# 创建分支，当前分支不会发生切换
git checnkout <branch-name>	# 切换分支
git checkout -b <branch-name>	# 切换分支，如果分支不存在就先创建分支
```

2. 分支合并（merge）

   分支合并之前需要先明确哪个是并入分支；比如我在dev分支上完成了一个功能，需要合并到master分支，master就是分支，因此在dev分支已提交的前提下，先切换到master分支，然后执行merge操作；

```shell
git checkout master		# 切换回master分支
git merge dev			# 将dev分支代码合并到master分支
```

3. 分支删除

   已合并的临时分支，可以直接删除

```shell
git branch -d <branch-name> 	# 删除本地分支
git branch -r -r origin/<branch-name> 	# 删除远程分支
```



## 其他

1. log

```shell
git log		# 查看日志
git log --oneline 	# 查看日志（简化显示）
git log --grahp		# 图形化展示
```

2. status

```shell
git status 	# 展示当前状态
```



