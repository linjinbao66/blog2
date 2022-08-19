---
title: kubernetes二进制部署升级
tags: ["kubernetes"]
date: 2022-06-07
---

## 前提

k8s的部署方式基本上有2中，其一是全部利用kubeadm部署，这种方式会以static pod形式部署各个组件，升级的时候按照kubeadm的手册就行，需要先使用`kubeadm upgrade plan`获得提示，第二种部署方式是组件全部以二进制形式部署，这种部署的升级的时候需要手动操作，以下记录了一次升级操作的过程。

| 组件                    | 升级前 | 升级后 |
| ----------------------- | ------ | ------ |
| Etcd                    | 3.4.15 | 3.4.15 |
| kube-apiserver          | 1.20.3 | 1.21.7 |
| kube-scheduler          | 1.20.3 | 1.21.7 |
| kube-controller-manager | 1.20.3 | 1.21.7 |
| kube-proxy              | 1.20.3 | 1.21.7 |
| kubelet                 | 1.20.3 | 1.21.7 |
| kubectl                 | 1.20.3 | 1.21.7 |

## 第一步 下载二进制文件

```shell
wget https://storage.googleapis.com/kubernetes-release/release/v1.21.7/kubernetes-server-linux-amd64.tar.gz 
tar -xf kubernetes-server-linux-amd64.tar.gz
```

## 第二步 移除节点

```shell
kubectl uncordon node2 
```

## 第三步 停止服务

```shell
systemctl stop kubelet kube-apiserver kube-scheduler kube-proxy  kube-controller-manager
```

## 第四步 替换二进制文件

```shell
scp copy/* root@node2:/opt/k8s/bin/
```

## 第五步 启动服务

```shell
systemctl start kubelet kube-apiserver kube-scheduler kube-proxy  kube-controller-manager
```

重复以上步骤，直到所有节点的文件替换完成即可
