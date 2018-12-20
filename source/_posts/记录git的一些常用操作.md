---
title: 记录git的一些常用操作
date: 2018-12-20 19:46:00
tags: git
categories: 工具
comments: true
---


## 记录账号
vscode或linux等环境里，每次提交git总需要输入密码，可以输入账号密码后保存下。
首次的话添加全局参数，可设置或修改账号:
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com

然后输入完账号密码后记录账号密码：
git config --global credential.helper store
<!-- more -->
## 分支管理
默认已有master主干分支，添加一个名字为dev的分支

查看分支：git branch
创建分支：git branch dev
分支提交到服务器：git push origin dev (master主分支默认的origin就是master 直接git pull和git push就好)

切换到分支：git checkout dev  （切换到分支后代码也会立即变为分支代码）
从分支拉新：git pull origin dev (或vscode里-拉取自)

从分支clone：git clone https://github.com/kiwiyan/branch-test.git -b dev

将分支dev合并到master: 
首先，切换到maste分支：git checkout master
接着，合并：git merge dev
可能有冲突，那就修改冲突部分，然后commit (可借助vscode的冲突处理-接受分支还是主干项)

## 回滚到制定版本

查看历史版本
git log 

修改为制定版本：
git reset --hard 139dcfaa558e3276b30b6b2e5cbbb9c00bbdca96  

把修改推到远程服务器：
git push -f -u origin master  

## 本地代码发布到线上仓库
1. gtihub上新建仓库，生成仓库链接。创建时可以先不添加md文件，之后由本地添加。
2. git init 在本地文件根路径下将项目初始为git项目。
3. git remote add origin https://github.com/xxx/xxxx.git  添加要推送到的刚新建的远程仓库链接。
4. git push -u origin master  推送到master主分支。
当然还有一种简单粗暴的方式， 将新创建的git仓库clone下来，用里边的.git文件夹替换你本地要上传的文件夹里的.git也可以，之后更改推送等操作照常


