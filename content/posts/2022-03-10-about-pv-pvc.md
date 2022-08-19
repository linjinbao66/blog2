---
title: 关于pv pvc的新认识
date: 2022-03-10
---

新认识

1. hostpath可以直接使用

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-hostpath
spec:
  nodeSelector:
    kubernetes.io/hostname: i-6fns0nua
  containers:
  - image: busybox
    name: app
    volumeMounts:
    - mountPath: /logs
      name: shared-dir
    args:
    - /bin/sh
    - -c
    - echo hostpath >> /logs/app.log; sleep 60000
  volumes:
  - name: shared-dir
    hostPath:
      path: /data/logs
```

1. hostpath可以作为pv后端使用

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local1
spec:
  capacity:
    storage: 30Gi 
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local
  local:
    path: /data/local1
  nodeAffinity:
      required:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/hostname
            operator: In
            values:
            - i-6fns0nua
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
spec:
  storageClassName: local
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 30Gi
---
kind: Pod
apiVersion: v1
metadata:
  name: pod1
spec:
  containers:
    - name: nginx1
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: pod1
  volumes:
    - name: pod1
      persistentVolumeClaim:
        claimName: pvc1
```
