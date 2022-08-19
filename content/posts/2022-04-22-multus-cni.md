---
title: k8sd多网卡方案
date: 2022-04-22
---

## 简介


k8s中，默认为每个容器分配一个网卡，（除了lookback回环外），在一些特定场景下，需要指定网卡信息，或者增加网卡，则需要制定多网卡方案。
场景如下：
应用启动时候需要检查mac和ip地址和证书验证，此时如果使用默认的动态网卡方案，则签发的证书无法长期生效。

## 方案

使用方案 multus-cni 
该方案，会在每个节点部署守护进程，根据pod上的注解信息，决定添加自定义网卡。GitHub地址:https://github.com/k8snetworkplumbingwg/multus-cni.git

## 使用

### 第一步

部署crd文件，multus-daemonset-thick-plugin.yml
```yaml
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: network-attachment-definitions.k8s.cni.cncf.io
spec:
  group: k8s.cni.cncf.io
  scope: Namespaced
  names:
    plural: network-attachment-definitions
    singular: network-attachment-definition
    kind: NetworkAttachmentDefinition
    shortNames:
      - net-attach-def
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          description: 'NetworkAttachmentDefinition is a CRD schema specified by the Network Plumbing
            Working Group to express the intent for attaching pods to one or more logical or physical
            networks. More information available at: https://github.com/k8snetworkplumbingwg/multi-net-spec'
          type: object
          properties:
            apiVersion:
              description: 'APIVersion defines the versioned schema of this represen
                tation of an object. Servers should convert recognized schemas to the
                latest internal value, and may reject unrecognized values. More info:
                https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
              type: string
            kind:
              description: 'Kind is a string value representing the REST resource this
                object represents. Servers may infer this from the endpoint the client
                submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
              type: string
            metadata:
              type: object
            spec:
              description: 'NetworkAttachmentDefinition spec defines the desired state of a network attachment'
              type: object
              properties:
                config:
                  description: 'NetworkAttachmentDefinition config is a JSON-formatted CNI configuration'
                  type: string
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: multus
rules:
  - apiGroups: ["k8s.cni.cncf.io"]
    resources:
      - '*'
    verbs:
      - '*'
  - apiGroups:
      - ""
    resources:
      - pods
      - pods/status
    verbs:
      - get
      - update
  - apiGroups:
      - ""
      - events.k8s.io
    resources:
      - events
    verbs:
      - create
      - patch
      - update
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: multus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: multus
subjects:
  - kind: ServiceAccount
    name: multus
    namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: multus
  namespace: kube-system
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-multus-ds
  namespace: kube-system
  labels:
    tier: node
    app: multus
    name: multus
spec:
  selector:
    matchLabels:
      name: multus
  updateStrategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        tier: node
        app: multus
        name: multus
    spec:
      hostNetwork: true
      tolerations:
        - operator: Exists
          effect: NoSchedule
        - operator: Exists
          effect: NoExecute
      serviceAccountName: multus
      containers:
        - name: kube-multus
          image: ghcr.io/k8snetworkplumbingwg/multus-cni:thick
          command: [ "/usr/src/multus-cni/bin/multus-daemon" ]
          args:
            - "-cni-version=0.3.1"
            - "-cni-config-dir=/host/etc/cni/net.d"
            - "-multus-autoconfig-dir=/host/etc/cni/net.d"
            - "-multus-log-to-stderr=true"
            - "-multus-log-level=verbose"
          resources:
            requests:
              cpu: "100m"
              memory: "50Mi"
            limits:
              cpu: "100m"
              memory: "50Mi"
          securityContext:
            privileged: true
          volumeMounts:
            - name: cni
              mountPath: /host/etc/cni/net.d
            - name: cnibin
              mountPath: /host/opt/cni/bin
      initContainers:
        - name: install-multus-binary
          image: ghcr.io/k8snetworkplumbingwg/multus-cni:thick
          command:
            - "cp"
            - "/usr/src/multus-cni/bin/multus"
            - "/host/opt/cni/bin/multus"
          resources:
            requests:
              cpu: "10m"
              memory: "15Mi"
          securityContext:
            privileged: true
          volumeMounts:
            - name: cnibin
              mountPath: /host/opt/cni/bin
              mountPropagation: Bidirectional
        - name: generate-kubeconfig
          image: ghcr.io/k8snetworkplumbingwg/multus-cni:thick
          command:
            - "/usr/src/multus-cni/bin/generate-kubeconfig"
          args:
            - "-k8s-service-host=$(KUBERNETES_SERVICE_HOST)"
            - "-k8s-service-port=$(KUBERNETES_SERVICE_PORT)"
          resources:
            requests:
              cpu: "10m"
              memory: "15Mi"
          securityContext:
            privileged: true
          volumeMounts:
            - name: cni
              mountPath: /host/etc/cni/net.d
              mountPropagation: Bidirectional
      terminationGracePeriodSeconds: 10
      volumes:
        - name: cni
          hostPath:
            path: /etc/cni/net.d
        - name: cnibin
          hostPath:
            path: /opt/k8s/bin/
```
注意挂载的宿主机目录，需要cni插件在其中

### 使用

sample.yaml
```yaml
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: ip-192
spec:
  config: '{
            "cniVersion": "0.3.1",
            "plugins": [
                {
                    "type": "macvlan",
                    "capabilities": { "ips": true },
                    "master": "eth0",
                    "mode": "bridge",
                    "ipam": {
                        "type": "static"
                    }
                } ]
        }'
---
apiVersion: v1
kind: Pod
metadata:
  name: centos-runtimeconfig
  annotations:
    k8s.v1.cni.cncf.io/networks: '[
            { "name": "ip-192",
              "ips": [ "192.168.111.10/24" ],
              "mac": "20:01:02:03:04:05" }
    ]'
spec:
  containers:
  - name: centos-runtimeconfig
    image: docker.io/centos/tools:latest
    command:
    - /sbin/init
    securityContext:
      privileged: true
```
