---
title: Vmware虚拟机Linux修改磁盘大小
date: 2019-10-18
---

## Linux修改磁盘大小

注明： 1. 教程是在Vmware虚拟机环境下制作 2. 真机和这个差不多，只是第一步不一样

## 第一步 在Vmware中进行分配磁盘大小

如图所示： ![&#x793A;&#x610F;&#x56FE;](https://raw.githubusercontent.com/linjinbao666/resources/master/amromPicture/Vmware%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%89%A9%E5%AE%B9.png)

## 第二步：fdisk命令

```text
fdisk /dev/sda
p　　　　　　　查看已分区数量（我看到有两个 /dev/sda1 /dev/sda2）
n　　　　　　　新增加一个分区
p　　　　　　　分区类型我们选择为主分区
　　　　　　     分区号输入3（因为1,2已经用过了,sda1是分区1,sda2是分区2,sda3分区3）
回车　　　　　  默认（起始扇区）
回车　　　　　  默认（结束扇区）
t　　　　　　　 修改分区类型
　　　　　　     选分区3
8e　　　　　 　修改为LVM（8e就是LVM）
w　　　　　  　写分区表
q　　　　　  　完成，退出fdisk命令
```

## 第三步：添加新LVM到已有的LVM组，实现扩容

```text
lvm　　　　　　　　　　　　           进入lvm管理

lvm>pvcreate /dev/sda3　　           这是初始化刚才的分区3

lvm>vgextend centos /dev/sda3     将初始化过的分区加入到虚拟卷组centos (卷和卷组的命令可以通过 vgdisplay ) 

vm>vgdisplay 

lvm>lvextend -l+6793 /dev/mapper/centos-root　　扩展已有卷的容量（6793 是通过vgdisplay查看free PE /Site的大小）

lvm>pvdisplay 查看卷容量，这时你会看到一个很大的卷了

lvm>quit 　退出
```

## 第四步：文件系统的扩容

```text
xfs_growfs /dev/mapper/centos-root
```

## 查看结果-扩容了30G

```text
[root@192-168-200-130 ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   47G   17G   31G  36% /
devtmpfs                 475M     0  475M   0% /dev
tmpfs                    487M     0  487M   0% /dev/shm
tmpfs                    487M  7.7M  479M   2% /run
tmpfs                    487M     0  487M   0% /sys/fs/cgroup
/dev/sda1               1014M  133M  882M  14% /boot
tmpfs                     98M     0   98M   0% /run/user/0
[root@192-168-200-130 ~]#
```
