---
title: 使用deployment来模拟daemonset
date: 2022-05-12
---

一个yaml文件

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    id: very-important
  name: deploy-important
  namespace: project-tiger
spec:
  replicas: 3
  selector:
    matchLabels:
      id: very-important
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        id: very-important
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - topologyKey: kubernetes.io/hostname
            labelSelector:
              matchExpressions:
              - key: id
                operator: In
                values:
                - very-important 
      containers:
      - image: nginx:1.17.6
        name: container1
        resources: {}
      - name: container2
        image: kubernetes/pause
status: {}
```

podAntiAffinity 告诉调度器避免多个带有id标签的pod在同一个节点上运行，这样就可以保证同一个节点上只有一个带有id标签的pod。
