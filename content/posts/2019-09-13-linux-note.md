---
title: Linux笔记
tags:
  - 笔记
  - 后端开发
  - Linux
  - shell
categories:
  - Linux
date: 2019-09-13
---

## 笔记

## 查找软件包rsync

```text
rpm -qa | grep rsync
```

## 客户端拉取

```text
 rsync -av root@192.168.59.130::backup /home/ljbao/
```

## 打印指定行数据

```text
sed -n '10,10p' file.txt
cat file.txt | head -n 10 | tail -n +10
```

## awk使用

```text
awk '{print $2} {print $1}' emp.data
```

## crond使用

```text
    service crond start    //启动服务
　　    service crond stop     //关闭服务
　    service crond restart  //重启服务
　　    service crond reload   //重新载入配置
　　    service crond status   //查看服务状态
```

第一步：写`cron`脚本文件,命名为`crontest.cron`。 `15,30,45,59 * * * * echo "xgmtest....." >> xgmtest.txt` 表示，每隔15分钟，执行打印一次命令 第二步：添加定时任务。执行命令 `crontab crontest.cron`。搞定 第三步：`crontab -l`查看定时任务是否成功或者检测`/var/spool/cron`下是否生成对应`cron`脚本

脚本实例：

```text
0 6 * * * echo "Good morning." >> /tmp/test.txt //注意单纯echo，从屏幕上看不到任何输出，因为cron把任何输出都email到root的信箱了。
0 */2 * * * echo "Have a break now." >> /tmp/test.txt  
0 23-7/2，8 * * * echo "Have a good dream" >> /tmp/test.txt
0 11 4 * 1-3 command line
0 4 1 1 * command line SHELL=/bin/bash PATH=/sbin:/bin:/usr/sbin:/usr/bin MAILTO=root //如果出现错误，或者有数据输出，数据作为邮件发给这个帐号 HOME=/ 
01 * * * * root run-parts /etc/cron.hourly
02 4 * * * root run-parts /etc/cron.daily 
22 4 * * 0 root run-parts /etc/cron.weekly 
42 4 1 * * root run-parts /etc/cron.monthly 
5，15，25，35，45，55 16，17，18 * * * command
00 15 * * 1，3，5 shutdown -r +5
10，40 * * * * innd/bbslink
```

第 1 列分钟 1～59

第 2 列小时 1～23（0 表示子夜）

第 3 列日 1～31

第 4 列月 1～12

第 5 列星期 0～6（0 表示星期天）

第 6 列要运行的命令

分 时 日 月 星期 要运行的命令

`30 21 * * * /usr/local/apache/bin/apachectl restart`

上面的例子表示每晚的 21:30 重启 apache。

## MySQL定时备份

```text
vim DatabaseName.sh
/usr/local/mysql/bin/mysqldump -uusername -ppassword DatabaseName > /home/backup/DatabaseName_$(date +%Y%m%d_%H%M%S).sql
/usr/local/mysql/bin/mysqldump -uusername -ppassword DatabaseName | gzip > /home/backup/DatabaseName_$(date +%Y%m%d_%H%M%S).sql.gz
```

**注意**：

把`username` 替换为实际的用户名； 把 `password`替换为实际的密码； 把 `DatabaseName` 替换为实际的数据库名；

```text
chmod u+x DatabaseName.sh
crontab -e
01   3 * * * root/home/backup/DatabaseName.sh
```
