---
title: k8s in action 阅读笔记
tags: ["kubernetes", "k8s", "cloud native"]
date: 2021-01-28
---

## 如何保证多次请求命中同一个pod？

> If you execute the same command a few more times, you should hit a different pod with every invocation, because the service proxy normally forwards each connection to a randomly selected backing pod, even if the connections are coming from the same client. If, on the other hand, you want all requests made by a certain client to be redirected to the same pod every time, you can set the service’s sessionAffinity property to ClientIP (instead of None, which is the default), as shown in the following listing

答案：

使用`sessionAffinity`

示例：

```yaml
apiVersion: v1
kind: Service
spec:
 sessionAffinity: ClientIP
 ...
```

sessionAffinity 有两个可选值， none和ClientIP，clientIP可以控制请求到达同一个pod。

> This makes the service proxy redirect all requests originating from the same client IP to the same pod. As an exercise, you can create an additional service with session affinity set to ClientIP and try sending requests to it. Kubernetes supports only two types of service session affinity: None and ClientIP. You may be surprised it doesn’t have a cookie-based session affinity option, but you need to understand that Kubernetes services don’t operate at the HTTP level. Services deal with TCP and UDP packets and don’t care about the payload they carry. Because cookies are a construct of the HTTP protocol, services don’t know about them, which explains why session affinity cannot be based on cookies.

## curl请求与浏览器请求区别

> Session affinity and web browsers Because your service is now exposed externally, you may try accessing it with your web browser. You’ll see something that may strike you as odd—the browser will hit the exact same pod every time. Did the service’s session affinity change in the meantime? With kubectl explain, you can double-check that the service’s session affinity is still set to None, so why don’t different browser requests hit different pods, as is the case when using curl? Let me explain what’s happening. The browser is using keep-alive connections and sends all its requests through a single connection, whereas curl opens a new connection every time. Services work at the connection level, so when a connection to a service is first opened, a random pod is selected and then all network packets belonging to that connection are all sent to that single pod. Even if session affinity is set to None, users will always hit the same pod (until the connection is closed).

解释：curl请求不会带有keep-alive头参数，所以无法保持连续的请求命中同一个pod。但是一般的浏览器可以通过带有这个参数，保证多次请求命中统一pod，nginx中也有此项配置。

## 外部请求NodePort或者loadbalancer服务如何保证流量分配在本机？

详细描述：例如通过NodePort=30089暴露服务，该服务后端有3个pod，分布在3个节点，此时默认场景通过任意${IP}:${NodePort}都可以访问该服务，具体由那个节点的pod提供服务，由kube-proxy公平分配。此时如果想要指定响应的pod为${IP}节点的，该如何才做？

> UNDERSTANDING AND PREVENTING UNNECESSARY NETWORK HOPS When an external client connects to a service through the node port (this also includes cases when it goes through the load balancer first), the randomly chosen pod may or may not be running on the same node that received the connection. An additional network hop is required to reach the pod, but this may not always be desirable. You can prevent this additional hop by configuring the service to redirect external traffic only to pods running on the node that received the connection. This is done by setting the externalTrafficPolicy field in the service’s spec section:
>
> spec: externalTrafficPolicy: Local ...
>
> If a service definition includes this setting and an external connection is opened through the service’s node port, the service proxy will choose a locally running pod. If no local pods exist, the connection will hang (it won’t be forwarded to a random global pod, the way connections are when not using the annotation). You therefore need to ensure the load balancer forwards connections only to nodes that have at least one such pod.

答案： 使用externalTrafficPolicy，该属性允许2个值：Cluster和Local，前者默认，表示kube-proxy公平分发流量，后者流量只发给本机的Pod。

**需要特别注意的是以下场景：3节点，启动一个pod，设置externalTrafficPolicy为Local，则虽然3个节点都占用了${NodePort}，但是只有真实运行了pod的节点在可以访问，其他的节点并不能访问。**

## Liveness probes区别 readiness probes

前者用于探针，即定时检测服务是否可用，已经重启；后者用于判断服务是否启东完成允许访问。

> Unlike liveness probes, if a container fails the readiness check, it won’t be killed or restarted. This is an important distinction between liveness and readiness probes. Liveness probes keep pods healthy by killing off unhealthy containers and replacing them with new, healthy ones, whereas readiness probes make sure that only pods that are ready to serve requests receive them. This is mostly necessary during container start up, but it’s also useful after the container has been running for a while.

## 无头服务headless

> 有时不需要或不想要负载均衡，以及单独的 Service IP。 遇到这种情况，可以通过指定 Cluster IP（`spec.clusterIP`）的值为 `"None"` 来创建 `Headless` Service。
>
> 你可以使用无头 Service 与其他服务发现机制进行接口，而不必与 Kubernetes 的实现捆绑在一起。
>
> 对这无头 Service 并不会分配 Cluster IP，kube-proxy 不会处理它们， 而且平台也不会为它们进行负载均衡和路由。 DNS 如何实现自动配置，依赖于 Service 是否定义了选择算符。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia-headless
spec:
  clusterIP: None
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: kubia
```

> Setting the clusterIP field in a service spec to None makes the service headless, as Kubernetes won’t assign it a cluster IP through which clients could connect to the pods backing it. You’ll create a h
>
> After you create the service with kubectl create, you can inspect it with kubectl get and kubectl describe. You’ll see it has no cluster IP and its endpoints include (part of)
>
> the pods matching its pod selector. I say “part of” because your pods contain a readiness probe, so only pods that are ready will be listed as endpoints of the service.

## emptyDir使用内存直接读写

> The emptyDir you used as the volume was created on the actual disk of the worker node hosting your pod, so its performance depends on the type of the node’s disks. But you can tell Kubernetes to create the emptyDir on a tmpfs filesystem (in memory instead of on disk). To do this, set the emptyDir’s medium to Memory like this:
>
> ``` code
> volumes:
>   - name: html
>   emptyDir:
>     medium: Memory
> ```
>
> An emptyDir volume is the simplest type of volume, but other types build upon it. After the empty directory is created, they populate it with data. One such volume type is the gitRepo volume type, which we’ll introduce next.

当挂载存储为emptyDir的时候，可以设置 medium: Memory达到直接使用内存读写的效果。

## 存储挂载git repository

git repository可以由一种取巧用法，由于每次创建pod都会自动从git拉取最新的提交，所以可以将git repository整合到CI，CD中。例如可以交由deployment托管pod，pod中执行镜像构建，镜像推送等操作，，，，

> For example, you can use a Git repository to store static HTML files of your website and create a pod containing a web server container and a gitRepo volume. Every time the pod is created, it pulls the latest version of your website and starts serving it. The only drawback to this is that you need to delete the pod every time you push changes to the gitRepo and want to start serving the new version of the website. Let’s do this right now. It’s not that different from what you did before

## sidecar

用于附加到容器上，做一些附加操作，例如istio中使用的envoy

## storageclass

storageclass用于动态创建存储卷，即充当集群管理员的角色，一旦检测到pvc，即按照申请创建pv。

storageclass-fast-gcepd.yaml

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
 name: fast
provisioner: kubernetes.io/gce-pd
parameters:
 type: pd-ssd
 zone: europe-west1-b
```

provisioner: kubernetes.io/gce-pd 需要重点关注。

> The StorageClass resource specifies which provisioner should be used for provisioning the PersistentVolume when a PersistentVolumeClaim requests this StorageClass. The parameters defined in the StorageClass definition are passed to the provisioner and are specific to each provisioner plugin.
>
> The StorageClass uses the Google Compute Engine (GCE) Persistent Disk (PD) provisioner, which means it can be used when Kubernetes is running in GCE. For other cloud providers, other provisioners need to be used.

## 存在动态存储卷的情况下强制pvc绑定手动创建的pv

> FORCING A PERSISTENTVOLUMECLAIM TO BE BOUND TO ONE OF THE PRE-PROVISIONED PERSISTENTVOLUMES This finally brings us to why you set storageClassName to an empty string in listing 6.11 (when you wanted the PVC to bind to the PV you’d provisioned manually). Let me repeat the relevant lines of that PVC definition here:

```yaml
kind: PersistentVolumeClaim
spec:
 storageClassName: ""
```

答案：在pvc中设置storageClassName为""，如果该项不设置则会寻找默认的sc。

## k8s存储总结

1. k8s的挂载由pod进行挂载，同一个pod中多容器可以共享该pv。容器使用volume挂载，pod使用pv的概念。
2. EmpDir也属于一种挂载方式，区别于persistenceVolume存在。
3. PersistenceVolume是存储的最小单位，属于集群共享的资源，由集群管理员事先创建好。
4. hostPath属于PersistentVolume的一种，还有nfs等
5. pvc用于范围为存储卷申明，用于寻找特定的pv，由使用存储卷的人写。
6. 动态存储卷storageclass中最关键的为provisioner，动态存储卷的作用是替代集群管理员来创建pv。provisioner的不同实现有fuseim.pri/ifs等。

## ConfigMap示例

fortune-config.yaml

```yaml
apiVersion: v1 
kind: ConfigMap
metadata:
  namespace: default
  name: fortune-config
data: 
  sleep-interval: "25"
```

my-config.yaml （从文件创建）

```yaml
apiVersion: v1
data:
  a.txt: |
    a
kind: ConfigMap
metadata:
  creationTimestamp: "2021-02-01T12:03:48Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:a.txt: {}
    manager: kubectl-create
    operation: Update
    time: "2021-02-01T12:03:48Z"
  name: my-config
  namespace: default
  resourceVersion: "4987639"
  selfLink: /api/v1/namespaces/default/configmaps/my-config
  uid: b2307e0a-a78c-4e84-9154-0a671f5f9535
```

## ConfigMap挂载示例

```yaml
...
      containers:
        - name: postgres
          image: postgres:10.4
          imagePullPolicy: "IfNotPresent"
          ports:
            - containerPort: 5432
          envFrom:
            - configMapRef:
                name: postgres-config
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: postgredb
...
```

fortune-pod-env-configmap.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
name: fortune-env-from-configmap
spec:
  containers:
  - image: luksa/fortune:env
    env:
    - name: INTERVAL
    valueFrom:
      configMapKeyRef:
      name: fortune-config
      key: sleep-interval
...
```

## ConfigMap对-的处理

key中不能包含横杠。

> Did you notice I said two variables, but earlier, I said the ConfigMap has three entries
> (FOO, BAR, and FOO-BAR)? Why is there no environment variable for the FOO-BAR
> ConfigMap entry?
> The reason is that CONFIG_FOO-BAR isn’t a valid environment variable name
> because it contains a dash. Kubernetes doesn’t convert the keys in any way (it doesn’t
> convert dashes to underscores, for example). If a ConfigMap key isn’t in the proper
> format, it skips the entry (but it does record an event informing you it skipped it).

## ConfigMap用作存储nginx配置

fortune-config.yaml

```yaml
apiVersion: v1
data:
  my-nginx-config.conf: |
    server {
        listen 80;
        server_name www.kubia-example.com;
        gzip on;
        gzip_types text/plain application/xml;
        location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
        }
    }
  sleep-interval: |
    25
kind: ConfigMap
```

fortune-configmap-volume.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
name: fortune-configmap-volume
spec:
  containers:
  - image: nginx:alpine
    name: web-server
    volumeMounts:
    ...
    - name: config
      mountPath: /etc/nginx/conf.d
      readOnly: true
      ...
  volumes:
  ...
  - name: config
    configMap:
      name: fortune-config
...
```

完整示例：

nginx.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configmap
data:
  my-nginx-confg.conf: |
    server {
      listen 80;
      server_name localhost; 
      location / {
        proxy_pass  http://172.16.1.223:8981/;
        proxy_set_header   X-Forwarded-Proto $scheme;
        proxy_set_header   Host              $host;
        proxy_set_header   X-Real-IP         $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_set_header   Connection "";
      }  
    }
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
      - name: config
        configMap:
           name: nginx-configmap
      containers:
      - name: nginx
        image: nginx:latest
        volumeMounts:
        - name: config
          mountPath: /etc/nginx/conf.d
          readOnly: true
        resources:
          requests:
            memory: "164Mi"
            cpu: "400m"
          limits:
            memory: "200Mi"
            cpu: "400m"
        ports:
        - name: http
          containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: NodePort
```
