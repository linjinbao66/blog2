---
title: linux network namespace 详解
date: 2022-04-26
---

# 网络名称空间

linux 网络空间用于管理容器的网络，编排工具创建容器的时候往往会为容器创建一个独立的namespace，然后将容器的网络与宿主机的网络打通，即可实现容器与外部网络的通信。

linux namespace的操作命令是`ip netns`

## 使用实例

1. 创建一个namespace

   ```shell
   [root@localhost ~]# ip netns add net1 
   [root@localhost ~]# ip netns ls 
   net1
   net0
   [root@localhost ~]# ip netns exec net1 ip addr 
   1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
   ```

2. 不同namespace通信

   不同的namespace通信有2种方案，一种是使用`veth pari`通信，另一种是使用`bridge`网桥通信。

   方案一：

   ```shell
   [root@localhost ~]# ip link add type veth 
   [root@localhost ~]# ip link 
   ## 省略
   36: veth0@veth1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
       link/ether 9a:9b:f7:b2:ca:62 brd ff:ff:ff:ff:ff:ff
   37: veth1@veth0: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
       link/ether 9e:8c:83:42:72:a7 brd ff:ff:ff:ff:ff:ff
   ## veth0添加到net0名称空间, veth1添加到net1名称空间
   [root@localhost ~]# ip link set veth0 netns net0
   [root@localhost ~]# ip link set veth1 netns net1
   ## 检查各名称空间信息，此时默认的名称空间veth0和veth1都消失
   [root@localhost ~]# ip netns exec net0 ip a
   1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
   36: veth0@if37: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
       link/ether 9a:9b:f7:b2:ca:62 brd ff:ff:ff:ff:ff:ff link-netnsid 1
   [root@localhost ~]# ip netns exec net1 ip a 
   1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
   37: veth1@if36: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
       link/ether 9e:8c:83:42:72:a7 brd ff:ff:ff:ff:ff:ff link-netnsid 0
   ## 启动net0中的veth0设备
   [root@localhost ~]# ip netns exec net0 ip link set veth0 up 
   [root@localhost ~]# ip netns exec net0 ip a 
   1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
   36: veth0@if37: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state LOWERLAYERDOWN group default qlen 1000
       link/ether 9a:9b:f7:b2:ca:62 brd ff:ff:ff:ff:ff:ff link-netnsid 1
   ## 添加ip
   [root@localhost ~]# ip netns exec net0 ip addr add 10.1.1.1/24 dev veth0 
   [root@localhost ~]# ip netns exec net0 ip a 
   1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
   36: veth0@if37: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state LOWERLAYERDOWN group default qlen 1000
       link/ether 9a:9b:f7:b2:ca:62 brd ff:ff:ff:ff:ff:ff link-netnsid 1
       inet 10.1.1.1/24 scope global veth0
          valid_lft forever preferred_lft forever
   [root@localhost ~]# ip netns exec net0 ip route 
   10.1.1.0/24 dev veth0 proto kernel scope link src 10.1.1.1 
   [root@localhost ~]# ip netns exec net1 ip link set veth1 up 
   [root@localhost ~]# ip netns exec net1 ip a 
   1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
   37: veth1@if36: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
       link/ether 9e:8c:83:42:72:a7 brd ff:ff:ff:ff:ff:ff link-netnsid 0
       inet6 fe80::9c8c:83ff:fe42:72a7/64 scope link 
          valid_lft forever preferred_lft forever
   [root@localhost ~]# ip netns exec net1 ip addr add 10.1.1.2/24 dev veth1 
   [root@localhost ~]# ip netns exec net1 ip a 
   1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
   37: veth1@if36: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
       link/ether 9e:8c:83:42:72:a7 brd ff:ff:ff:ff:ff:ff link-netnsid 0
       inet 10.1.1.2/24 scope global veth1
          valid_lft forever preferred_lft forever
       inet6 fe80::9c8c:83ff:fe42:72a7/64 scope link 
          valid_lft forever preferred_lft forever
   [root@localhost ~]# ip netns exec net1 ip route 
   10.1.1.0/24 dev veth1 proto kernel scope link src 10.1.1.2
   ## 测试连通性
   [root@localhost ~]# ip netns exec net0 ping 10.1.1.2
   PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
   64 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=0.049 ms
   64 bytes from 10.1.1.2: icmp_seq=2 ttl=64 time=0.039 ms
   64 bytes from 10.1.1.2: icmp_seq=3 ttl=64 time=0.056 ms
   ^C
   --- 10.1.1.2 ping statistics ---
   3 packets transmitted, 3 received, 0% packet loss, time 1999ms
   rtt min/avg/max/mdev = 0.039/0.048/0.056/0.007 ms
   ```

   方案二：

   创建bridge网桥

   ```shell
   [root@localhost ~]# ip link add br0 type bridge 
   [root@localhost ~]# ip link 
   ## 省略
   26: veth99caeb5a@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP mode DEFAULT group default 
       link/ether 7e:cb:e2:57:bf:83 brd ff:ff:ff:ff:ff:ff link-netnsid 5
   38: br0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
       link/ether 3a:80:ef:e3:44:95 brd ff:ff:ff:ff:ff:ff
   ```

   创建3对`veth pari`

   ```code
   38: br0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
       link/ether 3a:80:ef:e3:44:95 brd ff:ff:ff:ff:ff:ff
   39: veth0@veth1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
       link/ether a6:3f:78:43:bd:2c brd ff:ff:ff:ff:ff:ff
   40: veth1@veth0: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
       link/ether 46:2f:22:c8:b8:4d brd ff:ff:ff:ff:ff:ff
   41: veth2@veth3: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
       link/ether be:5b:27:50:27:b4 brd ff:ff:ff:ff:ff:ff
   42: veth3@veth2: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
       link/ether 1e:2c:68:2f:fa:e9 brd ff:ff:ff:ff:ff:ff
   43: veth4@veth5: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
       link/ether 9a:b2:c6:b9:e7:1c brd ff:ff:ff:ff:ff:ff
   44: veth5@veth4: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
       link/ether 96:10:ed:89:f1:92 brd ff:ff:ff:ff:ff:ff
   ```

   将 `veth pair` 的一头挂到`namespace`中，一头挂到`bridge`上，并设 IP 地址。

   ```shell
   ip link set dev veth0 netns net0 ##veth0移到net0名称空间
   ip netns exec net0 ip link set dev veth0 
   ip netns exec net0 ip addr add 10.0.1.1/24 dev veth0 ##添加网卡
   ip netns exec net0 ip link set dev veth0 up 
   ip link set dev veth1 master br0 ##veth1挂载到br0
   ip link set dev veth1 master br0
   ```

   其他`veth pari`类似配置，一个放到`namespace`，一个挂到br0网桥上。

   完成之后还需要做以下操作（二选一）：

   1. 关闭系统bridge的iptables功能，这样数据包转发就不受iptables影响了：`echo 0 > /proc/sys/net/bridge/bridge-nf-call-iptables`
   2. 为br0添加一条iptables规则，让经过br0的包能被forward：`iptables -A FORWARD -i br0 -j ACCEPT`

   操作后，即可实现多namespace的网卡通信了。