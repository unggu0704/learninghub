---
author: "unggu"
title: "[CKA] 쿠버네티스 핵심 리소스"
date: 2025-03-14 21:11:12 +0800
categories: [k8s, CKA]
tags: [CKA, k8s, Container, Docker]
render_with_liquid: true
comments: true
image:
  path: assets/img/metaimg/cka/cka.png
---

본 글은 [Udemy Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/?couponCode=KEEPLEARNING) 강의를 참조해 정리한 내용을 기록했습니다.


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