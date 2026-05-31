---
author: "unggu"
title: "[CKA] 쿠버네티스 클러스터 및 핵심 리소스 개념"
date: 2025-02-25 21:11:12 +0800
categories: [k8s, CKA]
tags: [CKA, k8s, Container, Docker]
render_with_liquid: true
comments: true
image:
  path: assets/img/metaimg/cka/cka.png
---

본 글은 [Udemy Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/?couponCode=KEEPLEARNING) 강의를 참조해 정리한 내용을 기록했습니다.

## Kubernetes 클러스터 핵심 개념

## **Master와 Worker Node**

![image.png]({{ site.baseurl }}{{ page.url }}/img/image25.png)_쿠버네티스 아키텍쳐_

**Master Node**

Master는 **kube-api**, **kube-scheduler**(pod를 관찰하며 pod가 할당되기 좋은 노드를 결정), **ETCD** 클러스터와 컨트롤러로 이루어진다.

**Worker Node**

Docker로 만들어지며 이를 관리하는 **kubelet** 이는 **kebe-api**와 지속적으로 통신한다. 

실질적으로 *Node*를 관리하며 *pod*의 생성과 모니터링을 담당

### ETCD

> 분산되고 신용될 수 있는 클러스터를 위한  `Key-Value` 저장소
> 

![image.png]({{ site.baseurl }}{{ page.url }}/img/image%20125.png)_etcd의 위치_

쿠버네티스 클러스터에는 노드의 상태, pod의 정보가 영속적으로 관리 되어질 필요가 있다. 

이를 위해 ETCD는 쿠버네티스의 모든 운영 정보가 들어가 있으며, **master-node**에 `etcd pod` 이라는 *static*형태로 존재한다.

주로 **kube-api Server**와 상호작용하며 요청이 들어올 경우 변경사항을 검증하고 ETCD에 저장한다. 

또한 **kube-controller-manager**와 **kube-scheduler** 같은 서비스도 ETCD의 데이터를 기반으로 동작한다.

고가용성(HA)의 환경에서 여러개의 **Master** **Node**가 구성되어 있을 경우 **ETCD도 다중 노드로 구성**되어진다.

단, 별도의 ETCD 전용 노드를 사용해 독립적인 운영도 가능하다. 결과적으로 ETCD의 목적을 위한 데이터 정합성 유지가 핵심 

**ETCD가 저장하고 있는 데이터**는 Node, Pod, Configs, Secrets, Roles 등이 있다.

## kube-api

> 클러스터의 모든 오브젝트와 통신하며 Master Node ↔ 사용자를 소통하는 REST API
> 

**kube-api**는 클러스터의 중앙 제어 역할을 하는 컴포넌트로 *인증, API Layer, 요청 검증*등을 담당한다.

또한 **ETCD 클러스터**와 직접 통신하는 오브젝트

Master Node에서 동작하는 주요 관리 컴포턴트로 우리가 일반적으로 사용하는 `kubectl` 명령을 담당한다. input 하여 검증한 뒤 **ETCD 클러스터**로부터 데이터를 받아 `POST` 해준다. 

![image.png]({{ site.baseurl }}{{ page.url }}/img/image%20225.png)_kubectl get nodes의 처리 방식_

만약 사용자가 *pod*를 생성 요청한다면 아래와 같은 순서를 거친다. 

1. 요청을 한 유저를 인증 
2. 요청 검증
3. pod 생성 과정에 필요한 데이터를 조회 (*Namespace나 RBAC*)
4. **ETCD**에 *pod* 생성한 사용자 기록 
5. **Sceduler**에게 node가 할당되지 않은 *pod*이 있다는 것을 알려 적절한 *node*를 고르게 함 
6. **kubelet**을 통해 *pod*를 생성, 그리고 **kube-api**에게 정보를 업데이트 

![image.png]({{ site.baseurl }}{{ page.url }}/img/image%20325.png)_pod를 생성하는 처리 방식_

## kube controller manager

> k8s에서 여러 컨트롤러 프로세스를 관리하는 컴포넌트
> 

k8s는 여러 컨트롤러를 통해서 **ETCD**를 비롯한 k8s 리소스들의 상태를 확인 리소스 관리를 통해 원하는 상태(*desired state*)를 유지한다. 

### Node Controller

AP가 정상적으로 수행될 수 있도록 *Node* 상태를 모니터링 하고 조치가 필요하다면 **kube-api**를 통해 수행 할 수 있도록 한다. 

![image.png]({{ site.baseurl }}{{ page.url }}/img/image%20425.png)

- 5초마다 *Node*의 상태를 테스트한다.
- *Node*가 접근 할 수 없으면 40초간 기다린다.
- 접근 할 수 없음을 표시하고 5분간의 회복을 기다린다.

### Replication Controller

*Replica Set*의 상태를 모니터링하고 정해진 *pod*의 개수를 보장한다. 

![image.png]({{ site.baseurl }}{{ page.url }}/img/image%20525.png)

### more controller…

![image.png]({{ site.baseurl }}{{ page.url }}/img/image%20625.png)

이외에도 K8s에는 다양한 컨트롤러가 존재한다. 특정 시간 간격마다 반복 실행되는 job(*배치*)을 관리하는 **CronJob** 같은…

**그렇기에 Controller는 쿠버네티스의 두뇌 역할을 한다고 알려져있다.**

### kube-scheduler

> 어떤 pod이 어떤 Node로 가야하는가
> 

*Node*가 할당되지 않은 *pod*이 존재할 때, 해당 *pod*이 어떤 *Node*로 가야하는지에 대해 **결정만 한다.**

실제 작업은 **kubelet**이 수행한다. 

![image.png]({{ site.baseurl }}{{ page.url }}/img/image%20725.png)

예를 들어 Cpu:10이라는 요구사항에 있어 맞지 않은 노드를 필터링 하고 가능한 노드 가운데 우선순위를 결정한다. 이때 사용되는 우선 순위는 커스텀이 가능하다.

### kubelet

> **각 Node의 핵심 컴포넌트로 *kube-api*와 통신하며 *pod*을 관리**
> 

**kubelet**은 *pod*을 관리하며 **kube-api**에 *Node*와 *pod*의 상태를 보고한다.
또한 *Volume*과 *Network*를 설정하고 관리한다.

이런 좋은 **kubelet**은 자동으로 *Node*에 배포되지 않으며, 반드시 **수동으로 kubelet을 설치해줘야한다!**

### Kube Proxy

> Node에서 실행하는 Service와 Pod의 네트워크를 관리하는 Proxy*
> 

주로 `iptables` 를 사용해 트래픽을 관리하며 *Service* 기반 네트워크 규칙으로 올바르게 트래픽을 pod에게 전달한다. 

이는 마치 **Interface**의 역할과도 같아 *Node*나 다른 *Pod*으로 교체되어져도 같은 방식으로 통신을 가능하게 한다. 

### Pod란?

![image.png]({{ site.baseurl }}{{ page.url }}/img/pod3.png)_pod의 일반적인 구성_

하나의 *Node*에는 n개 이상의 *pod*이 존재한다. 이러한 *pod*안에는 사용자가 만든 *AP Container*와 Helper Conatiner라고 불린 두개의 Container가 존재한다. (*AP Container가 죽으면 같이 죽는다.)*

이러한 *pod*라는 개념을 통해 *Node*안에 여러 개의 AP Container를 운용할 때, 큰 도움을 주는데…

***만약 Pod라는 개념이 없다면?***
![image.png]({{ site.baseurl }}{{ page.url }}/img/pod4.png)_pod라는 개념이 없는 세계_

사용자는 컨테이너에 n개의 AP Container를 가진다면 n개의 *helper container*를 가져야한다. 여기에 만약 *volume*을 사용한다면 n개의 *volume*까지… 
운영자 입장에서는 이를 관리하기 위해 별도의 테이블을 만들어 AP Container가 죽는다면 그에 종속된 모든 리소스들을 관리해줘야한다. -> 피로도 증가.

이를 *pod*라는 하나의 단위로 묶음으로써 관리를 용이하게 한다. (Pod의 출현 배경)

### pod를 생성하는법

`yaml` 을 통해 k8s를 배포하는데 있어 root 레벨 속성들은 `apiVersion` , `kind`, `metadata` , `spec` 으로 이루어진다.

**`pod-definition.yml` 을 사용해 pod를 배포하기**

```yaml
apiVersion: v1 # version
kind: Pod # 어떤 형태를 배포할건지..?
metadata: # 이 pod의 데이터는?
	name : myapp-pod
	labels: # dic 형태
		app: myapp
		type: front-end

spec:
	containers: #List 형태
		- name: nginx-container
			image: nginx
```

그리고 `kubectl creage -f pod-definition.yml`를 실행한다.

---

아래부터는 Kubernetes에서 사용하는 핵심 리소스에 대해 알아보겠다.


## Replicaset

> Pod의 집합을 안정적으로 유지하고 동일 pod의 갯수에 대한 가용성을 보장해주는 쿠버네티스 리소스
> 

![image.png]({{ site.baseurl }}{{ page.url }}/img/rimage.png)

쿠버네티스에서 pod의 갯수를 유지(*replica*)해주는 컨트롤러의 역할을 담당한다. 그렇기에 pod이 비정상 종료될 경우 자동으로 재생성을 하며 **Label  Selector**를 사용해 관리할 pod을 선택한다.  

**ReplicaSet을 만드는 `yaml`**

```yaml
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: myapp-replicaset
      labels:
        app: myapp
        type: front-end
    spec:
     template:
        metadata:
          name: myapp-pod
          labels:
            app: myapp
            type: front-end
        spec:
         containers:
         - name: nginx-container
           image: nginx
     replicas: 3
     selector:
       matchLabels:
        type: front-end
```

**생성된 레플리카셋을 확인**

```jsx
kubectl get rs
```

## Deployment

> Pod와 레플리카셋에 대한 선언적 업데이트 및 가용성 조절하는 k8s 리소스
> 

![image.png]({{ site.baseurl }}{{ page.url }}/img/rimage%201.png)

하나의 Pod를 바라보는 Replica를 설정하고 이를 모아 Replica-Set를 만든다. 

이보다 상위 계층에 있는 것이 Deployment이다.

만약 pod을 업데이트 해야하는 상황에 있어 (Replica Set의 교체) 새로운 Replica Set을 만들어 pod을 배포하고 배포가 완료될 수록 기존의 replica set의 pod를 하나씩 죽여나가는 방식(Rolling Upate)을 지원한다. → **무중단 배포**

또한 기존의 Replica Set을 관리하며 언제는지 Rollback 할 수 있도록 지원한다.

*참고: deployment가 소유하는 replicaset은 관리해서는 안된다.*

**Deployment를 만드는 `yaml`**

```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: myapp-deployment
      labels:
        app: myapp
        type: front-end
    spec:
     template:
        metadata:
          name: myapp-pod
          labels:
            app: myapp
            type: front-end
        spec:
         containers:
         - name: nginx-container
           image: nginx
     replicas: 3
     selector:
       matchLabels:
        type: front-end
```

**배포된 deployment 확인**

```jsx
kubectl get deploy
```

## Service

> 다양한 역할을 하는 pod들을 통신하고 서비스를 제공하는 쿠버네티스 리소스
> 

![image.png]({{ site.baseurl }}{{ page.url }}/img/rimage%202.png)

### Use Case: pod에 접근하고 싶음

![image.png]({{ site.baseurl }}{{ page.url }}/img/rimage%203.png)

사용자는 Node안에 있는 pod에 접근을 하고 싶다고 가정하자. 이럴 경우 `curl` 을 통해 직접 `10.244.0.2` 로 접근하여 pod에 접근 할 수 있다.

하지만 이런 주소를 동적인 주소며 매번 curl을 사용하여 접근하는 것은 옳지 않은 방법

이런 이슈를 해결하기 위해 등장한 것이 **Service Type**이다.

### Service Types

![image.png]({{ site.baseurl }}{{ page.url }}/img/rimage%204.png)

**Node Port Service**

![image.png]({{ site.baseurl }}{{ page.url }}/img/rimage%205.png)

NodePort 30008에서 사용자의 요청이 들어온다면 이는 우선적으로 NodePort로 전달 되어진다. 그리고 Nodeport는 설정된 pod의 target port를 찾아 전달한다. 

### Cluster-IP

![image.png]({{ site.baseurl }}{{ page.url }}/img/rimage%206.png)

외부에서 접근할 수 없는 클러스터 내부에서 일괄적으로 사용되는 Inteface 형태로 통신을 진행한다. 

하나의 front-end pod이 요청을 처리하기 위해 `cluster-ip`로 설정된 back-end pod에 처리를 할 때 일괄적으로 사용되는 통신을 통해 처리를 간단하게 한다.