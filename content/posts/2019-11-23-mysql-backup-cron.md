---
title: mysql 定时备份
tags:
  - mysql
date: 2019-11-23
---

## mysql 定时备份

第一步：写备份脚本 backup_full.sh

```text
#!/bin/bash

BACKUP_ROOT=/home/ljbao/myback
BACKUP_FILEDIR=$BACKUP_ROOT/files
BACKUP_LOGDIR=$BACKUP_ROOT/logs


#当前日期
DATE=$(date +%Y%m%d)

DATABASES=$(mysql -uroot -p***  -e "show databases" | grep -Ev "Database|sys|information_schema")

echo $DATABASES

#循环数据库进行备份
for db in $DATABASES
do
echo
echo ----------$BACKUP_FILEDIR/${db}_$DATE.sql.gz BEGIN----------
mysqldump -uroot -p***  --default-character-set=utf8 -q --lock-all-tables --flush-logs -E -R --triggers -B ${db} | gzip > $BACKUP_FILEDIR/${db}_$DATE.sql.gz
echo ----------$BACKUP_FILEDIR/${db}_$DATE.sql.gz COMPLETE----------
echo
done

echo "back complete"
```

第二步：注册定时任务

```text
[root@localhost myback]# crontab -l -uroot

## 每天3点
00 3 * * * sh /home/ljbao/myback/backup_full.sh
```

第三步：编写清理脚本 clean\_backup\_full.sh

```text
# !/bin/bash

## 清理7天前的备份
find /home/ljbao/myback/files -mtime +7 -name "*.gz" -exec rm {} \;
```

第四步：注册定时清理

```text
[root@localhost myback]# crontab -l -u root

00 3 * * * sh /home/ljbao/myback/backup_full.sh

00 1 * * * sh /home/ljbao/myback/clean_backup_full.sh
```

## 备注：MySQL全量备份

mysqldump -E -R --triggers --single-transaction --master-data=2 --default-character-set=utf8 -u root -p 库名 &gt; /tmp/库名.sql  
-R表示导出function和procedure -E表示导出事件 -x 给所有表加读锁（可选择是否读锁）会自动解锁 -master-data=2 是锁表操作 --single-transaction 不锁表备份，将备份的操作放到一个事务里进行
