---
title: MYSQL笔记
tags:
  - MYSQL
  - 数据库
  - 服务端
  - development
categories:
  - 数据库
date: 2019-07-28
---

## MYSQL笔记

### UUID

```sql
select uuid();
```

### 存储过程

存储过程（Stored Procedure）是一种在数据库中存储复杂程序，以便外部程序调用的一种数据库对象。 存储过程是为了完成特定功能的SQL语句集，经编译创建并保存在数据库中，用户可通过指定存储过程的名字并给定参数\(需要时\)来调用执行。 创建存储过程

```sql
DELIMITER //
CREATE PROCEDURE getAllUsers()
    BEGIN
    SELECT * FROM `user`;
    END //
DELIMITER ;
CALL getAllUsers();
```

调用存储过程

```sql
CALL getAllUsers();
```

### 触发器

```sql
CREATE TRIGGER add_userTime 
BEFORE 
    INSERT
on `user`
for each ROW 
INSERT INTO usercreatetime(create_time) VALUES(now());
```

### 创建表

```sql
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `account` varchar(255) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  `address` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
DROP TABLE IF EXISTS `user_history`;
CREATE TABLE `user_history` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `user_id` bigint(20) NOT NULL,
  `operatetype` varchar(200) NOT NULL,
  `operatetime` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### 创建触发器

```sql
DROP TRIGGER IF EXISTS `tri_insert_user`
delimiter ;;
CREATE TRIGGER `tri_insert_user` AFTER INSERT ON `user` FOR EACH ROW 
BEGIN 
    INSERT INTO user_history(user_id, operatetype, operatetime) VALUES (new.id, 'add a user', now());
END
;;
delimiter ;
```

### 创建触发器2

```sql
DROP TRIGGER IF EXISTS `tri_update_user`;
DELIMITER ;;
CREATE TRIGGER `tri_update_user` AFTER UPDATE ON `user` FOR EACH ROW begin
    INSERT INTO user_history(user_id,operatetype, operatetime) VALUES (new.id, 'update a user', now());
end
;;
DELIMITER ;
DROP TRIGGER IF EXISTS `tri_delete_user`;
DELIMITER ;;
CREATE TRIGGER `tri_delete_user` AFTER DELETE ON `user` FOR EACH ROW begin
    INSERT INTO user_history(user_id, operatetype, operatetime) VALUES (old.id, 'delete a user', now());
end
;;
DELIMITER ;
```

### 测试触发器

```sql
INSERT INTO user(account, name, address) VALUES ('user1', 'user1', 'user1');
INSERT INTO user(account, name, address) VALUES ('user2', 'user2', 'user2');
UPDATE user SET name = 'user3', account = 'user3', address='user3' where name='user1';
```

## 开启定时事件

```sql
SHOW VARIABLES LIKE '%event_scheduler%'
CREATE TABLE aaa(timeline TIMESTAMP);
```

### 创建事件

```sql
CREATE EVENT myevent_insert 
ON SCHEDULE EVERY 1 SECOND DO INSERT aaa VALUES(CURRENT_TIMESTAMP);
SHOW EVENTS
```

### 关闭事件

```sql
ALTER EVENT myevent_insert ON COMPLETION  PRESERVE DISABLE;
```

## 视图

视图的作用：简化SQL查询，提升开发效率，兼容老的表结构，安全性需要 MySQL处理视图的两种算法：merge和temptable merge，将视图sql合并到主查询 temptable，将视图单作临时表处理 尽量选择merge算法

```sql
CREATE VIEW test_view as SELECT * from aaa;
SELECT * FROM test_view
```

## 游标

游标的作用：游标提供了一种对从表中检索出来的数据进行操作的灵活手段，通过使用游标，是SQL这种面向集合的语言有了面向过程开发的能力；

## 数据库比较

非关系型数据库：NOSQL基于键值对匹配，性能比较高 关系型数据库：可以涉及多个表的查询

## 数据库范式

第一范式：确保每列所有字段值都是不可分解的原子值 第二范式：确保每列都和主键相关 第三范式：确保每列都和逐渐直接相关，不是间接相关 第四范式：要求把同一表内的多对多关系删除 第五范式：从最终结构重新建立原始结构

## 一些定义

内连接：只连接匹配的行 左外连接：包含左边表的全部行，以及右边表的全部匹配的行 右外连接：包含右边表的全部行，以及左边表的全部匹配的行 全外连接：包含左右两个表的全部行，不管另外一边是否与之匹配 交叉连接：生成笛卡尔积，将一个数据源中的所有行与另一个数据源中的每行进行一一匹配
