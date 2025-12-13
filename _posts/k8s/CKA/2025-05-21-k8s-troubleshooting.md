---
author: "unggu"
title: "[CKA] TroubleShooting - 1"
date: 2025-05-08 11:11:12 +0800
categories: [k8s, CKA]
tags: [CKA, k8s, Container, Docker]
render_with_liquid: true
comments: true
image:
  path: assets/img/metaimg/cka/cka.png
---

## Case 1-1. Service의 이름이 잘못 명명됨

![image.png]({{ site.baseurl }}{{ page.url }}/img/kt1.png)

2계층 구조의 web service에서 접속이 되지 않는다.

**pod와 svc, ep를 확인**

```yaml
controlplane ~ ➜  k get pods -n alpha
NAME                            READY   STATUS    RESTARTS   AGE
mysql                           1/1     Running   0          3m31s
webapp-mysql-5c4c675768-qw4vb   1/1     Running   0          3m31s

controlplane ~ ➜  k get svc -n alpha
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
mysql         ClusterIP   10.43.8.147    <none>        3306/TCP         3m41s
web-service   NodePort    10.43.61.170   <none>        8080:30081/TCP   3m41s

controlplane ~ ➜  k get ep -n alpha
NAME          ENDPOINTS         AGE
mysql         10.22.0.9:3306    8m20s
web-service   10.22.0.10:8080   8m20s
```

`svc` 항목의 name이 이상함을 확인할 수 있다. 
아키텍쳐에서는 MySQL의 svc의 이름이 `mysql-service`로 되어 있지만, 실제 `get svc`에서 나오는 항목에 대해서는 svc명이 `mysql`로 되어 있다. 

**yaml 파일을 수정**

```yaml
controlplane ~ ✖ k get svc mysql -n alpha -o yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2025-06-03T07:46:29Z"
  name: mysql # <- 여기가 이상하다. mysql-service로 변경이 필요해보임
  namespace: alpha
```

**yaml을 수정하고 재배포 하니 문제가 해결됨**

```yaml
controlplane ~ ➜  k delete svc mysql -n alpha
service "mysql" deleted

controlplane ~ ➜  k apply -f mysql-service.yml 
service/mysql-service created
```

## Case 1-2. svc의 TargetPort가 잘못됨

**svc의 target port를 확인**

```yaml
controlplane ~ ➜  k describe svc mysql-service -n beta
Name:                     mysql-service
Namespace:                beta
Labels:                   <none>
Annotations:              <none>
Selector:                 name=mysql
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.43.97.173
IPs:                      10.43.97.173
Port:                     <unset>  3306/TCP
TargetPort:               8080/TCP
Endpoints:                10.22.0.11:8080
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```

위의 이미지에서 `mysql-service` 의 pod는 mysql container로 국룰 port인 `3306`을 사용한다. 하지만 현재 `mysql-service`의 target port는 `8080`으로 잘못된 port가 설정되어 있다.

## Case 1-3. svc의 Endpoint가 없음

**mysql-service의 endpoint가 없는 것을 확인**

```yaml
controlplane ~ ➜  k describe svc mysql-service -n gamma
Name:                     mysql-service
Namespace:                gamma
Labels:                   <none>
Annotations:              <none>
Selector:                 name=sql00001
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.43.138.8
IPs:                      10.43.138.8
Port:                     <unset>  3306/TCP
TargetPort:               3306/TCP
Endpoints:                
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```

`Endpoint`를 비어져 있다는 것은 svc가 연결된 pod가 없다는 것을 의미한다. 그렇기에 본래 연결되어야할 `mysql` pod를 연결할 필요가 있다. 

**yaml을 수정하여 `selector`의 target을 적절한 이름으로 수정** 

```yaml
controlplane ~ ✖ k get svc mysql-service -n gamma -o yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2025-06-03T08:44:30Z"
  name: mysql-service
  namespace: gamma
  resourceVersion: "1039"
  uid: 1c098933-389e-4905-85cd-74fd44b49363
spec:
  clusterIP: 10.43.138.8
  clusterIPs:
  - 10.43.138.8
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 3306
    protocol: TCP
    targetPort: 3306
  selector:
    name: sql00001 # 이거를 본래 연결되어여할 pod의 이름인 mysql로 변경
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

## Case 2

### Case2-1. Cluster가 정상적으로 작동하지 않음

```yaml
controlplane ~ ➜  k get deploy
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
app    0/1     1            0   
```

`app` deploy가 정상적으로 작동되지 않는다. pod나 deploy 모두 `describe`로 검사를 하였지만 별도의 문제는 발견되지 않는다. pod의 `log` 를 확인해도 log가 떨어지지 않은 상태이다. 

`kube-system`을 확인하여 cluster의 상태를 확인한다.

```yaml
controlplane ~ ✖ k get pods -n kube-system
NAME                                   READY   STATUS             RESTARTS      AGE
coredns-7484cd47db-hzkmf               1/1     Running            0             5m33s
coredns-7484cd47db-kbgwt               1/1     Running            0             5m33s
etcd-controlplane                      1/1     Running            0             5m41s
kube-apiserver-controlplane            1/1     Running            0             5m41s
kube-controller-manager-controlplane   1/1     Running            0             5m41s
kube-proxy-qwz8v                       1/1     Running            0             5m33s
kube-scheduler-controlplane            0/1     CrashLoopBackOff   4 (23s ago)   2m4s

controlplane ~ ✖ k get pods -n kube-system
NAME                                   READY   STATUS             RESTARTS      AGE
coredns-7484cd47db-hzkmf               1/1     Running            0             5m33s
coredns-7484cd47db-kbgwt               1/1     Running            0             5m33s
etcd-controlplane                      1/1     Running            0             5m41s
kube-apiserver-controlplane            1/1     Running            0             5m41s
kube-controller-manager-controlplane   1/1     Running            0             5m41s
kube-proxy-qwz8v                       1/1     Running            0             5m33s
kube-scheduler-controlplane            0/1     CrashLoopBackOff   4 (23s ago)   2m4s
...
Events:
  Type     Reason   Age                   From     Message
  ----     ------   ----                  ----     -------
  Normal   Pulled   45s (x5 over 2m25s)   kubelet  Container image "registry.k8s.io/kube-scheduler:v1.32.0" already present on machine
  Normal   Created  45s (x5 over 2m25s)   kubelet  Created container: kube-scheduler
  Warning  Failed   44s (x5 over 2m24s)   kubelet  Error: failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "kube-schedulerrrr": executable file not found in $PATH: unknown
  Warning  BackOff  20s (x20 over 2m22s)  kubelet  Back-off restarting failed container kube-scheduler in pod kube-scheduler-controlplane_kube-system(e62b9c3ab48bb2ebb392179898b5495e)
```

`kube-scheduler-controlplane`의 상태가 정상적이지 않다. `describe`를 통해 확인 결과 `exec` 명령이 잘못 설정되어짐을 확인할 수 있다.

스케줄러는 `kubelet`이 관리하는 pod마다 있는 **static pod**로 재배포하기 위해서는 별도의 `yaml`을 배포하는 것이 아닌 특정한 경로(`/etc/kubernetes/manifest`)에 있는 yaml을 수정하면 kubelet이 이를 감지하여 자동으로 재배포한다. 

```yaml
controlplane /etc/kubernetes/manifests ➜  vi kube-scheduler.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-scheduler
    tier: control-plane
  name: kube-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-schedulererrr -> kube-scheduler 로 수정
    
controlplane /etc/kubernetes/manifests ➜  k get pod -n kube-system
NAME                                   READY   STATUS    RESTARTS      AGE
coredns-7484cd47db-hzkmf               1/1     Running   0             14m
coredns-7484cd47db-kbgwt               1/1     Running   0             14m
etcd-controlplane                      1/1     Running   0             14m
kube-apiserver-controlplane            1/1     Running   0             14m
kube-controller-manager-controlplane   1/1     Running   0             14m
kube-proxy-qwz8v                       1/1     Running   0             14m
kube-scheduler-controlplane            1/1     Running   1 (22s ago)   53s
```

스케줄러가 정상적으로 가동되어진다.

### Case 2-2. deployment가 정상적으로 작동하지 않음 (scailing 문제)

```yaml
controlplane /etc/kubernetes/manifests ➜  k get deploy
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
app    1/2     1            1           12m

controlplane /etc/kubernetes/manifests ➜  k get rs
NAME             DESIRED   CURRENT   READY   AGE
app-7f9667c9d9   1         1         1       12m
```

app의 deployment는 두 개의 pod를 배포할려고 하지만 이 영향이 하위 리소스인 replicaset에 적절하게 반영되지 않은 현상이 발생했다. 마찬가지로 `kube-system`의 원인으로 추정하고 상태를 확인한다.

```yaml
controlplane /etc/kubernetes/manifests ➜  k get pods -n kube-system
NAME                                   READY   STATUS             RESTARTS      AGE
coredns-7484cd47db-hzkmf               1/1     Running            0             16m
coredns-7484cd47db-kbgwt               1/1     Running            0             16m
etcd-controlplane                      1/1     Running            0             16m
kube-apiserver-controlplane            1/1     Running            0             16m
kube-controller-manager-controlplane   0/1     CrashLoopBackOff   4 (42s ago)   2m15s
kube-proxy-qwz8v                       1/1     Running            0             16m
kube-scheduler-controlplane            1/1     Running            2 (65s ago)   3m34s
```

```yaml
controlplane /etc/kubernetes/manifests ➜  k describe pod kube-controller-manager-controlplane -n kube-system
Name:                 kube-controller-manager-controlplane
Namespace:            kube-system
Priority:             2000001000
...
--controllers=*,bootstrapsigner,tokencleaner
--kubeconfig=/etc/kubernetes/controller-manager-XXXX.conf
--leader-elect=true

controlplane /etc/kubernetes/manifests ➜  vi kube-controller-manager.yaml 
    - --cluster-name=kubernetes
    - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
    - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
    - --controllers=*,bootstrapsigner,tokencleaner
    - --kubeconfig=/etc/kubernetes/controller-manager.conf <- 수정함
```

`kube-controller-manager`의 설정 파일의 이름이 `--kubeconfig=/etc/kubernetes/controller-manager-XXXX.conf`로 되어 있다. 뒤의 `XXXX`를 제거하여 수정 후 이를 `kubelet`이 자동으로 감지하여 재배포하도록 한다.

**잘못된 `k8s-certs`를 올바른 path로 변경하기**
```yaml
  k8s-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/WRONG-PKI-DIRECTORY
    HostPathType:  DirectoryOrCreate
    
    
    k8s-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki
    HostPathType:  DirectoryOrCreate
  kubeconfig:
```

정상적으로 `kube-system`의 pod들이 가동되고 deploy 또한 적절하게 scaling이 되어졌다.
```yaml
controlplane /etc/kubernetes/manifests ➜  k get pods -n kube-system
NAME                                   READY   STATUS             RESTARTS      AGE
coredns-7484cd47db-hzkmf               1/1     Running            0             16m
coredns-7484cd47db-kbgwt               1/1     Running            0             16m
etcd-controlplane                      1/1     Running            0             16m
kube-apiserver-controlplane            1/1     Running            0             16m
kube-controller-manager-controlplane   0/1     CrashLoopBackOff   4 (42s ago)   2m15s
kube-proxy-qwz8v                       1/1     Running            0             16m
kube-scheduler-controlplane            1/1     Running            2 (65s ago)   3m34s
```