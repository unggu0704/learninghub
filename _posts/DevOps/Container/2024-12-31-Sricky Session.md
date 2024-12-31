---
author: "unggu"
title: "Sticky Session이란?"
date: 2024-12-31 16:15:02 +0800
categories: [DevOps, Container]
tags: [Container, Docker, Kubernetes, Cloud, DevOps]
render_with_liquid: true
comments: true
image:
  path: https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQOCrvcXJF6MW0GXDdW_aR54l4LToSk3B2DIg&s
---

![image](https://smjeon.dev/assets/img/loadbalancing/before-lb.png)_Load Balancer를 통해 여러대의 pod(서버)로 요청을 분리_

Load Balancer를 통해 서버의 부담을 줄이고 각 요청이 Round Robin 방식으로 나누어진다... _일반적으로_
하지만 이런 대규모 트래픽이 발생하는 서비스에 있어 이러한 방식에는 치명적인 단점이 있는데... 

**세션 유실**
![image](https://smjeon.dev/assets/img/loadbalancing/load-balancing-effect.gif)_내 Session이 어디갔지?_
사용자는 **1번 서버**로 Login 요청을 보낸다.. **1번 서버**는이 요청에 대한 True 값을 가지고 있는 상태 
곧이어 사용자는 유저 정보 조회하는 요청을 보낸다. 이때 보내는 서버는 **2번 서버** _(Load Balancer는 라운드 로빈 상태)_ **2번 서버**는 사용자가 보낸 요청에 대해 로그인 정보가 존재하지 않기 때문에 다시 **사용자를 Login 페이지로 redirect 시킨다!!**

> *이러한 현상은 일반적으로 AP 문제인지 Infra 문제인지 디버깅이 힘들다...*

### 해결법 - Sticky Session

![image](https://smjeon.dev/assets/img/loadbalancing/sticky-session.gif)_Sticky Session을 적용한 구조_
사용자가 **1번 서버**로 요청을 보냈다. 이렇게 된다면 사용자가 보낸 요청은 앞으로 **1번 서버**에 고정이 되는 방식 -> **사용자의 로그인 정보는 지속적으로 남게 된다!!**

### 이러한 방법은 어떻게 구현되는걸까?

이러한 방식은 Cookie나  Redis를 사용하여 IP Tracking 하는 방식이 있다. 컨테이너 환경에서는 대게 **Ingress Controller** 또는 **Session Affinity**를 사용!!

**Ingress Controller**를 사용하여 Sticky Session을 구현
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/affinity: "cookie"
    nginx.ingress.kubernetes.io/session-cookie-hash: "sha1"
    nginx.ingress.kubernetes.io/session-cookie-name: "route"
spec:
  rules:
  - host: test.s
```
_출저 : https://rinko.tistory.com/13_

**Session Affinity**를 사용하여 Sticky Session을 구현
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
  labels:
    app: my-app
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
  sessionAffinity: ClientIP # sessionAFfinity 설정 부분
```
SessionAFfinity를 ClientIP로 설정하여 `EndPoint`에서 확인한 IP 주소를 기반으로 pod이 할당되어 요청을 처리시킨다.
