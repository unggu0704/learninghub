---
title: Azure Load Balancer와 Ingress
author: "unggu"
date: 2024-12-17 21:13:04 +0800
categories: [Azure, Azure 공부]
tags: [Azure, AKS, Kubernets, Container, DevOps]
render_with_liquid: true
comments: true
image:
  path: https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTBbsmfOGVH_DT-XFBw1YRfhOPh05tA4dZyCg&s
---



## Load Balancer & Ingress


## Load Balancer
![image]({{ site.baseurl }}{{ page.url }}/image/Pasted image 20241211144010.png)

- Azure의 리소스 간 트래픽을 분산하여 높은 가용성과 분산 기능을 제공
    - 대부분의 효과는 일반적인 Load Balancer와 동일
- 주로 두가지 유형으로 이루어진다.
    - **공용 Load Balancer**: 인터넷에서 들어오는 트래픽을 백엔드 풀의 리소스에 분산
    - **내부 Load Balancer**: 가상 네트워크(VNet) 내부에서 트래픽을 분산

### 비용

**공용 Load Balancer**

- **인바운드 트래픽**: 인터넷에서 Azure Load Balancer로 들어오는 트래픽은 무료
- **아웃바운드 트래픽**: Azure Load Balancer에서 인터넷으로 나가는 트래픽에는 비용이 발생

**내부 Load Balancer**

- **내부 트래픽**: VNet 내에서 Load Balancer를 통해 이동하는 트래픽에도 비용이 발생

### L4 LoadBalancer VS L7 LoadBalncer

**L4 로드밸런서**는 일반적으로 TCP와 UDP기반으로 트래픽을 분산시킨다.
- 주로 IP주소와 Port 로드밸런싱 -> 빠른 속도를 보장하지만 낮은 유연성
**L7 로드밸런서**는 HTTP 및 HTTPS 프로토콜 기반으로 서버의 트래픽을 분산 
- 주로 URL, 헤더, 쿠키 로드밸런싱 -> 늦은 속도를 제공하지만 높은 유연성

| 항목        | L4 로드 밸런서      | L7 로드 밸런서            |
| --------- | -------------- | -------------------- |
| 작동 계층     | 전송 계층(Layer 4) | 애플리케이션 계층(Layer 7)   |
| 주요 프로토콜   | TCP, UDP       | HTTP, HTTPS          |
| 로드 밸런싱 기준 | IP 주소, 포트      | 요청 내용(URL, 헤더, 쿠키 등) |
| 처리 속도     | 상대적으로 빠름       | 상대적으로 느림             |
| 기능 및 유연성  | 상대적으로 제한적      | 다양한 기능 및 유연성         |

**Azure 환경에서는?**
Azure에서는 일반적으로 **Public Load Balancer**(인터넷 -> Azure)와 **Internel Load Balancer**(Azure VNet -> Azure VNet)에서 L4 로드밸런싱을 사용한다. 
반대로 **Azure Application Gateway**에서 L7 로드밸런싱을 사용하는데 URL 기반 라우팅과 쿠키기반 세션 지속성(Session Affinity), SSL 종료 기능에 사용된다. 


## Ingress
![image]({{ site.baseurl }}{{ page.url }}/image/Pasted image 20241211144121.png)
### Ingress

- 각각의 svc와 pod의 포트를 설정하지 않고 쿠버네티스 내부에서 **리버스 프록시 서버** 역할을 수행하는 기능 제공
- 쿠버네티스가 제공하는 **L7 로드밸런싱** 기능을 제공하는 컴포넌트
![image]({{ site.baseurl }}{{ page.url }}/image/Pasted image 20241211150205.png)

### Ingress Controller

- ingress 까지는 쿠버네티스 안에 있는 API이지만, 어떤 방식으로 작동할지는 Ingress Controller가 결정
    - Ingress의 리소스들을 규칙 적용 관리는 controller가 한다. 
	    - ex) https는 443 포트로 설정
- KT Azure에서는 **Nginx Ingress Controller**를 사용하는 것으로 보인다.


> **전체적인 흐름** _로컬PC -> 로드밸런서 -> Ingress class(AKS) -> Ingress(AKS) -> svc -> pod_
