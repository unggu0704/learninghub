---
author: "unggu"
title: "[AZ-204] Azure App Service"
date: 2025-01-02 22:33:43 +0800
categories: [Azure, Azure 204]
tags: [Azure, AKS, AZ204]
render_with_liquid: true
comments: true
image:
  path: assets/img/metaimg/azure204/App-Service.png
---
### 아키텍쳐

![스크린샷 2024-10-14 오후 2.31.32.png]({{ site.baseurl }}{{ page.url }}/img/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2024-10-14_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_2.31.32.png)_일반적인 Azure App Service의 구조_

### Azure App Service란

> *Azure App Service란 web, Rest API 및 백엔드 호출을 위한 HTTP 기반 서비스 (PaaS)*
> 

**자동 적인 크기 조정 지원**

Azure App Service는 자동적으로 스케일을 조정해주는데 이러한 리소스에는 가능한 코어 수와 RAM크기가 포함(스케일 업/다운), 머신의 인스턴스 수를 조절(스케일 인/아웃) 기능이 포함되어 있다. 

### 가격 계층 구조

- **Standard**: 자동 확장, 로드 밸런싱, 스테이징 슬롯 지원
- **Premium**: 고성능 VM, 30개의 인스턴스, 높은 비용
- **Isolated**: VNet의 격리된 환경, 100개 인스턴스, 매우 높은 비용 
- **Premium V3**: 기존의 Premium 보다 가성비가 훌륭함 

**컨테이너 및 CI/CD 지원**

Azure App Service는 컨테이너화 된 앱을 ACR 또는 Docker Hub등에서 가져와 배포하고 실행 할 수 있다. CI/CD에서는 Gitun와 Azure Devops 같은 곳과 연결하여 배포를 진행한다. 

또한 배포 할때  프로덕선 슬롯과 배포 슬롯을 지원하여 두 배포 슬롯간의 교환이 가능하다.

> ***배포 슬롯이란?**
일반적인 (PRD/DEV) 환경을 나누는 것으로 **Production Slot**과 **Staging Slot**으로 나눈다. 
이를 통해 Prodcution 환경에서 문제 발생시 롤백 전략을 사용할 수 있게 된다..*
> 

### Azure App Service의 구조

![스크린샷 2024-10-14 오후 2.35.00.png]({{ site.baseurl }}{{ page.url }}/img/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2024-10-14_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_2.35.00.png)
_Azure App Service의 구조_

Azure App Service는 PaaS형 분산 구조로 하나의 계획에 다양한 서비스(Web, Mobile, API)를 제공할 수 있다.

일반적으로 들어오는 HTTPS 요청을 처리하는 **프론트엔드**와 요청에 맞추어 실제 코드를 실행하는 **작업자**로 구분 되어 진다.  또한 자체적인 **파일 시스템**을 구축하여 지속성 있는 저장소(AP의 코드, 설정, 로그)를 제공한다. 

### **App Service의 인증 및 권한**

Azure App Service는 보안을 위해 다양한 인증 및 권한 부여를 기본적으로 제공한다. 

이러한 기능은 최소 코드 또는 Azure Functions을 사용하거나 또는 전혀 코드를 작성 하지 않고 또는 수정을 최소화 하며 데이터에 엑세스 할 수 있게 한다. 

또한 Federation ID 기능을 지원하여 다양한 ID 공급자를 사용가능하다.

**지원 하는 ID 공급자**

- MS Entra ID
- Facebook
- Google
- X
- Github
- 기타 등등…

> **Federation ID란?**
*서*로 다른 *도메인간 신뢰 관계를 설정하여 동일한 ID로 접근할 수 있게 하는 ID 
ID공급자와 SP공급자로 나뉘며 일반적으로 SSO를 지원한다. 
사용되는 기술 : `OAuth`, `OIDC`, `WS-Federation`*
> 

### 인증의 흐름

일반적으로 ID 공급자의 SDK를 활용할 수 있다. 
이런 경우 사용자는 공급자의 SDK에서 인증을 하고 유효성 검사를 위해 인증토큰을 App Service에 제출하는 방식이다. 이러한 방법을 *클라이언트 흐름*이라고 한다.
이러한 방식은 고객이 한번 앱을 종료하더라도 다시 인증상태가 유지되는 경험을 가질 수 있다.

만약 공급자의 SDK를 활용하지 않는다면 클라이언트를  `/.auth/login/<provider>` 로 리다이렉트 시켜 고객에게 인증을 요구한다. 인증이 성공적이라면 인증 쿠키를 사용해 클라이언트는 다시 원래 페이지로 리다이렉트 된다. 이러한 방법은 다시 로그인할 필요 없이 모든 페이지에 접근이 가능하다.

### App Service 네트워크 기능

App Service의 네트워크 기능은 Inboud 트래픽과 Outbound 트래픽을 제어하거나 관리할 수 있게 도와준다. 

**Inboud 트래픽 관리**

앱에 고유한 IP주소를 할당해 IP 기반 SSL을 가능하게 한다. 또한 특정 IP 범위에서만 앱에 접근할 수 있도록 제한을 두기도 하며, PE(Private Endpoint)를 사용하게 지원한다. 

**Outbound 트래픽 관리**

온프레미스 환경 통합(사내 시스템)으로 연동을 가능하게 하며, VNet을 통해 다른 리소스(*ex) 데이터베이스)*와 통신을 제공한다.
여기서 사용되는 Outbound 주소는 `possibleOutboundIpAddresses` 에서 선택되며 Azure Portal 또는 CLI를 통해 확인이 가능하다.

### Azure WebJobs

Azure App Service에서 백그라운드 작업을 실행하는데 사용되는 기능으로 AP와 통합된 백그라운드 처리를 쉽게 구현할 수 있다. 
Azure Storage와 Service Bus등과 트리거 기반 기능 설정 및 연속적 실행이 가능하다.