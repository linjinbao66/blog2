---
title: kubernetes多用户切换
tags: ["kubernetes"]
date: 2021-06-30
---

k8s中多用户使用，主要命令在kubectl config命令下，执行kubectl config可以看到提示：

```shel
[root@node1 ansible]# kubectl config 
Modify kubeconfig files using subcommands like "kubectl config set current-context my-context"

 The loading order follows these rules:

  1.  If the --kubeconfig flag is set, then only that file is loaded. The flag may only be set once and no merging takes
place.
  2.  If $KUBECONFIG environment variable is set, then it is used as a list of paths (normal path delimiting rules for
your system). These paths are merged. When a value is modified, it is modified in the file that defines the stanza. When
a value is created, it is created in the first file that exists. If no files in the chain exist, then it creates the
last file in the list.
  3.  Otherwise, ${HOME}/.kube/config is used and no merging takes place.

Available Commands:
  current-context 显示 current_context
  delete-cluster  删除 kubeconfig 文件中指定的集群
  delete-context  删除 kubeconfig 文件中指定的 context
  delete-user     Delete the specified user from the kubeconfig
  get-clusters    显示 kubeconfig 文件中定义的集群
  get-contexts    描述一个或多个 contexts
  get-users       Display users defined in the kubeconfig
  rename-context  Renames a context from the kubeconfig file.
  set             设置 kubeconfig 文件中的一个单个值
  set-cluster     设置 kubeconfig 文件中的一个集群条目
  set-context     设置 kubeconfig 文件中的一个 context 条目
  set-credentials 设置 kubeconfig 文件中的一个用户条目
  unset           取消设置 kubeconfig 文件中的一个单个值
  use-context     设置 kubeconfig 文件中的当前上下文
  view            显示合并的 kubeconfig 配置或一个指定的 kubeconfig 文件

Usage:
  kubectl config SUBCOMMAND [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
[root@node1 ansible]# 

```

kubectl confg的子命令包含，对user和context以及cluster的操作，其实现原理为修改~/.kube/config文件内容。

案例：创建一个新的用户，限定只能操作`test`名称空间资源。

1. 创建名称空间test

   ```code
   kubectl create ns test
   ```

1. 创建用户test-user

   ```code
   kubectl create -n test serviceaccount test-user
   ```

1. 设置证书

```shell
export TOKEN_CONTENT=`kubectl describe -n test sa test-user | grep Tokens | awk -F ":" '{print$2}' | xargs kubectl describe secret -n test|grep token: | awk -F ':' '{print $2}' | tr -d ' \t'`
kubectl config set-credentials test-cred --token=${TOKEN_CONTENT}
```

1. 设置context

   ```shell
   kubectl config set-context test-context --user=test-cred --cluster=kubernetes --namespace=test
   ```

   注意--user=test-cred需要和上一步设置证书中的test-cred一致

1. 切换到context

   ```shell
   kubectl config use-context test-context
   ```

1. 授予权限

按照上述流程操作得出的context，使用时时没有权限的。

想要授予权限，可以使用rolebinding，绑定到admin角色

```shell
kubectl create rolebinding test-admin --clusterrole=admin --serviceaccount=test:test-user --namespace test 
```

注意此处的--serviceaccount=test:test-user使用的上面的serviceaccount test-user

至此，使用test-context即可访问名称空间test下的所有资源了，访问其他名称空间下的会报没权限。

附：kubeconfig签发

```shell
export KUBE_APISERVER="https://172.20.0.113:6443"
# 设置集群参数
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER}
# 设置客户端认证参数
kubectl config set-credentials admin \
  --client-certificate=/etc/kubernetes/ssl/admin.pem \
  --embed-certs=true \
  --client-key=/etc/kubernetes/ssl/admin-key.pem
# 设置上下文参数
kubectl config set-context kubernetes \
  --cluster=kubernetes \
  --user=admin
# 设置默认上下文
kubectl config use-context kubernetes
```
