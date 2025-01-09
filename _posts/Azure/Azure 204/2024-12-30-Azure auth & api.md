---
author: "unggu"
title: "[AZ-204] Azure Aut & API Management"
date: 2024-12-30 17:05:13 +0800
categories: [Azure, Azure 204]
tags: [Azure, AKS, AZ204]
render_with_liquid: true
comments: true
image:
  path: assets/img/metaimg/azure204/featured-image.png
---

# Azure 사용자 인증

![image2](https://media.licdn.com/dms/image/D5612AQHrEDZaoNJ2jw/article-cover_image-shrink_600_2000/0/1713377589427?e=2147483647&v=beta&t=QYG2h1KPfj3SycX0pDNRSZ_uDVXSMFJ5YtnzgdbrDV4)
## Microsoft ID 플랫폼 탐색

서비스에서 사용되는 ID 및 엑세스 등의 기능을 Microsoft Entra ID로 위임하는 방식
개발자가 직접 ID 및 보안공간의 기술에 대해 고민하지 않고 Microsoft ID 플랫폼에 이를 위임하는 방식이다.
이는 OAuth2.0 권한 부여 프로토콜을 통해 구현되어 있다.

Azure portal에 앱을 등록하는 경우 이러한 기능을 사용하기 위해 두가지 테넌트 type이 존재한다.
- **단일 테넌트** : 해당 테넌트에만 엑세스 가능
- **다중 테넌트** : 다른 테넌트에도 엑세스 가능

### Application 개체 
Microsoft Entra ID 테넌트내에 생성되는 독특한 개체이다. 
이 Application 개체는 다른 테넌트를 관리하며 템플릿 역할도 제공한다. 그 외에 토큰 발급, 리소스 엑세스 등을 수행하여 전체적으로 Entra ID 환경 내의 핵심 구성 요소이다. 

### 서비스 주체 개체 
Entra ID로 보호되는 개체에 엑세스 하기 위해서는 엔티티가 보안 주체로 표시되어야한다. 이러한 보안 주체는 EntraID에서 정의 되는데 주로 **관리 ID**, **어플리케이션(테넌트 관리)**, **credentials(레거시)** 등이 사용된다.

### 권한의 유형
Microsoft ID에 권한을 위임하고 **위임된 엑세스**와 **앱 전용 엑세스 권한**이라는 두가지 형식을 지원하는데 **위임된 엑세스**는 사용자 또는 관리자가 앱을 실행하고 역할을 받는 방식이며, **앱 전용 엑세스 권한**은 사용자 없이 백그라운드에서 실행하는 옵션이다. 오직 관리자만 동의 할 수 있다.

> Q. Entra ID를 사용하여 인증을 부여할려고 한다. 만약 앱 등록이 제거된다면 사용자 정의 역할도 제거되어야 한다면 어떤식으로 하면 좋을까?

*정답: Application Manifest 같은 곳에 `appRoles` 속성을 설정하여 역할 정보를 정의한다. 이러한 정보는 App이 삭제될 때 같이 제거된다.*

--- 
## API Management

![image3](https://miro.medium.com/v2/resize:fit:600/0*GfcNbZljSEJYQ0Va.jpg)

API 게이트웨이, 관리 평면, 개발자 포털로 이루어져 하나이상의 API들로 구성된 완전 관리 API Management 프로그램

API Management는 시스템 그룹이 존재하는데 **관리자**, **개발자**, **게스트**로 이루어져 있으며 사용자는 추가적인 그룹을 만들 수 있다.

또한 API 요청에 있어 **정책**을 사용하여 순차석으로 실행되는 명령문의 컬렉션을 생성 가능하다. 
(*ex: XML -> JSON*) 또한 사용자의 요구에 맞추어 전역 API, 특정 API의 작업이 가능하다.

### API Gateway

![image](https://github.com/unggu0704/learninghub/raw/backup/cloud%20computing/azure/Azure-204/Azure%20Key%20Vault%20%26%20API/Azure%20Key%20Vault%20&%20API%20121b01a7807e802fb7eddb1ec057d5b4/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2024-10-16_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_2.00.05.png)

서비스에는 몇가지 엔드포인트가 존재한다. 하지만 이런 엔드포인트를 그대로 노출시키는 것은 코드의 복잡성과 추적성에 있어 비효율적이다. 
이를 해결하기위해 **API Management Gateway**로써 클라이언트와 서비스 사이에 배치되어 요청을 라우팅하는 *(역방향 프록시)* 로 사용된다. HTTPS 통신을 할때 클라이언트 인증서를 사용학거나 혼잡성 제어 기능도 제공한다. -> **디자인 패턴의 완성**


API Management는 관리형과 자체 호스팅 모두 지원하는데
**관리형**은 Azure에 배포된 기본 게이트웨이의 구성 요소로 관리형 게이트웨이를 허용하면 모든 API 트래픽이 Azure로 통과되는 형식이다.
**자체 호스팅**은 게이트웨이의 컨테이너화 버전으로 하이브리드 및 다중 클라우드 시나리오에 주로 사용된다.

### API 보호 
구독 Key나 인증된 CA를 통해 인증과정을 거쳐 API Magament를 보호 할 수 있다.
