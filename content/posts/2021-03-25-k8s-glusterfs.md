---
title: k8s整合glusterfs做后端存储
tags: ["kubernetes"]
date: 2021-03-25
---

1. 安装glusterfs和heketi

   ```code
   #所有存储服务器下载安装glusterfs
   yum install centos-release-gluster -y
   yum install glusterfs-server -y
   
   #启动
   systemctl  start glusterd 
   
   #安装heketi
   yum install -y heketi heketi-client
   ```

2. 部署gluster集群

   ```code
   gluster peer probe node0
   gluster peer probe node1
   gluster peer probe node2
   
   gluster peer status
   
   mkdir /data/gluster/data -p
   
   gluster volume create glusterfs_volume replica 3 node0:/data/gluster/data node1:/data/gluster/data node2:/data/gluster/data force
   
   gluster volume info
   
   gluster volume start glusterfs_volume
   
   yum install -y glusterfs glusterfs-fuse
   ```

3. 配置heketi

   ```code
   [root@node0 ~]# cat /etc/heketi/heketi.json 
   {
     "_port_comment": "Heketi Server Port Number",
     "port": "8080",
   
     "_use_auth": "Enable JWT authorization. Please enable for deployment",
     "use_auth": false,
   
     "_jwt": "Private keys for access",
     "jwt": {
       "_admin": "Admin has access to all APIs",
       "admin": {
         "key": "My Secret"
       },
       "_user": "User only has access to /volumes endpoint",
       "user": {
         "key": "My Secret"
       }
     },
   
     "_glusterfs_comment": "GlusterFS Configuration",
     "glusterfs": {
       "_executor_comment": [
         "Execute plugin. Possible choices: mock, ssh",
         "mock: This setting is used for testing and development.",
         "      It will not send commands to any node.",
         "ssh:  This setting will notify Heketi to ssh to the nodes.",
         "      It will need the values in sshexec to be configured.",
         "kubernetes: Communicate with GlusterFS containers over",
         "            Kubernetes exec api."
       ],
       "executor": "mock",
   
       "_sshexec_comment": "SSH username and private key file information",
       "sshexec": {
         "keyfile": "/etc/heketi/heketi_key",
         "user": "root",
         "port": "22",
         "fstab": "/etc/fstab"
       },
   
       "_kubeexec_comment": "Kubernetes configuration",
       "kubeexec": {
         "host" :"https://127.0.0.1:8443",
         "cert" : "/path/to/crt.file",
         "insecure": false,
         "user": "kubernetes username",
         "password": "password for kubernetes user",
         "namespace": "OpenShift project or Kubernetes namespace",
         "fstab": "Optional: Specify fstab file on node.  Default is /etc/fstab"
       },
   
       "_db_comment": "Database file name",
       "db": "/var/lib/heketi/heketi.db",
   
       "_loglevel_comment": [
         "Set log level. Choices are:",
         "  none, critical, error, warning, info, debug",
         "Default is warning"
       ],
       "loglevel" : "debug"
     }
   }
   ```

4. 配置免密登陆

   ```code
   #设置heketi免密访问GlusterFS
   [root@master heketi]# ssh-keygen -t rsa -q -f /etc/heketi/heketi_key -N ""
   [root@master heketi]# chown heketi:heketi /etc/heketi/heketi_key
   
   #分发公钥
   [root@master heketi]# ssh-copy-id -i /etc/heketi/heketi_key.pub root@master
   [root@master heketi]# ssh-copy-id -i /etc/heketi/heketi_key.pub root@node1
   #将秘钥充master服务器复制到node1服务器上
   [root@master heketi]# rsync -avz /etc/heketi/heketi_key root@node1:/etc/heketi/
   ```

5. 启动heketl

   ```code
   systemctl enable heketi
   systemctl restart heketi
   systemctl status heketi
   
   [root@master ~]# curl http://localhost:8080/hello
   Hello from Heketi
   ```

6. 设置集群

   ```code
   [root@node0 ~]# cat /etc/heketi/topology.json 
   {
       "clusters": [
           {
               "nodes": [
                   {
                       "node": {
                           "hostnames": {
                               "manage": [
                                   "192.168.90.219"
                               ],
                               "storage": [
                                   "192.168.90.219"
                               ]
                           },
                           "zone": 1
                       },
                       "devices": [
                           "/dev/vdb"
                       ]
                   },
                   {
                       "node": {
                           "hostnames": {
                               "manage": [
                                   "192.168.90.217"
                               ],
                               "storage": [
                                   "192.168.90.217"
                               ]
                           },
                           "zone": 2
                       },
                       "devices": [
                           "/dev/vdb"
                       ]
                   },
                   {
                       "node": {
                           "hostnames": {
                               "manage": [
                                   "192.168.90.216"
                               ],
                               "storage": [
                                   "192.168.90.216"
                               ]
                           },
                           "zone": 3
                       },
                       "devices": [
                           "/dev/vdb"
                       ]
                   }
               ]
           }
       ]
   }
   ```

7. 通过topology.json组建GlusterFS集群

   ```code
   [root@master ~]# heketi-cli --server http://localhost:8080 --user admin --secret admin@key topology load --json=/etc/heketi/topology.json
   Creating cluster ... ID: 2865ef5ac77aae777bbfaf3f27e456ef
       Allowing file volumes on cluster.
       Allowing block volumes on cluster.
       Creating node 172.16.208.210 ... ID: 474894862effef22952e7c0d4542605b
           Adding device /dev/vdb ... OK
       Creating node 172.16.208.211 ... ID: 156c6b793ef761f68b317d0cfe8e7ec1
           Adding device /dev/vdb ... OK
           
           
   [root@master ~]# heketi-cli --server http://localhost:8080 --user admin --secret admin@key topology info
   
   ```

8. 配置StorageClass

   ```code
   [root@master ~]# cat  gluster-heketi-secret.yaml 
   apiVersion: v1
   kind: Secret
   metadata:
     name: heketi-secret
     namespace: default
   data:
     # base64 encoded password. E.g.: echo -n "mypassword" | base64
     key: *
   type: kubernetes.io/glusterfs
   
   [root@master ~]# kubectl  apply -f gluster-heketi-secret.yaml 
   secret/heketi-secret created
   
   [root@master ~]# cat  gluster-heketi-storageclass.yaml 
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: gluster-heketi-storageclass
   provisioner: kubernetes.io/glusterfs
   reclaimPolicy: Delete
   parameters:
     resturl: "http://192.168.*.*:8080"
     restauthenabled: "true"
     restuser: "admin"
     secretNamespace: "default"
     secretName: "heketi-secret"
     volumetype: "replicate:2"
   
   [root@master ~]# kubectl apply -f  gluster-heketi-storageclass.yaml 
   storageclass.storage.k8s.io/gluster-heketi-storageclass created
   ```

9. 测试

   ```code
   [root@master ~]# cat test-pvc.yaml 
   kind: PersistentVolumeClaim
   apiVersion: v1
   metadata:
     name: test-claim
     annotations:
       volume.beta.kubernetes.io/storage-class: "gluster-heketi-storageclass"
   spec:
     accessModes:
       - ReadWriteMany
     resources:
       requests:
         storage: 1Gi
   ```
