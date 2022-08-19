---
title: git单独忽略一个文件的历史提交
tags: ["git"]
date: 2021-03-19
---

使用场景：

将一个重要文件，勿加入git的版本管理中了，后续又更新了好多版本，此时想要去除该文件的所有历史提交。

* 方案一：删除所有git记录
* 方案二：单独删除该文件的历史记录

方案二操作如下：

```code
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch visitor-consumer/src/main/java/com/fline/util/SMSUtil.java' --prune-empty --tag-name-filter cat -- --all

git push origin --force --all

git push origin --force --tags

git for-each-ref --format='delete %(refname)' refs/original | git update-ref --stdin

git reflog expire --expire=now --all

git gc --prune=now
```
