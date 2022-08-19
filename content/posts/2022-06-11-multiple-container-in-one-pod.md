---
title: multiple-container-in-one-pod
date: 2022-06-11
tags: ["kubernetes"]
---

在kubernetes中，默认场景下，initContainers中的容器是按照顺序启动的，且存在先后依赖关系，即前一个启动完了，才会启动后一个，containers中则是按照顺序启动，但是不存在依赖关系，这就给一些使用场景带来了麻烦。例如，在使用pod作为流水线的时候，containers中的容器顺序需要存在先后依赖关系。以下记录一些实践方案：

## 方案一

利用原生的`postStart`机制，该机制保证了后一个容器在前一个容器发出该信号前不会创建。

实例：

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: my-pipline
  name: my-pipline
spec:
  volumes:
  - name: data
    emptyDir: {}
  initContainers:
  - image: "alpine/git"
    name: prepare
    env:
    - name: repo
      value: "https://github.com/linjinbao666/vrmanager.git"
    args:
    - clone
    - --single-branch
    - --branch=main
    - --
    - "https://github.com/linjinbao666/vrmanager.git"
    - "/opt/workspace"
    volumeMounts:
    - name: data
      mountPath: "/opt/workspace" 
  containers:
  - image: "docker.io/linjinbao66/dt-maven:0.0.2"
    name: my-pipline
    workingDir: "/opt/workspace"
    env:
    - name: repo
      value: "https://github.com/linjinbao666/vrmanager.git"
    command:
    - "/bin/sh"
    - "-c"
    args: 
    - "mvn clean package; /bin/sidecar"
    lifecycle:
      postStart:
        exec:
          command:
          - "/bin/entrypoint"
          - "wait"
    volumeMounts:
    - name: data
      mountPath: "/opt/workspace"
  - image: "busybox:latest"
    name: busybox
    command:
    - "/bin/sh"
    - "-c" 
    args: ["date; echo 'app container started'"]
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

实例中的2个容器，my-pipline和busybox，busybox一定会在my-pipline的`entrypoint wait`信号给出返回后，才会创建。

该方案的技术要点：

* 需要重写前一容器的启动脚本，使得该容器启动时，执行完主程序命令后，启动sidecar一个端口，（或者监听文件），该端口给entrypoint探测用，entryoint是一只在探测，只有得到想要的结果后，才会放行。
* 该方案需要前容器会保持Running状态，由于sidecar程序监听，所以整个Pod不会出现Completed状态，如果后一容器会执行完则退出，则会出现NotReady状态。
* 该方案在前一容器sidecar程序执行前的日志无法实时获取，如果该阶段比较耗时，例如是mvn build命令，则不利于日志的实时推出

## 方案二

该方案利用多容器间共享存储卷实现

实例：

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: my-pipline
  name: my-pipline
spec:
  volumes:
  - name: data
    emptyDir: {}
  initContainers:
  - image: "alpine/git"
    name: prepare
    env:
    - name: repo
      value: "https://github.com/linjinbao666/vrmanager.git"
    args:
    - clone
    - --single-branch
    - --branch=main
    - --
    - "https://github.com/linjinbao666/vrmanager.git"
    - "/opt/workspace"
    volumeMounts:
    - name: data
      mountPath: "/opt/workspace" 
  containers:
  - image: "docker.io/linjinbao66/dt-maven:0.0.3"
    name: my-pipline
    workingDir: "/opt/workspace"
    env:
    - name: repo
      value: "https://github.com/linjinbao666/vrmanager.git"
    command:
    - "/bin/sh"
    - "-c"
    args: 
    - "mvn clean  package && echo 'build finished' > /opt/workspace/mvn.log"
    volumeMounts:
    - name: data
      mountPath: "/opt/workspace"
  - image: "busybox:latest"
    name: busybox
    command:
    - "/bin/sh"
    - "-c" 
    args: ["while true; do sleep 10 && [ -f '/opt/workspace/mvn.log' ] && exit 0|| echo 'file not exists' ;  done"]
    volumeMounts:
    - name: data
      mountPath: "/opt/workspace"
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

容器my-pipline将执行完的结果写入共享文件`/opt/workspace/mvn.log`，容器busybox检测共享文件是否存在，存在则执行自身命令，执行完退出。

该方案可以同时查看2个容器的日志。

参考链接：

* https://medium.com/@marko.luksa/delaying-application-start-until-sidecar-is-ready-2ec2d21a7b74
* https://atbug.com/k8s-1.18-container-start-sequence-control/
* https://atbug.com/control-process-order-of-pod-containers/