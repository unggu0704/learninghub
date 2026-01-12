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

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fw0iYZ%2FbtsdzBTv9Id%2FkWABhIvQraporzX256wWYK%2Fimg.png)

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
    
서버는 표준이 XFF 헤더에서 추출하였기에 실제 Client IP를 기대했지만
실제로는 Application Gateway의 IP 대역(Proxy)이 확인됩니다.


## 🤔 왜 이런 현상이 발생할까?

### Application Gateway 동작

Microsoft 공식 문서에 따르면

Application Gateway는 자신이 바라본 Client의 TCP IP를 자동으로 X-Forwarded-For 헤더에 올바르게 삽입합니다.
(별도의 Rewrite Rule 불필요)

> [how-application-gateway-works?](https://learn.microsoft.com/en-us/azure/application-gateway/how-application-gateway-works)


###  WAS에 도달한 Header 확인

WAS에 모든 Header를 추가하는 로직을 추가한 뒤 확인한 결과는 다음과 같습니다.

```
    X-Original-Forwarded-For: {Client IP}:{Port} <- 이건 뭐지?
    X-Real-IP: {AppGW IP}
    X-Forwarded-For: {AppGW IP}
```

로그 내용을 정리하면 아래와 같습니다.

- 실제 Client IP는 **X-Original-Forwarded-For** 에 존재
- X-Forwarded-For 는 **AppGW IP**로 변경됨

AppGW가 XFF를 보냈지만
WAS에 도달하기 전에 값이 변경되었음을 알 수 있습니다.

### Nginx Ingress Controller의 동작

현재 아키텍쳐를 다시 보면 

![image.png]({{ site.baseurl }}{{ page.url }}/img/azure4042.png)

AppGW와 Pod 사이 Ingress(Nginx)가 존재함을 확인할 수 있습니다. 

그리고 결국 이 친구가 XFF헤더를 일부 조작하는 것을 파악하였습니다.

> [notes-from-the-field-ingress-controller-troubleshooting-of-x-forwarded-for-header](https://techcommunity.microsoft.com/blog/azurestackblog/notes-from-the-field-ingress-controller-troubleshooting-of-x-forwarded-for-heade/3753946)

> [ngress-nginx/issues/5970](https://github.com/kubernetes/ingress-nginx/issues/5970#issuecomment-879855750)


## Client IP 흐름 정리

### 1. Application Gateway

Client 요청 수신 시 XFF 헤더에 정상적으로 Client IP를 삽입해서 뒤로 넘깁니다. (Ingress에게...)
```
X-Forwarded-For: {Client IP}
```

### 2. Nginx Ingress Controller

Nginx 입장에서는 외부에서 전달된 XFF를 신뢰하지 않습니다.
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

예를 들어 모니터링 툴에서 XFF 헤더를 가져가서 ClientIP로 인지 해버릴수가 있습니다.

만약 일반적인 Nginx였다면, 신뢰할 수 있는 IP대역 설정이 가능하지만 

Azure의 App-rotung Nginx에서는 이러한 설정은 제힌되어 있습니다.

그래서 우선 서비스 내 Ingress의 어노테이션을 활용해 이 설정을 오버라이딩 시도해보겠습니다.

**proxy-set-headers를 사용하여 헤더 rewrite**

```
metadata:
  annotations:
    nginx.ingress.kubernetes.io/proxy-set-headers: "X-Forwarded-For $http_x_forwarded_for";
```

위 설정을 할 때 Azure Nginx는 `{`, `}`와 같은 금지된 문법은 Invaild syntax가 발생하기에 주의해야합니다.

**Invaild Syntax 문법을 사용한 예**
```
metadata:
  annotations:
    nginx.ingress.kubernetes.io/proxy-set-headers: |
      if ($http_x_proxied_peer != "") {
        more_set_headers "X-Forwarded-For: $http_x_proxied_peer";
      }
```
해당 설정을 통해 문제가 되었던 자세한 이야기는 아래 글에서 확인이 가능합니다.

 > [Azure환경에서 Ingress 라우팅 실패로 404 발생한 이야기](https://unggu.dev/lessons-learned/devops/Azure-Ingress-%EC%9D%B4%EC%8A%88/)

서비스 내 Ingress에 적용하고 상위 Ingress Controller의 Nginx Pod내 `nginx.conf`를 확인하여 설정이 잘 올라갔는지 체크합니다.

**nginx Pod내 `nginx.conf` 파일**
```

# Pass the original X-Forwarded-For
proxy_set_header X-Original-Forwarded-For  $http_x_forwarded_for;
# Pass the original X-Forwarded-Host
proxy_set_header X-Original-Forwarded-Host $http_x_forwarded_host;

# Custom headers to proxied server 

# Ingress 어노테이션을 통해 xff 헤더를 원본으로 지정 (추가 설정)
proxy_set_headers "X-Forwarded-For $http_x_forwarded_for";

```
**결과 (실패)**
```
클라이언트 IP: {appgw IP}
```

정상적으로 `nginx.conf` 에 반영되었지만 여전히 xff 헤더에 대해 원하는 결과를 얻지 못하였습니다.

**테스트**

Ingress Annotaion을 통한 `Proxy_set_header`의 설정 자체가 정상적으로 작동하는지 의심스러웠고 이를 테스트 하기 위해 아래와 같은 테스트를 시도했습니다.

1. AppGW에서 `test`라는 이름의 Custom Header를 생성
2. 서비스 내 Ingress의 `proxy_set_header`를 통해 해당 Custom Header를 변조 시도
3. Was Log로 확인 변조를 확인


**AppGW내 Custom Header 추가**

![image.png]({{ site.baseurl }}{{ page.url }}/img/image.png)_test라는 헤더를 `1.1.1.1`로 설정_

**Ingress 어노테이션 추가**
```
metadata:
  annotations:
 nginx.ingress.kubernetes.io/configuration-snippet: |
   # 테스트 1: 고정값으로 변경
   proxy_set_header test-fixed "nginx-changed";
   
   # 테스트 2: $remote_addr로 변경
   proxy_set_header test-remote $remote_addr;
   
   # 테스트 3: AppGW가 보낸 test 헤더값 그대로 재전송
   proxy_set_header test-original $http_test;
   
   # 테스트 4: X-Forwarded-For 변경
   proxy_set_header X-Forwarded-For $http_x_forwarded_for;
   
   # 테스트 5: 원본 test 헤더 유지하면서 추가
   proxy_set_header test-append "$http_test-and-more";

 ```
해당 설정 후 Nginx reload 후 WAS log 확인 결과입니다.

```
`10:28:28,926 INFO [stdout] (default task-7) === All Headers ===`
`10:28:28,926 INFO [stdout] (default task-7) test-remote: 10.241.166.132`
`10:28:28,926 INFO [stdout] (default task-7) test-append: 1.1.1.1-and-more`
`10:28:28,926 INFO [stdout] (default task-7) test-original: 1.1.1.1`
`10:28:28,926 INFO [stdout] (default task-7) test-fixed: nginx-changed`
`10:28:28,926 INFO [stdout] (default task-7) test: 1.1.1.1`
`10:28:28,926 INFO [stdout] (default task-7) X-Forwarded-For: {AppGW IP}
```
정상적으로 `proxy_set`을 통해 커스텀 헤더는 조작을 확인하였습니다.

다만, 정작 중요한 XFF 헤더는 여전히 조작할 수 없었습니다.

아마 `lua` 등의 스크립트에서 더 강력한 제어를 통해 위변조를 막은것으로 추정하며 다른 방식이 필요할듯 합니다.

### 방법1. 믿을 수 있는 header 설정하기 (use-forwarded-headers) (실패)

`nginx.ingress.kubernetes.io/use-forwarded-headers: "true"` 어노테이션을 적용했으나, 이 설정은 이전 프록시의 X-Forwarded-For 헤더를 신뢰하여 전달하는 기능만 제공합니다.

실제 클라이언트 IP를 정확히 추출하려면 `proxy-real-ip-cidr`을 통해 신뢰할 수 있는 프록시 IP 대역을 지정해야 하지만, Azure 관리형 Nginx는 ConfigMap 수정을 지원하지 않아 이 설정이 불가능합니다.


### 방법2. Java Agent 오버라이딩 (실패)

현재 Applcation Insgiht(Azure 모니터)는 Agent Jar 형태로 WAS가 빌드 되어질 때, 함께 기동하는 식으로 구성되어 있습니다. 

이 Jar를 오버라이딩 하는 방안을 생각해보았지만 아래 공식문서에서 Agent 3.x 버전부터는 `TelemetryInitializer`를 사용하지 않고 자동화된 방안으로 데이터를 수집한다고 나와있어 불가능 하다고합니다. 
> [java-standalone-upgrade-from-2x](https://learn.microsoft.com/en-us/azure/azure-monitor/app/java-standalone-upgrade-from-2x)

### 우회 방안

XFF헤더는 Nginx가 강력하게 제어하고 Java Agent 설정도 막혀있음을 확인했습니다. 

하지만 다른 Custom 헤더는 느슨한 방식으로 제어함도 알았습니다. 

그렇기에 실제 Client IP를 표준 XFF헤더가 아닌 별도의 헤더를 통해 뒤로 넘겨 이를 Azure 모니터가 Custom 헤더로 수집하도록 설정을 할 수 있습니다. (별도의 컬럼으로 저장됨)

```
{
  "connectionString": "...",
  "preview": {
    "captureHttpServerHeaders": {
      "requestHeaders": [
        "X-Original-Forwarded-For"
      ]
    }
  }
}
```

이를 KQL 쿼리 등에서 아래와 같이 조회하면 ClientIP와 같이 조회가 가능합니다.
```
requests
| extend realClientIp = tostring(customDimensions["http.request.header.x_original_forwarded_for"])
| project timestamp, client_Ip, realClientIp
```

이를 통해 모니터링 툴에서 여전히 AppGW IP(잘못된 IP)로 보이지만, 커스텀 헤더를 추가 조회함으로써 실제 Client IP를 추가 조회는 가능합니다.

현재로써는 이러한 방법이 최선이라고 생각되지만, 추후 다른 방법 또는 Azure의 업데이트를 통해 더 나은 방향으로 개선됨을 기대합니다.

### 번외. 다른 모니터링 툴은?

**Datadog** 

환경변수 `DD_TRACE_CLIENT_IP_HEADER`로 커스텀 헤더 직접 지정 가능. 

**WhaTap** 

설정 옵션 `trace_http_client_ip_header_key`로 커스텀 헤더 직접 지정 가능. 

**Elastic APM** 

커스텀 헤더 지원 안 함. Forwarded, X-Real-IP, X-Forwarded-For 3개만 하드코딩되어 있고 추가/변경 불가.

**Application Insights** 

커스텀 헤더 지원 안 함. X-Forwarded-For만 사용하며 마지막 IP만 수집. Java Agent에서 헤더 선택 불가.

**Jennifer** 

WAS 레벨에서 설정된 헤더를 바라보는것으로 추정


© 2026 unggu. All rights reserved.



