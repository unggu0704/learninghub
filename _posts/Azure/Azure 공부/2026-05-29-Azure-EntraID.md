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

![image]({{ site.baseurl }}{{ page.url }}/image/entraid.png)

모든 인증은 Entra ID와 통한다는 말과 비슷하게 이것 또한 Entra ID의 일부분인 SSO 인증의 일부로 Azure 관련 접근을 위해 사람(ID/Password)를 통해 인증을 요구합니다. 

```
az login
```

### App Registration

사용자가 SSO를 통해 Azure 리소스에 접근하듯, 외부 서비스가 Azure 리소스에 접근이 필요할 떄가 있습니다. 

그럴 때 **App Registration**를 사용합니다. App Registration는 테넌트 단위의 객체이기에 같은 구독이 아닌 다른 구독 / 외부 서비스에서Service Principal를 통해 `az login`을 직접 한 것과 같은 효과를 내며 접근이 가능합니다.

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

이것들은 각각 아래와 같은 의미로 사용됩니다.

- AZURE_TENANT_ID → "어느 Entra ID 테넌트냐" 
-  AZURE_CLIENT_트 내 다른 Azure 리소스에 인증할 때 사용한다는 점이 Service Principal과 비슷합니다.

다만, 자격증명을 직접 관리하지 않고, **Azure에서 관리하며 같은 구독 내 한정되어 사용한다는 점**에 있어서 차이점이 존재합니다.

즉 Service Principal는 범위가 훨씬 넓지만 이 값이 유출되지 않도록 언제나 주의해야합니다.

(유출 시에는 외부에서 악의적인 목적으로 Azure 리소스에 접근 가능)
