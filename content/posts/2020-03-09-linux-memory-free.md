---

title: 服务器内存优化
date: 2020-03-09

---

## 服务器内存优化

1. 释放缓存

```text
sync
echo 3 > /proc/sys/vm/drop_caches
```

1. 排序

```text
ps -eo pmem,pcpu,rss,vsize,args | sort -k 1 -r | less
```

1. 查看启动的服务

```text
systemctl list-unit-files --type=service --state=enabled
```
