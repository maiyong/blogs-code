---
title: git 基本操作
date: 2020-07-27 10:55:02
tags:
- git
---

## git 常用操作
不会的可以查看git --help

### 创建工作空间

#### clone 
把远程代码克隆到本地
- 克隆默认分支
```
git clone http://10.1.5.50:8081/9fstock-web/ipo-h5.git
```
- 克隆指定分支
```
git clone -b develop http://10.1.5.50:8081/9fstock-web/ipo-h5.git
```

#### init
初始化git目录，使用场景主要是创建本地git管理，如果需要上传到远程服务器，则需要指定远程仓库地址
```
cd existing_folder
git init
git remote add origin http://10.1.5.50:8081/maiyong/my-git.git
git add .
git commit -m "Initial commit"
git push -u origin master
```

### 处理当前修改的文件

#### add 
把快照文件添加到缓存
- 添加单个文件
```
git add fileName
```
- 添加全部文件
```
git add -A
```

#### mv（少用）
重命名或者移动文件
```
git mv README.md READYOU.md
```

#### rm（少用）
删除文件
```
git rm README.md
```

#### restore（少用）
撤销修改，同`git checkout --` 命令，我一般用的是`checkout --`
```
git restore README.md
```

#### commit
把暂存的文件提交版本区
```
git commit // 进入vi编写commit日志
git commit -m "日志" // 直接编写日志，不需要进入vi
git commit --amend // 修改上一次的commit日志，用于结合多个commit节点
```

#### reset
还原文件到某个节点
- 取消暂存
```
git reset // 取消全部文件缓存
git reset  '文件' or '目录' // 取消文件或目录缓存
```
- 把文件重设到某个节点，即把head重制到某个节点
```
git reset '节点'
```

#### cherry-pick
这个是很好用的一个命令，作用是把复制别的commit节点并应用到当前分支，使用场景比如a分支需要使用b分支的某个节点，这时候直接在a分支上去cherry-pick b分支的节点j即可
```
git cherry-pick '节点'
```

### 检查历史和状态

#### bisect（少用）
使用二分法查找出现错误的`commit`节点，多用于找bug。
```
git bisect start [终点] [起点]
git bisect good // 没问题的节点
git bisect bad // 有问题的节点
// 一直重复上述2个命令，直到找到出现问题节点
// dc683cfa3660697c3f80b3299886ff342c93d5d4 is the first bad commit
```

#### diff
与HEAD节点对比文件
- 与工作空间对比
```
git diff // 对比所有的文件
git diff README.md // 对比单个文件
```
- 与缓存空间进行对比
```
git diff --cached [节点] // 与特点节点的缓存空间进行对比  
```

#### log
查看提交日志

#### show（少用）
查看提交的详细内容

#### status
查看工作空间的文件状态

### 分支管理

#### branch
查看、创建、删除分支
```
git branch --all // 查看所有分支
git branch -C 'branchName' // 基于当前节点创建分支
git branch -D 'branchName'  // 删除分支
git branch -M 'branchName' // 重命名
```

#### switch 切换分支（少用）
```
git switch 'branchName'
```

#### checkout
切换、创建分支的集合，还有就是处理未暂存文件
- 切换分支，同git switch
```
git checkout 'branchName' // 切换分支
git checkout -b 'branchName' // 创建分支并切换
```
- 撤销修改
```
git checkout -- '文件' or '目录'
```

#### merge/rebase
这两个的作用都是合并分支，合并分支，会经常出现冲突，主要不同是两个命令生成的节点树不一样。rebase类似于执行一系列的cherry-pick命令，最后生成的节点树是一条线，待合并分支的节点都在主分支节点后。 而merge生成的是两条线，而且节点还是保留在原有位置，如果有冲突的话会自动生成一个merge节点。
- rebase
```
git rebase master
// 解决冲突完成后
git rebase --continue
git rebase --abort // 中止rebase操作
```
- merge
```
git merge master
// 解决冲突完成后
git merge --continue // gti commit 也可以完成合并
```

#### tag
新建、删除标签
```
git tag -a v1.0.0 -m "这是一个tag" // 与commit类似
git tag // 展示所有tag
git push origin v1.0.0 // 推到远程服务器
git tag -a v1.0.1 '分支/节点' // 基于分支或节点新建tag
```

### 与远程仓库交互

#### fetch
```
git fetch // 把远程的更新内容全部拉取下来
```

#### pull
在当前分支上执行fetch 和 merge
```
git pull origin master
```

#### push
把本地内容推到服务器
```
git push origin master
```

## git流程

### 正常迭代
- 基于master拉取代码
- 本地修改完成后推送到远程仓库
- 符合上线标准后，有条件可以别人帮忙review代码并提交合并请求
- 合并到master
- 新建tag

