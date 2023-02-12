---
title: git本地仓库与github仓库关联命令
date: 2020-04-20 16:28:13
tags:
categories:
- Git教程
---

```
cd test/
ls
git init
git add .
git commit -m 'first commit'
git status
git remote add origin git@github.com:zhx2020/test.git
git remote rm origin
git remote -v
git pull --rebase origin master
git push -u origin master
```

+ 进入到需要推送的文件夹
+ 初始化目录：`git init`
+ 添加所有文件到本地库：`git add .`
+ 提交文件到本地库：`git commit -m "discription"`
+ 将本地库和远程库进行关联：`git remote add origin git@github.com:zhx2020/test.git`
+ 取消本地库与远程库的关联：`git remote rm origin`
+ 查看远程库的信息：`git remote -v`
+ 拉取远程库文件同步到本地：`git pull --rebase origin master`
+ 将本地库文件推到远程库：`git push -u origin master`

参考

+ [git本地仓库与github仓库关联命令](https://www.jianshu.com/p/54630079c7df)
