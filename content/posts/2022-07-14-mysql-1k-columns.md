---
title: mysql生成宽表脚本 
date: 2022-07-14
---

## 需求描述

生成一张有1000个字段的表，每个字段的类型随机，用测试使用。

## 解决方案

### shell脚本生成建表sql

generate_sql.sh

```shell
#!/bin/bash 
echo "USE amrom;"
echo "CREATE TABLE ONE_K_COLS (" 
for (( i = 0; i < 999; i++ )) {
  col_type=`echo "tinyint int(11) bigint(20) varchar(50) char(30) date  $RANDOM" | awk '{print $($NF%(NF-1)+1)}'`
  suffix=`cat /proc/sys/kernel/random/uuid | cut -f5 -d"-"`
  echo "col"_$suffix' '${col_type}','
} 
echo "id int auto_increment," 
echo "primary key (id)" 
echo ") engine=innodb;" 
 
exit 0
```

- 使用方式：./generate_sql.sh > 1ktab.sql

### 修改mysql配置

```shell
[mysqld]

innodb_log_file_size = 512M

innodb_strict_mode = 0
```

## 注意

- 随机数的生成，直接使用$RANDOM会重复

- mysql默认限制了行级存储大小，会报错`MySQL: Error Code: 1118 Row size too large (> 8126). Changing some columns to TEXT or BLOB`，需要禁用严格模式解决

![生成的sql](/images/mysql-1k-cols.png)
