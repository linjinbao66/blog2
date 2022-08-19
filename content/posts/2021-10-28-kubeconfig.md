---
title: kubeconfig签发
date: 2021-10-28
---

# 签发一个kubeconfig文件的详细流程

大致步骤：

1. cfssl证书
2. 定义集群
3. 定义用户
4. 定义上下文
5. 绑定角色

## cfssl证书

ca-config.json cfssl的配置文件

```json
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "876000h"
      }
    }
  }
}
```

user-csr.json 当前证书签发内容

```json
{
  "CN": "linjb",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```

```shell
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes user-csr.json | cfssljson -bare user
```

```shell
[root@node1 practice]# ls -l user* 
-rw-r--r-- 1 root root  993 10月 28 17:17 user.csr
-rw-r--r-- 1 root root  217 10月 28 16:59 user-csr.json
-rw------- 1 root root 1679 10月 28 17:17 user-key.pem
-rw-r--r-- 1 root root 1294 10月 28 17:17 user.pem
```

## 定义集群

```shell
kubectl config set-cluster linjb --server=https://127.0.0.1:8443 --certificate-authority=ca.pem --embed-certs=true --kubeconfig=linjb.kubeconfig
```

## 定义用户

```shell
kubectl config set-credentials linjb --client-certificate=user.pem --client-key=user-key.pem --embed-certs=true --kubeconfig=linjb.kubeconfig 
```

## 定义上下文

```shell
kubectl config set-context linjb --cluster=linjb --user=linjb --kubeconfig=linjb.kubeconfig 
```

## 使用上下文

```shell
kubectl config view --kubeconfig=linjb.kubeconfig
```



## 创建rolebinding

```shell
kubectl create rolebinding linjb-binding --clusterrole=admin --user=linjb 
##user=linjb为user-csr.json文件中的CN
##clusterrole=admin为集群中已存在的角色
```

## 完整的kubeconfig文件

linjb.kubeconfig

```shell
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURvRENDQW9pZ0F3SUJBZ0lVR2ZMckUrRHJNZWZud20zejd1a3h1alFJOGIwd0RRWUpLb1pJaHZjTkFRRUwKQlFBd1p6RUxNQWtHQTFVRUJoTUNRMDR4RURBT0JnTlZCQWdUQjBKbGFVcHBibWN4RURBT0JnTlZCQWNUQjBKbAphVXBwYm1jeEREQUtCZ05WQkFvVEEyczRjekVPTUF3R0ExVUVDeE1GWVcxeWIyMHhGakFVQmdOVkJBTVREV3QxClltVnlibVYwWlhNdFkyRXdJQmNOTWpFd05ESTNNRGswT0RBd1doZ1BNakV5TVRBME1ETXdPVFE0TURCYU1HY3gKQ3pBSkJnTlZCQVlUQWtOT01SQXdEZ1lEVlFRSUV3ZENaV2xLYVc1bk1SQXdEZ1lEVlFRSEV3ZENaV2xLYVc1bgpNUXd3Q2dZRFZRUUtFd05yT0hNeERqQU1CZ05WQkFzVEJXRnRjbTl0TVJZd0ZBWURWUVFERXcxcmRXSmxjbTVsCmRHVnpMV05oTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUE0c3YzVWJlTVlHOE4KcXBqSzRXTmliNk1VamVCcWorbVZuWjNybTdZc3JQY1pkdWFzbERzYkMxUTR0WkRGVWxIakNCY0ssXlMMgpTQkI4QnJsM0hDS21KN21vZzh4Q3dJRVo2Z3BML09mSVNzSlpKSUVuQTAzVEc1eWxCaFVFMzVid1hvU2xTdkorCnRJN3lyTlBiSWdTblhkMXA3Tm5ZaXJPVjZINkF6SGNmbmJBT3hOZy9ZK3ZLMWJUbndlNlNTM2xyUWNDb3I1ZUIKblM2dTlwQU5KWWlyQy8wbkdkYzVWdU5JckNIMDdTL2xRcUtuUk8ybnBUMkdxOFZCRTVpNnZzdXIrOGkxVGl4NwpIVDlFbHdUcjAyZlc4ak54SkdSL0M3SXR2WUNSUjRDOExpNGpkek1DQ3NwQUxFaTY4Q0JjQUd1eGpWbE5pQ1FSCksrZjR2L0Y3M3dJREFRQUJvMEl3UURBT0JnTlZIUThCQWY4RUJBTUNBUVl3RHdZRFZSMFRBUUgvQkFVd0F3RUIKL3pBZEJnTlZIUTRFRmdRVW5Lci80cGU4SXdCbHY4MmFxUFErdXFIbXNNQXdEUVlKS29aSWh2Y05BUUVMQlFBRApnZ0VCQUp1V092UE8xRUQybkY4YTRTWWFwaUFvbDJKVWNSN0RVR01pZ1hhYk9LcWVXV3ZTUWtLc2o3MnlRR1lTCjBFcEVwUlRTK1M5ZTRkS3UzQVN1cjhncU1SWTROMG1KelQvMFdJSzZZZlFXWU5pVDgwR2swakltbVJSWnZqRzkKYWVOT2J5R3hWSC9jTmdFaGZ5THJTZGQ1MkVHODNFNXRNU1RTekRmWUpWaFk4b2lmTGd0MFpXdU9CaXpQK3JNUApGdXQzeGhlWlR5M1RMMjJ2NGxYQkNmSmZ4bFJOVXZUWkNMalFKNFV2VUNpYlR0NGkzbUFtNzUyeG1MVjV1MzQ0ClJEV0EvZ1VFcG8zOVNmU2QxaWZJZVN4UUFyWkJxM3lzUDhPQStWZ1Nic3VDemVrTEsxMkNIeWx5c3pzZSs5S0MKc2s3V1FETjNVVmF4eW96V3JXVlpoVEZFdlFVPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://127.0.0.1:8443
  name: linjb
contexts:
- context:
    cluster: linjb
    user: linjb
  name: linjb
current-context: linjb
kind: Config
preferences: {}
users:
- name: linjb
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUQxakNDQXI2Z0F3SUJBZ0lVWmE2YzdnTW11Z2lGR2sxTC95VElBaVlUcDFRd0RRWUpLb1pJaHZjTkFRRUwKQlFBd1p6RUxNQWtHQTFVRUJoTUNRMDR4RURBT0JnTlZCQWdUQjBKbGFVcHBibWN4RURBT0JnTlZCQWNUQjBKbAphVXBwYm1jeEREQUtCZ05WQkFvVEEyczRjekVPTUF3R0ExVUVDeE1GWVcxeWIyMHhGakFVQmdOVkJBTVREV3QxClltVnlibVYwWlhNdFkyRXdJQmNOTWpFeE1ESTVNREkxTVRBd1doZ1BNakV5TVRFd01EVXdNalV4TURCYU1HQXgKQ3pBSkJnTlZCQVlUQWtOT01SQXdEZ1lEVlFRSUV3ZENaV2xLYVc1bk1SQXdEZ1lEVlFRSEV3ZENaV2xLYVc1bgpNUXd3Q2dZRFZRUUtFd05yT0hNeER6QU5CZ05WQkFzVEJsTjVjM1JsYlRFT01Bd0dBMVVFQXhNRmJHbHVhbUl3CmdnRWlNQTBHQ1NxR1NJYjNEUUVCQVFVQUE0SUJEd0F3Z2dFS0FvSUJBUUM5RTZyR0ZPbUFveUdGaVgrNkJBemIKZlpQZEZCNnQ5QTRScHZ5WHdNOTdCUi9jaW1UMnlXc3FVblMrQmpRelpMcFBjUEJPeE00UmlJbjZQNmN6L2ZYcApyUTJkSWNGTENHZFZqUytjZ0ZHZHp2eXdUUHE3YWVMM3A4MnhKTC93OUZkTE1EdDFuZzE1YjVVSTNlOEdyOFhJCjZTR1lKRHNzRzRFNHQ4RUpUNzBIQWRDUkJaeXpLVXhuaEI0UVc3a3FiZk1iWDdtM3VEb0hzZFRBMklEUFZtcHkKaG9PYk5UVDBXckhvSmUxcGZ0T3dTQmU4d0ljV3k1NThaR1FxZFprRGVVM2FVbssN0MHoyRkJaCkFnTUJBQUdqZnpCOU1BNEdBMVVkRHdFQi93UUVBd0lGb0RBZEJnTlZIU1VFRmpBVUJnZ3JCZ0VGQlFjREFRWUkKS3dZQkJRVUhBd0l3REFZRFZSMFRBUUgvQkFJd0FEQWRCZ05WSFE0RUZnUVU0SEluMnVScWtRVWNVTHJqb2RBZApwOFAycWM4d0h3WURWUjBqQkJnd0ZvQVVuS3IvNHBlOEl3Qmx2ODJhcVBRK3VxSG1zTUF3RFFZSktvWklodmNOCkFRRUxCUUFEZ2dFQkFFd2NpUEJoZnFaMEtEeEpmUTVPUXgrMVFwQXZQUWh1QVBzbDBDYjIzaXE5VFJnU3hmeE0KQ2FVS0U4cVp6dkRqb044OHBnYVc3WXorWG8rd1FhUU0yRmNudE9EK1VhZ2hjL2lHM0ZoVzFicW1OcHlVVXE0dwpCT2VtR3RRSHdHYzAvNllma1dPRWM5TFlDRjdPN29rcmI5MS9VbVV6MDIwQVkzTG8wam5HNGIwaEdkbG96cVF1CjZuNlFPZzl6OEhxdHlLV2M5bnQ1VmZ6Y1d4REt0aGFJUWVmR3grd2EwYnV5WlFzVjliaENtODBNWjQwM05XN3AKNjNtNVVuK0xUWE84NmNHUmVaRWdLeEErNmlyayt1d1duZjVZczBON0RpNkgwM1piZHFuWmg0Vm1GdC9nY0x1RQo3aDR3RFZTcWR1OFBPcnVYQWU2eWhEK1JDYmh6SWg1djdqMD0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBdlJPcXhoVHBnS01oaFlsL3VnUU0yMzJUM1JRZXJmUU9FYWI4bDhEUGV3VWYzSXBrCjlzbHJLbEowdmdZME0yUzZUM0R3VHNUT0VZaUoraituTS8zMTZhME5uU0hCU3doblZZMHZuSUJSbmM3OHNFejYKdTJuaTk2Zk5zU1MvOFBSWFN6QTdkWjROZVcrVkNOM3ZCcS9GeU9raG1DUTdMQnVCT0xmQkNVKzlCd0hRa1FXYwpzeWxNWjRRZUVGdTVLbTN6RzErNXQ3ZzZCN0hVd05pQXoxWnFjb2FEbXpVMDlGcXg2Q1h0YVg3VHNFZ1h2TUNICkZzdWVmR1JrS25XWkEzbE4ybEtFU3NYUE9keW5YSHFaZmtYamVISUEzMm9IZnhwdFJlcVFjcVRCQmZOVTlmNk8Kdy9VbUlPdWs3NnRqWHFoQWRZMTNIcmt6VmFGSzZITGRNOWhRV1FJREFRQUJBb0lCQUh2UWVUQWxXWk1uUURoVwpCaElsdk5XdXNqay9oNmVaL2V5SlVUZCt4MTlqeDYxLzR3WEllQ2pLdmpBQm1BVmZuTEdRMzR4MVRBd25RVk5pCmczZUVncGgyL2tjN1ozeGZFR3Z4ZkpBYloxYlR5SjBhaThaV1hJNllrQlhFWHZ6R3hMTXo3bnZpK0NmssUKb2tYYXJNWVlCQ3ZBN2c3QUpDcWtDZ0N5K3JHdTJVc3g5VmtuMzd1bGRHQlpodWloQWdRb0hFVFc1UEVDcTUzeAorWmpjb2hNbG9sMlduU0Q0Rm5sZUNFUTR5YVk4SUdSblhsSWVaRWNCK1hSZ0k4R0ZLQmNtZ3hBY1p4dEc0RHJwCkJwSzBXYUZRMzhjQ29JUllldWpmYURqZXVTTTluc1ZxbHlpZy9qMnJ6VFM3dVRWdzdFZVB4WVIvWWs3MVpIdHAKeEFqNFJnRUNnWUVBeVp2WkxZcmJyMkgxMmNJdHQvWmtvK2NQUTNONGdDWk01NmE2M0RtVVhBZVFTdnlKM0d5YwpLcFhxNllBZVRmbFBKSW05VWEzclhyTTZ6WUR0ZTQxaVhydmhlOW14N0o4ZE9wd2t4eXdWd2VMZUxMS29KUGF2CjJ6S2daQWtkSCtHZmRpdzF2VDdFNnVCSVVUdnZ3YUJIUTZsS2tGUzNQdWhjeW91YWlNT0xrMmtDZ1lFQThCWkwKNXk1dGxNcnNXVkRYT202Ukc2SjlYWElwWWNtdDNUc25OcmJLcHdnRGhyNXlIS3FsSHRPK0hvenBPM0ZRR05MLwp3YWM0S1RxdGRvNUYveVlDRWdCUVlPYXFiUmZXUFFzYlZKbnhjbzZlZk9yZEx5ZkFUbG5iQUNZem1yeE5qeCtnCmEwaExtelFyckVUeFdxNFJ3cGxkejA4R1lFbFYyZlJNYWk3bFozRUNnWUFCNnFCYTVYb0hKY2cvaExBSWtxZ00KUXRNTFVocXdKUzBQK3E3R0R5b1E0ZVdHUVBaU2lSbkc4ZHZrMGxuM1pjcFJ6NWxrSUdJZmFWYkN3MW4rbGE1OAp0ZzZEcmVNYUc3MGNaSVdZK3h0TjE0bERKWU9ocmxLbm84aVFpdUdpL3ZNVUhZRjVSaEo3SlJ5cTRRWDdjam1iCk5BM0laM1hDZkZUOWUreEJKcEs2R1FLQmdGUWs3RnRNMlZrUnViNGY2Qjd4aTJmdERoVUhJdHZEN0d5aUE3OVkKVnpRdFNkY2F2akd2MlFreDJ1c29KY0lRbDZycm9IbUZtdGhRV1dVNHZlMkxxNlFWZWhaTUlhVDBlc0NRT1BidwpzVCtlSG92WFlNZmJIYm9ZSWZvdWFWMnM2MTNqRHIwcTdGeXI3emFFUHpheDFVV25yTW93ZnlLVjhVSGZManowCnpLL3hBb0dBZEhURkZoNzN5eTdQbldqbWtyUW1idUFrUk93d1MyK1VRV0JZZjNaWnkxbTFHaDJqcVJKTk9oYVkKWlVlMHFoRm1vV280NU1KVVFUem10YldZNm9xdUtFMXpXSGJOd3E2dmZ3NXQxVUc4YkxaQnh4Q2U0TVc2RWNiMAo1dHR4ZWs1REUycG5BN2d3RVcvbXdPM0IrZGJsWHNtTTd2a2FxU1RsOEZsT2xhamJ4UUU9Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
```
