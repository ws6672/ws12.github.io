---
title: 恢复git删除的分支
date: 2021-05-21 15:21:52
tags: [tool]
---


误删分支可以分为以下几种情况：
+	分支数据存在本地，远端没有最新分支数据，通过本地删除
+	分支数据存在远端，通过本地删除
+	分支数据存在远端，通过github网页删除，而本地没有分支数据

Git会自行负责分支的管理，所以当我们删除一个分支时，Git只是删除了指向相关提交的指针，但该提交对象依然会留在版本库中。如果在本地进行删除，那么一般会返回一个相关提交的哈希值，通过该哈希值，可以找回分支。


1. 删除本地分支如何恢复

在删除分支的位置打开bash，输入以下命令：
``` 
git log -g 
git branch newbranch commit_id
git checkout newbranch
```

2. 在本地误删远程分支
```
-- 按时间格式输出HEAD(当前分支)日志
git reflog --date=iso
	ce0eee6a HEAD@{2021-05-21 14:53:26 +0800}: commit: XXX
	第一个序列号，就是相关分支ID	
git checkout -b recovery_branch_name commit_ID
-- 恢复后推送远端即可
git push  origin recovery_branch_name 
```

3. 恢复在网页端删除的远端分支

暂时没找到法子，因为网页删除后不会返回哈希之类的信息；只能通过本地推送恢复。

4. 删除分支的相关命令(慎用)

```
git checkout other
-- 本地
git branch -d XX
-- 强制删除
git branch -D XX
-- 删除远端分支
git push origin --delete XX
-- 远端拉取分支
git fetch origin XX1:XX1
git checkout XX1
```