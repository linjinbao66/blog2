---
title: esxi-cloud-init使用
date: 2022-03-03
tags: ["虚拟化"]
---

esxi是虚拟化的常见方案，其直接部署到物理机上，在之上可以虚拟化出来不同的系统。部署一个虚拟机，我们常见的操作方式是登录esxi的控制台或者vcenter的控制台操作，步骤比较繁琐。

cloud-init是云厂商常用的方案，用于工作在虚拟机初始化的时候。例如，原生部署虚拟机的时候，需要在网页上设置磁盘，用户，网络等等，在cloud-init中，这些不必操作；原生部署虚拟机的时候，无法添加自定义源，cloud-init可以实现。

## 使用到的工具

* govc 

govc 是vmware出的cli工具，用于操作vcenter

## 第一步 配置esxi.env配置

```shell
# vCenter host
export GOVC_URL=192.168.123.138
# vCenter credentials
export GOVC_USERNAME=administrator@vsphere.local
export GOVC_PASSWORD=rpK0qGVhd#YIxP4~S26+
# disable cert validation
export GOVC_INSECURE=1
export GOVC_DATASTORE=bigdata
export GOVC_NETWORK=""
export GOVC_RESOURCE_POOL='default-cluster/Resources'
export GOVC_DATACENTER=default-dc
```

```shell
jinbao666lin@jinbao666deMacBook-Pro govc-practice % govc about
FullName:     VMware vCenter Server 6.7.0 build-17712750
Name:         VMware vCenter Server
Vendor:       VMware, Inc.
Version:      6.7.0
Build:        17712750
OS type:      linux-x64
API type:     VirtualCenter
API version:  6.7.3
Product ID:   vpx
UUID:         e8dd6d90-2d67-4803-a46a-22494e87a678
```

## 第二步 准备ubuntu模版

[https://cloud-images.ubuntu.com/releases/18.04/release/ubuntu-18.04-server-cloudimg-amd64.ova ↗](https://cloud-images.ubuntu.com/releases/18.04/release/ubuntu-18.04-server-cloudimg-amd64.ova)

下载到`~/Downloads`目录

## 第三步 导出模版spec

```shell
govc import.spec ~/Downloads/ubuntu-18.04-server-cloudimg-amd64.ova | python -m json.tool > ubuntu.json
```

ubuntu.json是模版的信息所在

## 第四步 准备user-data文件

创建user-data文件，里面的信息定义了启动脚本的信息：

```yaml
#cloud-config
chpasswd:
    list: |
      deploy:369369
    expire: false
groups:
  - deploy
users:
  - default
  - name: deploy
    ssh-authorized-keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDXyE5xMca36mPqld/vA75oDNIxc955RLGR9oHDmo8Rko4wUUZj0A0Zgqiykyx6KHKAF/IdXKt7zLQsJbMPl09sE74g0UAt96rpGEj0rVrxH+pjrcl8HwT68ZSXuJrqGzBIzN/elKvwK9Zsf6OYzrHsBt0yGsOvTw2z9AT9ProjTldxPcyssWrPoYeP3qcdOfugn4y2ZKSweSO78HUd9dz/7Tj4zPSWQq+fPsCkb0WzvfaseJbLe3fJ3ZrQfADv4EIv3zsQd6/lZ4ypM/7KSCsal0qcX3buP0bsDom9TEwdRTswAk9NYIg2NwZPEWqU8HNccJKRFBMxIVS1aC8NMIhYveckUrsmHlXXYvqwMseYfb9V8FIyUuwLkzv3uNlIidtdQv0sas5/3f3T7UBS1Cv9hZQnJ6KFVjSg4ocOIA5mcAFYUIkSrvmHLxWZ7SNtUS6l0bucBwuI/JlXLkMAhJn69c261GEIrpBCOgKKsu8RNLWUoaPbGC+gJzS+YjZQa8M= jinbao666lin@jinbao666deMacBook-Pro.local
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: sudo, deploy
    shell: /bin/bash
package_upgrade: false
runcmd:
  - swapoff --all
  - sed -ri '/\sswap\s/s/^#?/#/' /etc/fstab
  - sysctl net.bridge.bridge-nf-call-iptables=1
  - sysctl net.bridge.bridge-nf-call-ip6tables=1
  - 'echo "disable_vmware_customization: false" >> /etc/cloud/cloud.cfg'
  - sed -i 's/D \/tmp 1777 root root -/#D \/tmp 1777 root root -/g' /usr/lib/tmpfiles.d/tmp.conf
  - echo -n > /etc/machine-id
final_message: "The system is prepped, after $UPTIME seconds"
power_state:
  timeout: 30
  mode: poweroff
```

参考：https://cloudinit.readthedocs.io/en/latest/topics/examples.html

## 第五步 修改模版spec

ubuntu.json修改，最终形态：

```json
{
    "DiskProvisioning": "thin",
    "IPAllocationPolicy": "dhcpPolicy",
    "IPProtocol": "IPv4",
    "InjectOvfEnv": false,
    "MarkAsTemplate": false,
    "Name": "template-ubuntu-1804",
    "NetworkMapping": [
        {
            "Name": "VM Network",
            "Network": ""
        }
    ],
    "PowerOn": true,
    "PropertyMapping": [
        {
            "Key": "instance-id",
            "Value": "id-ovf"
        },
        {
            "Key": "hostname",
            "Value": ""
        },
        {
            "Key": "seedfrom",
            "Value": ""
        },
        {
            "Key": "public-keys",
            "Value": ""
        },
        {
            "Key": "user-data",
            "Value": "I2Nsb3VkLWNvbmZpZwpjaHBhc3N3ZDoKICAgIGxpc3Q6IHwKICAgICAgZGVwbG95OjM2OTM2OQogICAgZXhwaXJlOiBmYWxzZQpncm91cHM6CiAgLSBkZXBsb3kKdXNlcnM6CiAgLSBkZWZhdWx0CiAgLSBuYW1lOiB1YnVudHUKICAgIHNzaC1hdXRob3JpemVkLWtleXM6CiAgICAgIC0gc3NoLXJzYSBBQUFBQjNOemFDMXljMkVBQUFBREFRQUJBQUFCZ1FEWHlFNXhNY2EzNm1QcWxkL3ZBNzVvRE5JeGM5NTVSTEdSOW9IRG1vOFJrbzR3VVVaajBBMFpncWl5a3l4NktIS0FGL0lkWEt0N3pMUXNKYk1QbDA5c0U3NGcwVUF0OTZycEdFajByVnJ4SCtwanJjbDhId1Q2OFpTWHVKcnFHekJJek4vZWxLdndLOVpzZjZPWXpySHNCdDB5R3NPdlR3Mno5QVQ5UHJvalRsZHhQY3lzc1dyUG9ZZVAzcWNkT2Z1Z240eTJaS1N3ZVNPNzhIVWQ5ZHovN1RqNHpQU1dRcStmUHNDa2IwV3p2ZmFzZUpiTGUzZkozWnJRZkFEdjRFSXYzenNRZDYvbFo0eXBNLzdLU0NzYWwwcWNYM2J1UDBic0RvbTlURXdkUlRzd0FrOU5ZSWcyTndaUEVXcVU4SE5jY0pLUkZCTXhJVlMxYUM4Tk1JaFl2ZWNrVXJzbUhsWFhZdnF3TXNlWWZiOVY4Rkl5VXV3TGt6djN1TmxJaWR0ZFF2MHNhczUvM2YzVDdVQlMxQ3Y5aFpRbko2S0ZWalNnNG9jT0lBNW1jQUZZVUlrU3J2bUhMeFdaN1NOdFVTNmwwYnVjQnd1SS9KbFhMa01BaEpuNjljMjYxR0VJcnBCQ09nS0tzdThSTkxXVW9hUGJHQytnSnpTK1lqWlFhOE09IGppbmJhbzY2NmxpbkBqaW5iYW82NjZkZU1hY0Jvb2stUHJvLmxvY2FsCiAgICBzdWRvOiBBTEw9KEFMTCkgTk9QQVNTV0Q6QUxMCiAgICBncm91cHM6IHN1ZG8sIGRlcGxveQogICAgc2hlbGw6IC9iaW4vYmFzaApwYWNrYWdlX3VwZ3JhZGU6IGZhbHNlCnJ1bmNtZDoKICAtIHN3YXBvZmYgLS1hbGwKICAtIHNlZCAtcmkgJy9cc3N3YXBccy9zL14jPy8jLycgL2V0Yy9mc3RhYgogIC0gc3lzY3RsIG5ldC5icmlkZ2UuYnJpZGdlLW5mLWNhbGwtaXB0YWJsZXM9MQogIC0gc3lzY3RsIG5ldC5icmlkZ2UuYnJpZGdlLW5mLWNhbGwtaXA2dGFibGVzPTEKICAtICdlY2hvICJkaXNhYmxlX3Ztd2FyZV9jdXN0b21pemF0aW9uOiBmYWxzZSIgPj4gL2V0Yy9jbG91ZC9jbG91ZC5jZmcnCiAgLSBzZWQgLWkgJ3MvRCBcL3RtcCAxNzc3IHJvb3Qgcm9vdCAtLyNEIFwvdG1wIDE3Nzcgcm9vdCByb290IC0vZycgL3Vzci9saWIvdG1wZmlsZXMuZC90bXAuY29uZgogIC0gZWNobyAtbiA+IC9ldGMvbWFjaGluZS1pZApmaW5hbF9tZXNzYWdlOiAiVGhlIHN5c3RlbSBpcyBwcmVwcGVkLCBhZnRlciAkVVBUSU1FIHNlY29uZHMiCnBvd2VyX3N0YXRlOgogIHRpbWVvdXQ6IDMwCiAgbW9kZTogcG93ZXJvZmY="
        },
        {
            "Key": "password",
            "Value": ""
        }
    ],
    "WaitForIP": false
}
```

`user-data`内容使用的是`user-data`的base64编码后的值。`base64 user-data`

## 第六步 部署虚拟机

```shell
govc import.ova -options=ubuntu.json ~/Downloads/ubuntu-18.04-server-cloudimg-amd64.ova 
```

部署完成，即出现了一个虚拟机，使用`deploy|ubuntu`用户登录