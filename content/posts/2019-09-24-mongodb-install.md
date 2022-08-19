---
title: Mongodb安装与使用
tags:
  - Mongodb
  - 数据库
  - 教程
categories:
  - 教程
date: 2019-09-24
---

## Mongodb安装与使用

## 背景知识

最近在项目实施过程中遇到一个问题：由于客户提供的Linux机器没有最下滑化安装，导致使用公司的`bin`包方式安装`Mongodb`失败。本着能自己解决就自己解决的态度，我开始了Mongodb的研究。

## 安装和启动

1. 下载

   因为`centos`的官放的`yum`源中没有提供`Mongodb`的安装包，此时我们需要自己下载下来（也可以修改`yum`源，不推荐） 执行命令`wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.0.tgz` 下载安装包

2. 解压

   执行解压命令`tar zxvf mongodb-linux-x86_64-4.0.0.tgz`，如果为了统一的化，可以把解压出来的文件夹**重命名**，以及移动到`/opt/`下去。这个不重要，可以不用看。

3. 添加启动配置

   ```text
    vi mongodb.conf
   ```

   添加一下配置：

   ```text
    port=27017 #端口
    bind_ip = 0.0.0.0  #这样就可外部访问了，例如从win10中去连虚拟机中的MongoDB
    fork=true#后台启动
    logpath=/usr/mongodb/log/log.log
    dbpath=/usr/mongodb/db
   ```

   其他配置不用管他，`Mongodb`会使用默认配置。

4. 设置权限

   ```text
    cd /usr/mongodb
    chmod 777 db
    chmod 777 log
   ```

   管他呢，全部**777**

5. 启动服务

   ```text
   mongod --config mongodb.conf
   ```

6. 修改用户 使用`mongo`修改用户，注意不是`mongod`

   ```text
   [root@jk-zhengwu03 bin]# ./mongo
   MongoDB shell version v4.0.0
   connecting to: mongodb://127.0.0.1:27017
   MongoDB server version: 4.0.0
   Server has startup warnings:
   2019-09-24T20:05:18.388+0800 I CONTROL  [initandlisten]
   2019-09-24T20:05:18.388+0800 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
   2019-09-24T20:05:18.388+0800 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
   2019-09-24T20:05:18.388+0800 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not recommended.
   2019-09-24T20:05:18.388+0800 I CONTROL  [initandlisten]
   2019-09-24T20:05:18.388+0800 I CONTROL  [initandlisten]
   2019-09-24T20:05:18.388+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
   2019-09-24T20:05:18.388+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
   2019-09-24T20:05:18.389+0800 I CONTROL  [initandlisten]
   2019-09-24T20:05:18.389+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/defrag is 'always'.
   2019-09-24T20:05:18.389+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
   2019-09-24T20:05:18.389+0800 I CONTROL  [initandlisten]
   > use admin
   switched to db admin
   ```

   创建用户的命令为：`db.createUser({user:"admin",pwd:"123456",roles:["root"]})`，返回值是`1`则创建成功.

## 修改系统配置文件

修改`epointframe.properties`

```text
#
MongodbUrl=jdbc:mongodb://192.168.202.161/epointlog
MongodbUserName=admin
MongodbPassword=111111
```

修改成

```text
#
MongodbUrl=jdbc:mongodb://X.X.X.X/X
MongodbUserName=admin
MongodbPassword=123456
```

此处注意,url后面不需要加端口号`27017`,虽然它的端口号确实是这个.

## 总结

`Mongodb`的教程安装教程结束.`Mongodb`是一个什么样的数据库呢,给我的感觉就是\[key,value\]的样子,他不像传统的`关系型`数据库那样,翻来覆去都是表格约束,有横有纵,`mongodb`,更像是一个一对一的关系,例如**一夫一妻**制度,所以可以唯一确定.我感觉这万一使用简单,带来的就是空间的极度浪费,有时间我再去研究研究. 喜欢我的博客欢迎转载[菜鸡聪](https://github.com/linjinbao66/blog/tree/a3d1413b612e726e60070174f80d386f3177521a/linjinbao666.github.io).
