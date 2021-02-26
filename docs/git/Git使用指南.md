# Git 使用指南

## 如何创建Git项目
创建Git项目有两种方式，本地创建和Clone
1. 本地创建
> git init [projectName]    # 初始化Git项目，命令会创建目录projectName
> git remote add origin [git@gitee.com:childepeng/gittest2.git]     # 设置项目的远程仓库地址
2. 从远程仓库clone已有的项目，（github\gitee等）
> git clone xxxx

## 创建一次commit
> touch README      # 创建README文件
> git add README    # 将文件加入到暂存区
> git commit -m 'add readme'    # 提交修改


