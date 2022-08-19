---
title: linux find 按文件时间查询
tags:
  - Linux
date: 2019-11-23
---

## 需求:删除 /home/ljbao/myback/files 目录下7分钟之前生成的gz类型的文件

```text
find /home/ljbao/myback/files -mtime +7 -name "*.gz" -exec rm {} \;
```

//时间条件

```text
-amin n: 查找n分钟以前被访问过的所有文件。
-atime n: 查找n天以前被访问过的所有文件。
-cmin n: 查找n分钟以前文件状态被修改过的所有文件。
-ctime n: 查找n天以前文件状态被修改过的所有文件。
-mmin n: 查找n分钟以前文件内容被修改过的所有文件。
-mtime n: 查找n天以前文件内容被修改过的所有文件。
```
