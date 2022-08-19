---
title: 冒号端口
date: 2021-08-05
---

Linux程序监听网卡端口的时候会有各种缩写，其中有一种叫做冒号端口的写法。

## 现象

首先，我们看一段netstat的输出：

```shell
root@VM-0-4-debian:~# netstat -tunlp 
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:25672           0.0.0.0:*               LISTEN      639/beam.smp        
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      895/mysqld          
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      16717/redis-server  
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      581/etcd            
tcp        0      0 127.0.0.1:2380          0.0.0.0:*               LISTEN      581/etcd            
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      737/nginx: master p 
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      627/sshd            
tcp        0      0 127.0.0.1:5432          0.0.0.0:*               LISTEN      766/postgres        
tcp        0      0 127.0.0.1:8088          0.0.0.0:*               LISTEN      604/influxd         
tcp6       0      0 :::45473                :::*                    LISTEN      504/java            
tcp6       0      0 :::9090                 :::*                    LISTEN      601/prometheus      
tcp6       0      0 :::2181                 :::*                    LISTEN      504/java            
tcp6       0      0 :::5672                 :::*                    LISTEN      639/beam.smp        
tcp6       0      0 ::1:6379                :::*                    LISTEN      16717/redis-server  
tcp6       0      0 :::9100                 :::*                    LISTEN      595/prometheus-node 
tcp6       0      0 :::4369                 :::*                    LISTEN      1/init              
tcp6       0      0 :::8086                 :::*                    LISTEN      604/influxd         
tcp6       0      0 :::22                   :::*                    LISTEN      627/sshd            
tcp6       0      0 ::1:5432                :::*                    LISTEN      766/postgres        
udp        0      0 0.0.0.0:68              0.0.0.0:*                           533/dhclient        
udp        0      0 0.0.0.0:51591           0.0.0.0:*                           639/beam.smp 
```

这段输出中，`Local Address`一栏，出现了各种表示法，`0.0.0.0:25672`，`:::45473`，`::1:5432`，这几种写法，其中0.0.0.0这种写法很明显是监听所有网卡，后面两种则叫做冒号端口。

## 解释

冒号端口是ipv6才有的表示法

`:::`表示ipv6的所有网卡，类比ipv4的`0.0.0.0`

`::1`表示ipv6的本地回环网卡，类比ipv4中的`127.0.0.1`
