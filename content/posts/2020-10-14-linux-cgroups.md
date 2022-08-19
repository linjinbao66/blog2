---
title: linux cgroups简单使用
date: 2020-10-14
---

## 简介

cgroups 是Linux内核提供的一种可以限制单个进程或者多个进程所使用资源的机制，可以对 cpu，内存等资源实现精细化的控制，目前越来越火的轻量级容器 Docker 就使用了 cgroups 提供的资源限制能力来完成cpu，内存等部分的资源控制。另外，开发者也可以使用 cgroups 提供的精细化控制能力，限制某一个或者某一组进程的资源使用。比如在一个既部署了前端 web 服务，也部署了后端计算模块的八核服务器上，可以使用 cgroups 限制 web server 仅可以使用其中的六个核，把剩下的两个核留给后端计算模块。

简单理解：cgroups可以实现cpu和内存的机器资源的精细化控制，比如限制使用10%的cpu资源，使用100M内存等功能。Docker容器运行时通过参数指定内存和cpu占用功能就是基于这个实现的。

## 主要作用

实现 cgroups 的主要目的是为不同用户层面的资源管理提供一个统一化的接口。从单个任务的资源控制到操作系统层面的虚拟化，cgroups 提供了四大功能：

- 资源限制：cgroups 可以对任务是要的资源总额进行限制。
- 比如设定任务运行时使用的内存上限，一旦超出就发 OOM。
- 优先级分配：通过分配的 CPU 时间片数量和磁盘 IO 带宽，实际上就等同于控制了任务运行的优先级。
- 资源统计：cgoups 可以统计系统的资源使用量，比如 CPU 使用时长、内存用量等。这个功能非常适合当前云端产品按使用量计费的方式。
- 任务控制：cgroups 可以对任务执行挂起、恢复等操作。

## 使用

cgroups 以文件的方式提供应用接口，我们可以通过 mount 命令来查看 cgroups 默认的挂载点：

```code
root@instance-1:~# mount | grep cgroup
tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,mode=755)
cgroup2 on /sys/fs/cgroup/unified type cgroup2 (rw,nosuid,nodev,noexec,relatime,nsdelegate)
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,name=systemd)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpu,cpuacct)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
cgroup on /sys/fs/cgroup/rdma type cgroup (rw,nosuid,nodev,noexec,relatime,rdma)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_cls,net_prio)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,pids)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
```

笔者系统的debian，cgroup默认是安装在`/sys/fs/cgroup`文件夹下，centos好像是安装在`/cgroup`下。

第一行的 tmpfs 说明 /sys/fs/cgroup 目录下的文件都是存在于内存中的临时文件。
第二行的挂载点 /sys/fs/cgroup/systemd 用于 systemd 系统对 cgroups 的支持，相关内容笔者今后会做专门的介绍。
其余的挂载点则是内核支持的各个子系统的根级层级结构。

需要注意的是，在使用 systemd 系统的操作系统中，/sys/fs/cgroup 目录都是由 systemd 在系统启动的过程中挂载的，并且挂载为只读的类型。换句话说，系统是不建议我们在 /sys/fs/cgroup 目录下创建新的目录并挂载其它子系统的。这一点与之前的操作系统不太一样。

### 查看进程所属的 cgroups

可以通过 /proc/[pid]/cgroup 来查看指定进程属于哪些 cgroup：

```shell
root@instance-1:~# ps -ef | grep nginx
root     12006 11947  0 Oct13 ?        00:00:00 nginx: master process nginx -g daemon off;
systemd+ 12045 12006  0 Oct13 ?        00:00:00 nginx: worker process
root     13643 10606  0 06:29 pts/1    00:00:00 grep nginx
root     14516 14463  0 Oct13 ?        00:00:00 nginx: master process nginx -g daemon off;
systemd+ 14553 14516  0 Oct13 ?        00:00:00 nginx: worker process
root@instance-1:~# cat /proc/14553/cgroup 
11:memory:/kubepods/besteffort/pod8bd7f8dd-1676-4eed-a5fa-aeb5b769d9f5/8b4bad1ca396b50bcb33d8b2068bcc6685f2386260d2dcfaec8327cde52afa33
10:pids:/kubepods/besteffort/pod8bd7f8dd-1676-4eed-a5fa-aeb5b769d9f5/8b4bad1ca396b50bcb33d8b2068bcc6685f2386260d2dcfaec8327cde52afa33
9:devices:/kubepods/besteffort/pod8bd7f8dd-1676-4eed-a5fa-aeb5b769d9f5/8b4bad1ca396b50bcb33d8b2068bcc6685f2386260d2dcfaec8327cde52afa33
8:net_cls,net_prio:/kubepods/besteffort/pod8bd7f8dd-1676-4eed-a5fa-aeb5b769d9f5/8b4bad1ca396b50bcb33d8b2068bcc6685f2386260d2dcfaec8327cde52afa33
7:freezer:/kubepods/besteffort/pod8bd7f8dd-1676-4eed-a5fa-aeb5b769d9f5/8b4bad1ca396b50bcb33d8b2068bcc6685f2386260d2dcfaec8327cde52afa33
6:rdma:/
5:perf_event:/kubepods/besteffort/pod8bd7f8dd-1676-4eed-a5fa-aeb5b769d9f5/8b4bad1ca396b50bcb33d8b2068bcc6685f2386260d2dcfaec8327cde52afa33
4:cpu,cpuacct:/kubepods/besteffort/pod8bd7f8dd-1676-4eed-a5fa-aeb5b769d9f5/8b4bad1ca396b50bcb33d8b2068bcc6685f2386260d2dcfaec8327cde52afa33
3:blkio:/kubepods/besteffort/pod8bd7f8dd-1676-4eed-a5fa-aeb5b769d9f5/8b4bad1ca396b50bcb33d8b2068bcc6685f2386260d2dcfaec8327cde52afa33
2:cpuset:/kubepods/besteffort/pod8bd7f8dd-1676-4eed-a5fa-aeb5b769d9f5/8b4bad1ca396b50bcb33d8b2068bcc6685f2386260d2dcfaec8327cde52afa33
1:name=systemd:/kubepods/besteffort/pod8bd7f8dd-1676-4eed-a5fa-aeb5b769d9f5/8b4bad1ca396b50bcb33d8b2068bcc6685f2386260d2dcfaec8327cde52afa33
0::/system.slice/k3s.service
```

解释：预先使用k3s启动两个nginx pod，使用ps查询出pid，根据pid查询cgroup。每一行包含用冒号隔开的三列，他们的含义分别是：

- cgroup 树的 ID， 和 /proc/cgroups 文件中的 ID 一一对应。
- 和 cgroup 树绑定的所有 subsystem，多个 subsystem 之间用逗号隔开。这里 name=systemd 表示没有和任何 subsystem 绑定，只是给他起了个名字叫 systemd。
- 进程在 cgroup 树中的路径，即进程所属的 cgroup，这个路径是相对于挂载点的相对路径。

cgroups结构：

```code
root@instance-1:~# cat /proc/cgroups 
#subsys_name	hierarchy	num_cgroups	enabled
cpuset	2	30	1
cpu	4	121	1
cpuacct	4	121	1
blkio	3	120	1
memory	11	186	1
devices	9	120	1
freezer	7	30	1
net_cls	8	30	1
perf_event	5	30	1
net_prio	8	30	1
pids	10	126	1
rdma	6	1	1
```

### 实例： 限制进程可用的cpu

```pseudocode
sudo apt install cgroup-bin ##安装cgroups工具
```

1. 新建目录

```code
cd /sys/fs/cgroup/cpu
sudo mkdir nick_cpu
```

cgroups 的文件系统会在创建文件目录的时候自动创建配置文件，如下：

```code
root@instance-1:~# ls /sys/fs/cgroup/cpu/nick_cpu/
cgroup.clone_children  cpu.cfs_quota_us  cpuacct.stat	    cpuacct.usage_percpu       cpuacct.usage_sys   tasks
cgroup.procs	       cpu.shares	 cpuacct.usage	    cpuacct.usage_percpu_sys   cpuacct.usage_user
cpu.cfs_period_us      cpu.stat		 cpuacct.usage_all  cpuacct.usage_percpu_user  notify_on_release
```

2. 写入cpu限制配置

```code
echo 100000 > nick_cpu/cpu.cfs_period_us
echo 10000 > nick_cpu/cpu.cfs_quota_us
```

以上配置现在cpu周期限制在十分之一。

3. 验证

创建一个 CPU 密集型的程序：

cputime.c

```c
#include <stdio.h>
void main()
{ 
  unsigned int i, end;
  end = 1024 * 1024 * 1024; 
  for(i = 0; i < end; )
  {
    i ++; 
  }
}
```

编译运行：

```shell
gcc cputime.c -o cputime  ##编译

root@instance-1:~# time ./cputime  ##正常运行

real	0m2.948s
user	0m2.942s
sys	0m0.000s
root@instance-1:~# time cgexec -g cpu:nick_cpu ./cputime  ## 带cgroup限制运行

real	0m29.466s
user	0m2.961s
sys	0m0.003s
```

time 命令可以为我们报告程序执行消耗的时间，其中的 real 就是我们真实感受到的时间。使用 cgexec 能够把我们添加的 cgroup 配置 nick_cpu 应用到运行 cputime 程序的进程上。 上图显示，默认的执行只需要 3s 左右。通过 cgroups 限制 CPU 资源后需要运行 29s。



## 相关命令

1. 查看所有的cgroup：`lscgroup`
2. 查看所有支持的子系统：`lssubsys -a`
3. 查看所有子系统挂载的位置：`lssubsys -m`

