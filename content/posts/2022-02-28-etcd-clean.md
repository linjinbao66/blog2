---
title: etcd强制删除k8s数据
tags: ["etcd","kubernetes"]
date: 2022-02-28
---

k8s由于各种原因，导致pod等数据一直处于terminting状态中，可以考虑到etcd中强制删除

```code
#检索pod数据
etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/ca.crt --key=/etc/kubernetes/pki/etcd/ca.key get /registry/pod --prefix --keys-only
```

```shell
#根据关键字清楚
export ssss=calico
etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/ca.crt --key=/etc/kubernetes/pki/etcd/ca.key get /registry --prefix --keys-only | grep ${ssss} | xargs -I{} etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/ca.crt --key=/etc/kubernetes/pki/etcd/ca.key del {}
```
