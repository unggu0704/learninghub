---
title: Proxy 환경(AppGW, Nginx)에서 X-Forwarded-For 헤더 사용기
author: "unggu"
date: 2025-12-24 19:12:42 +0800
categories: [lessons-learned, DevOps]
tags: [Azure, AKS, Kubernets, Container, DevOps]
render_with_liquid: true
comments: true
image:
  path: assets\img\metaimg\xff.png
---

## 서비스에서 Client IP를 사용하는 이유

서비스에서 접속한 Client IP를 확인할 일은 많습니다.

- Admin 페이지의 IP 기반 접근 제어
- 고객 요청에 대한 감사 / 추적 로그

On-Premise 환경의 경우
사내 별도의 프록시 서버 없이 사내 L4로 고객 요청이 직접 유입되기 때문에
별도의 처리 없이 logic level에서 Client IP를 수집해도 문제가 없었습니다.

Java에서는 아래와 같은 방식이 대표적입니다.
```
request.getRemoteAddr();
```
이 코드를 통해 TCP 연결의 socket 정보에서 직접 접속한 IP를 추출합니다.


## Proxy 환경 (Azure AppGW <-> AKS)에서는 어떻게 될까?

Azure Application Gateway와 같은 Proxy 환경에서는 상황이 달라집니다.
```
String ipAddress = request.getRemoteAddr();
```
이 방식으로 IP를 추출하면 대부분의 경우
실제 Client IP가 아니라 마지막으로 거쳐온 Proxy(AppGW)의 IP가 반환됩니다.

그렇다면 이런 환경에서는 어떻게 Client IP를 식별해야 할까요?

## X-Forwarded-For (XFF) 헤더의 사용

프록시나 L7 Load Balancer 환경에서는
Client IP 식별을 위해 X-Forwarded-For(XFF) 헤더를 사용하는 것이 일반적입니다.

XFF 헤더는 Client → Proxy → Backend로 이어지는 체인에서
최초 Client IP를 전달하기 위한 사실상의 표준 헤더로 널리 사용됩니다.


## 서비스 적용 예시

아래와 같은 아키텍처를 가정합니다.

![image.png]({{ site.baseurl }}{{ page.url }}/img/azure4042.png)


Azure Application Gateway 뒤에 있는 WAS에서
XFF 헤더를 통해 Client IP를 수집할 수 있을 것으로 기대하며 코드를 작성했다고 가정합니다.
```
String xff = request.getHeader("X-Forwarded-For");
String clientIp = xff.split(",")[0].trim();
System.out.println("클라이언트 IP: " + clientIp);
```
하지만 서버 로그를 확인하면 다음과 같은 결과가 출력됩니다.
```
클라이언트 IP: {AppGW IP} 
```
    
서버는 Client IP를 기대했지만
실제로는 Application Gateway의 IP 대역만 확인됩니다.


## 🤔 왜 이런 현상이 발생할까?

### Application Gateway 동작

Microsoft 공식 문서에 따르면

Application Gateway는 자신이 바라본 Client의 TCP IP를 자동으로 X-Forwarded-For 헤더에 삽입합니다.
(별도의 Rewrite Rule 불필요)

> https://learn.microsoft.com/en-us/azure/application-gateway/how-application-gateway-works


###  WAS에 도달한 Header 확인

WAS에 모든 Header를 추가하는 로직을 추가한 뒤 확인한 결과는 다음과 같습니다.

```
    X-Original-Forwarded-For: {Client IP}:{Port}
    X-Real-IP: {AppGW IP}
    X-Forwarded-For: {AppGW IP}
```

로그 내용을 정리하면 아래와 같습니다.
*
- 실제 Client IP는 **X-Original-Forwarded-For** 에 존재
- X-Forwarded-For 는 **AppGW IP**로 변경됨

AppGW가 XFF를 보냈지만
WAS에 도달하기 전에 값이 변경되었음을 알 수 있습니다.

### Nginx Ingress Controller의 동작

원인을 추적해보면 Nginx Ingress Controller가 XFF 헤더를 일부 조작하는걸 알 수 있습니다.

> https://techcommunity.microsoft.com/blog/azurestackblog/notes-from-the-field-ingress-controller-troubleshooting-of-x-forwarded-for-heade/3753946

> https://github.com/kubernetes/ingress-nginx/issues/5970#issuecomment-879855750


## Client IP 흐름 정리

### 1. Application Gateway

Client 요청 수신 시 XFF 헤더
```
X-Forwarded-For: {Client IP}
```

### 2. Nginx Ingress Controller

Nginx는 외부에서 전달된 XFF를 신뢰하지 않습니다.
(XFF는 조작 가능하다고 판단)

따라서 다음과 같은 처리를 수행합니다.

1. 기존 X-Forwarded-For → X-Original-Forwarded-For 로 이동
2. 자신이 직접 본 전송 IP(AppGW)를 새로운 X-Forwarded-For 로 설정

최종적으로 WAS에는 아래와 같은 헤더가 도달합니다.

    X-Forwarded-For: {AppGW IP}
    X-Original-Forwarded-For: {Client IP}

Nginx 입장에서

- 자신에게 접속한 IP = AppGW
- XFF는 위조 가능 → 신뢰 불가

이 동작은 보안 관점에서 충분히 합리적입니다.


## 코드 레벨에서의 해결 방법

이 구조에서 실제 Client IP를 확인하려면
X-Forwarded-For 가 아니라
X-Original-Forwarded-For 를 사용해야 합니다.
```
String xff = request.getHeader("X-Original-Forwarded-For");
String clientIp = xff.split(",")[0].trim();
System.out.println("클라이언트 IP: " + clientIp);
```


## 남는 문제: 외부 시스템은 어떻게 될까?

코드 단에서는 대응이 가능합니다.
하지만 다음과 같은 경우는 여전히 문제가 됩니다.

- 외부 모니터링 툴
- Azure Monitor / Application Insights
- 보안 분석 / SIEM 도구

이들이 X-Forwarded-For 헤더를 그대로 Client IP로 인식한다면

- 실제 Client IP 대신 AppGW IP를 Client IP로 오인
- 사용자 추적 / 감사 로그 왜곡
- 보안 분석, Rate Limit, Bot 탐지 오류 발생


