---
title: network-policy
date: 2022-04-03
---

## 一个网络隔离策略讲解

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-networkpolicy
  namespace: mysql
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: internal
    ports:
    - protocol: TCP
      port: 8080
```

## 解释

该策略：

1. 作用与mysql名称空间下所有的pod

2. 只有名称空间带有`name=internal`标签的下的pod可以访问

3. 作用端口为8080端口，在流量入站
