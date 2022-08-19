---
title: windows共享开启
date: 2019-10-15
---

## Windows共享命令

1. `net share`

```text
C:\Users\linjinbao666>net share

共享名       资源                            注解

-------------------------------------------------------------------------------
C$           C:\                             默认共享
D$           D:\                             默认共享
E$           E:\                             默认共享
F$           F:\                             默认共享
IPC$                                         远程 IPC
ADMIN$       C:\Windows                      远程管理
命令成功完成。
```

1. net share blog=E:\amrom

```text
PS E:\> net share blog=E:\amrom
blog 共享成功。

PS E:\> net share

共享名       资源                            注解

-------------------------------------------------------------------------------
IPC$                                         远程 IPC
C$           C:\                             默认共享
D$           D:\                             默认共享
E$           E:\                             默认共享
F$           F:\                             默认共享
ADMIN$       C:\Windows                      远程管理
blog         E:\amrom
命令成功完成。
```

1. 删除共享目录

```text
PS E:\> net share blog /del
blog 已经删除。

PS E:\> net share

共享名       资源                            注解

-------------------------------------------------------------------------------
C$           C:\                             默认共享
D$           D:\                             默认共享
E$           E:\                             默认共享
F$           F:\                             默认共享
IPC$                                         远程 IPC
ADMIN$       C:\Windows                      远程管理
命令成功完成。
```

## net use使用学习（重要）
