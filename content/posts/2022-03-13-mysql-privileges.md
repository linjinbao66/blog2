---
title: mysql权限高级
date: 2022-03-12
---

**问题：**
如何创建一个新用户，并屏蔽旧库权限？

**答案：**
问题是对mysql的权限理解有问题。
mysql的权限有2个操作符,grant(授予)和revoke(撤销)。旧库的权限是不能屏蔽的，必须每一个剔除允许的列表。

操作实例

```code
create user 'test'@'%' identified by '123456';
grant all privileges on `test%` to 'test'@'%';
```

以上操作会给予test用户，作用域test开头的所有库的所有权限。

```code
revoke all privileges on test.* from 'sql'@'%';
```

以上操作撤销test用于针对test库的操作。
