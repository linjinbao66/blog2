---
title: MySQL一键备份脚本
tags:
  - MYSQL
  - bat
categories:
  - MySQL
date: 2019-09-24
---

## MySQL一键备份脚本

## 备份脚本-Windows版本

[脚本下载](https://raw.githubusercontent.com/linjinbao666/resources/master/bat%E8%84%9A%E6%9C%AC/backup.bat)

```text
@echo off

pause
set year=%Date:~0,4%
set month=%Date:~5,2%
set day=%Date:~8,2%

set address=f:\
set host=localhost
set user=root
set password=123321
set mysqlpath=D:\epoint_mysql_5.7.24\mysql-5.7.24-winx64\bin\mysqldump.exe
set port=3306

rem 用户输入：
set /p  address=备份地址(例如：F:\)
set /p mysqlpath=请输入MySQL安装，不输入则使用默认地址D:\epoint_mysql_5.7.24\mysql-5.7.24-winx64\bin\mysqldump.exe
set /p host=请输入主机地址（默认localhost）
set /p  user=请输入数据库用户名：（默认root）
set /p password=请输入数据库密码：(默认123321)
set /p port=请输入端口号（默认3306）
set /p dbname=请输入数据库名称
rem 用户输入结束：

echo 用户输入结束，备份地址为%address%； 数据库用户名为：%user%；数据库密码为：%password%; 
pause

set backupfile=%address%%year%-%month%-%day%-%dbname%.sql
echo 备份文件: %backupfile%

pause

%mysqlpath%  -u%user%  -p%password%  %dbname% > %backupfile%

echo 请检查数据库备份是否成功，备份文件为%backupfile%
echo 欢迎下次使用
pause
exit
```

## 备份脚本-Linux版本

待续
