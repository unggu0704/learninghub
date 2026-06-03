---
title: "[Azure] Azure Entra ID"
author: "unggu"
date: 2026-05-29 19:12:42 +0800
categories: [Azure, Azure 공부]
tags: [Azure, AKS, EntraID, Key Vault]
render_with_liquid: true
comments: true
image:
  path: assets/img/metaimg/azure204/featured-image.png
---



> Azure 환경의 많은 인증은 Entra ID를 기반으로 이루어진다.

Azure 환경에서 모든 인프라에 대한 인증은 과거  Azure Active Directory라는 이름에서 2023년부로 Microsoft Entra ID로 통일 되었습니다. 

### SSO 인증

Azure Portal을 비롯한 MS 시스템에 로그인을 하게 되면 아래와 같은 인증 창을 볼 수 있습니다.

![image]({{ site.baseurl }}{{ page.url }}/image/entraid.png)_익숙한 MS로그인 화면_

모든 인증은 Entra ID와 통한다는 말과 비슷하게 이것 또한 Entra ID의 일부분인 SSO 인증의 일부로 Azure 관련 접근을 위해 사람(ID/Password)를 통해 인증을 요구합니다. 

```
az login
```

### App Registration

사용자가 SSO를 통해 외부에서 Azure 리소스에 접근하듯, 외부 서비스가 Azure 리소스에 접근이 필요할 떄가 있습니다. 

그럴 때 **App Registration**를 사용합니다. App Registration는 테넌트 단위의 객체이기에 같은 구독이 아닌 다른 구독 / 외부 서비스에서 `az login`을 직접 한 것과 같은 효과를 내며 접근이 가능합니다.

단, 이를 위해서 **Service Principal**라는 실제 권한이 할당되는 인스턴스가 필요합니다.

#### Service Principal

하나의 App Registration을 등록하면 테넌트 단위로 정의됩니다.  

이때 Service Principal는 멀티 테넌트 같은 환경이 아니라면 App Registration와 1대1로 매칭됩니다.

Service Principal의 주요 역할은 App Registration 내에 **실제로 권한이 부여**되는 인스턴스라고 볼 수 있습니다. 

만약 외부에서 구독 A의 Log Analytics의 Log를 검사하고 구독B의 Storage에 객체를 확인해야 한다고 가정하면 하나의 Service Principal에 아래처럼 권한(RBAC)을 직접 할당 합니다.
 
Service Principal에 할당된 권한 목록
- 구독 A의 Log Analytics Reader 역할 할당 
- 구독 B의 Storage Reader 역할 할당

만들어진 Service Principal은 아래 3개의 정보를 인증에 사용하고 해당 정보를 통해 소스코드 내 인증이 가능합니다.

```
credential = ClientSecretCredential( 
      tenant_id=..., 
      client_id=..., 
      client_secret=... 
)
```



- `AZURE_TENANT_ID` → "어느 Entra ID 테넌트냐" (KT 테넌트 전체에 1개, 고정값)
- `AZURE_CLIENT_ID` → "어떤 App Registration이냐" (App Registration 생성 시 자동 발급, 고정값)
- `AZURE_CLIENT_SECRET` → "비밀번호" (직접 생성, 만료기간 있음 / 1년·2년 등 설정 가능)

인증 흐름은 아래와 같습니다.

1. 코드 실행 (어디서든)
2. `login.microsoftonline.com/{tenant_id}/oauth2/token` 에 POST 요청
3. `client_id` + `client_secret`를 통한 검증
4. Bearer Token 발급 (1시간 유효)
5. Azure 리소스 (LAW, Storage 등) 접근

이 흐름에서 **호출 위치는 전혀 관계없습니다.** 외부 테넌트 AKS, 로컬 PC, 다른 클라우드 어디서든 동일하게 동작합니다.

### 관리 ID와의 비교

관리ID(Managed Identity)는 Azure 컴퓨팅 리소스(VM, AKS 등)에 직접 부여하는 신원으로, 테넌트 내 다른 Azure 리소스에 인증할 때 사용한다는 점이 Service Principal과 비슷합니다.

다만 아래와 같은 차이점이 존재합니다.

- **자격증명 관리**: Service Principal은 Client Secret을 직접 생성하고 만료를 관리해야 하지만, **Managed Identity는 Azure가 자동으로 관리**하며 Secret 자체가 존재하지 않습니다.
- **사용 범위**: Service Principal은 호출 위치에 관계없이 어디서든 사용 가능하지만, Managed Identity는 해당 **Azure 리소스(VM, AKS 등)에서만 토큰 요청**이 가능합니다.
- **보안**: Service Principal은 Secret이 유출될 경우 외부에서 악의적으로 Azure 리소스에 접근이 가능하지만, **Managed Identity는 Secret이 없으므로 유출 위험 자체가 없습니다.**

즉 Managed Identity가 보안상 유리하지만,  해당 테넌트 내부 리소스에서만 사용 가능하다는 제약이 있습니다.

외부에서 접근시에는 Service Principal을 사용해야 합니다.

다만 유출시에는 보안상 치명적일 수 있음을 명시해야합니다.

