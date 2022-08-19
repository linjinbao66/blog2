---

title: containerd手动导入镜像
date: 2020-11-16

---

众所周知，k8s.gcr.io 长期被墙，导致 k8s 的基础容器 pause 经常无法获取。k8s docker 可使用代理服拉取，再利用 docker tag 解决问题

```code
docker pull mirrorgooglecontainers/pause:3.1 
docker tag mirrorgooglecontainers/pause:3.1 k8s.gcr.io/pause:3.1
```

我在 k8s 集群中尝试使用 containerd 作为 CRI，发现镜像下载和导入与 docker 存在一些区别，大致如下：

- containerd 命令行工具 **ctr** 特性不如 docker 丰富，如 ctr 1.2 并没有 tag 子命令，直到 1.3 才有
- 为支持多租户隔离，containerd 有 namespace 概念，不同 namespace 下的 image、container 均不同，直接使用 ctr 操作时，会使用 default namespace

如果使用的是 ctr 1.2，可以通过 docker tag 镜像，再使用 ctr 导入镜像

```code
docker save k8s.gcr.io/pause -o pause.tar 
ctr -n <namespace> images import pause.tar 
```

刚开始导入时，没有指定 namespace，pause 导入在 default 空间，整晚上创建 Pod 均处于如下状态，心态一度爆炸

```code
Warning  FailedCreatePodSandBox  9s         
kubelet, worker-2  Failed to create pod sandbox: rpc error:  
code = Unknown desc = failed to get sandbox image "k8s.gcr.io/pause:3.1": 
failed to pull image "k8s.gcr. io/pause:3.1": failed to pull and unpack image "k8s.gcr.io/pause:3.1": 
failed to resolve reference "k8s. gcr.io/pause:3.1": failed to do request: 
Head https://k8s.gcr.io/v2/pause/manifests/3.1: dial tcp 108. 177.97.82:443: i/o timeout 
```

仔细看文档发现有 namespace 这回事时，才恍然大悟 k8s 只会使用 k8s.io namespace 中镜像。于是再往 k8s.io 导入镜像，containerd worker 终于能正常被调度了

```code
ctr namespace ls NAME    LABELS default k8s.io
ctr -n k8s.io images import pause.tar
```
