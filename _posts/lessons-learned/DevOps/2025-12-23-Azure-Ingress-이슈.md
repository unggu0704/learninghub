---
title: Azure환경에서 Ingress 라우팅 실패로 404 발생한 이야기
author: "unggu"
date: 2025-11-27 19:12:42 +0800
categories: [lessons-learned, DevOps]
tags: [Azure, AKS, Kubernets, Container, DevOps, Key Vault]
render_with_liquid: true
comments: true
image:
  path: assets\img\metaimg\azure_404.png
---

2025년 10월 4일 파리의 한 숙소에서 아침을 맞이하고 눈을 떳을 때, 핸드폰에 다수의 알람과 부재중 전화가 찍혀있던 경험이 어제와 같이 생생합니다.

저의 서비스가 자는 시간 동안 (한국 기준 오전 10시 경)에 웹 사이트 접속 불가 화면이 발생하면서 이상 징후가 발생하였고 이를 해결하기 위해 유관부서 및 팀원분들이 열심히.. 원인을 파악한 일화가 있습니다.
당시에는 Ingress의 라우팅 실패로 인한 서비스 장애로 결론 내렸고, 결국 Ingress의 특정 어노테이션을 제거함으로써, 제가 자는동안(?) 모든 상황은 다행히도 해결 되었습니다.

저는 귀국과 동시에 약간의 눈치와 함께 왜 이런 서비스의 Slient Failure이 발생했는지 집중적으로 조사하였고 이 과정에서 얻은 Lesson Learned을 정리해 보겠습니다.


## 구조 및 개념 설명

현재 서비스의 네트워크 구조는 간단하게 아래와 같이 이루어져 있습니다.

![image.png]({{ site.baseurl }}{{ page.url }}/img/azure4041.png)

이번에 말썽을 부린건 Ingress 이 친구가 원인이였습니다. 
쿠버네티스의 리소스인 Ingress는 HTTP/HTTPS 트래픽 라우팅 규칙을 정의하는 기능을 가지고 있지만, 이런 Ingress 자체가 기능을 제공하는 주체가 아닌, 그 상위 리소스인인 **Ingress Controller**가 실제 기능을 가지고 수행합니다.
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  ingressClassName: nginx  # 어떤 Controller를 사용할지 지정
  ```
그리고 다수의 Ingress Controller가 있다면 이걸 어떤 Controller가 처리할지를 정의한 것이 **IngressClass**입니다. 

```
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: nginx
spec:
  controller: k8s.io/ingress-nginx  # 실제 Controller 식별자
  ```

이런 Controller에는 Nginx, HAProxy 등 다양한 컨트롤러가 존재하지만 AKS는 기본적으로 **Application Routing Add-on** 플러그인의 **Managed NGINX Ingress Controller**를 기본으로 제공합니다.

이 관리형 Ingress는 기본적으로 내부 nginx Deployment/Pod(Nginx)/ConfigMap(NGINX 설정)/Secret(TLS) 등 라우팅에 필요한 모든 정보를 가지고 있습니다.


#### 설정이 적용되는 원리

> [NGINX Ingress Controller - How it works](https://kubernetes.github.io/ingress-nginx/how-it-works/)

Ingress Controller의 최종 목표는 `nginx.conf`의 최종 조립이라고 볼 수 있습니다.

![image.png]({{ site.baseurl }}{{ page.url }}/img/azure4042.png)

kube-api가 Ingress를 비롯해 관련된 secret, svc, cm 등을 관리하며 변화를 감지하는데 `Ingress.yml`의 변화가 발생하면 kube-api는 이 변화를 Work Queue에 추가하여 새로운 `nginx.conf`를 생성하여 최종적으로 이 파일을 바탕으로 새로운 Ingress Controller는 동작합니다.

새로운 Ingress Controller가 만들어질 때 두 가지 방식으로 만들어집니다 (reload vs restart)

#### restart는 아래와 같은 조건에서 동작합니다.
- 모든 process 종료
- NGINX pod의 재시작


#### reload는 아래와 같은 방식에서 동작합니다.
- Ingress 추가/삭제
- Ingress의 host, path 규칙 변경
- TLS Secret 추가/변경
- Annotation 변경 (`nginx.conf` 구조에 영향을 주는 것)

만약 해당 구조 변경 중 **실패하면 변경사항을 롤백하고 기존 `nginx.conf`를 계속해서 사용합니다.**

-> **이런 의도치 않은 slient failure가 최종적으로 장애의 원인이 되었습니다.**


## 타임 테이블

10/4부터 약 10일 전 9/24쯤  Ingress에 Annotaion 추가 작업이 있었습니다.

```
annotations:
  nginx.ingress.kubernetes.io/configuration-snippet: |
    some config {
      # 대강 설정 내용
    }  
```

일반적인 Ingress에서는 `{`, `}`를 문법에서 사용하는게 큰 제약사항은 없지만 Azure의 IngressController는 해당 문법을 금지한다고 합니다. (왜?)

> [MS Learn - app-routing limitaions](https://learn.microsoft.com/en-us/azure/aks/app-routing#limitations)

그 외 금지하는 문법
```
load_module
lua_package
_by_lua
location
root
proxy_pass
serviceaccount
{ (중괄호 열기)
} (중괄호 닫기)
' (작은따옴표)
```

즉 해당 설정은 Invaild syntax로 정상적으로 reload되지 못하였고, Ingress는 이전 `nginx.conf` 기반으로 롤백하여 서비스 되었습니다. 

그리고 시간이 흘러.. 10/4에 Azure의 App-routing Manenged 영역의 작업이 있었고 이로 인해 nginx pod가 강제 재기동(restart) 되어졌습니다. (Azure측 별도 공지X)
 
 Invaild Syntax를 가지고 있던 우리 Ingress는 restart 당시 정상적으로 `nginx.conf`가 생성되지 못하였고, 이는 해당 Ingress 리소스 자체가 생성되지 못하는 영향을 발생시켰습니다.

-> **라우팅 실패로 인한 404 에러 발생**

## 장애 대응적 관점

해당 이슈는 여러가지 상황이 복합적으로 맞물려 발생한 이슈로 보입니다. Azure에서 지속적으로 서비스를 운영할 때 다시 이런 문제가 발생하지 않기 위해서 어떤 절차 or 프로세스가 필요할지 스스로 검토해보았습니다.

#### 절차적 관점

**환경의 통일성 관리**

  Ingress Annotaion을 변경할려고 한것은 AppGW에서 넘어온 xff 헤더에 대한 값을 변조시킬 필요가 있었기 떄문입니다. 이런 AppGW 환경은 운영 환경에서만 적용이 되어 있었기 때문에 이런 반영은 다른 환경에서 충분한 검토를 할 기회가 없었습니다. 
  
  이를 통해 개발/디버그/운영 환경에서의 환경 통일이 필요할듯 보입니다. 
  (현재는 예산의 이유로 운영환경에만 설치되어 있음)

**운영은 언제나 clean해야 한다**

  운영 반영 이후 해당 설정을 통해 만족할만한 결과를 얻지 못했습니다. (Ingress는 실제로 롤백됨으로 설정이 먹히지 않음)
  
  하지만 단순히 서비스 점검 결과 이상없음으로 해당 설정을 원복하지 않고 운영환경에 그대로 남겨두었습니다. 운영환경에서는 언제나 clean한 형상관리가 필요할듯 보입니다.

### 기술적 관점

**역량의 부족** 

  공식문서는 언제나 정답이 있습니다. 기술적 검증 절차를 통해 반영 내용에 대한 영향도를 검토하고 이를 꼼꼼히 검증해야합니다.

**신호 감지의 중요성**

  운영 환경이 발생하기 약 3일전 개발 서버가 404가 발생하는 이슈가 있었습니다. 

  당시에는 이를 대수롭지 않게 여기며 해당 Annotation 제거를 통해 해결하였는데 Azure라는 관리형 서비스 특성상 유지관리 일정을 지속적으로 확인하여 이에 대비해야 할듯 보입니다. (개발 이슈 또한 JIRA 이슈화하여 운영 형상 대비)
  
  ```
  ### AKS 유지관리 일정 확인
  az aks get-upgrades \
  --resource-group myRG \
  --name myCluster
  ```

**검증 체계의 부족**

  사람은 실수할 수 있음을 인정하고 모니터링 및 사전검증 체계 구축이 필요해보입니다.

  실제로 reload 후 Nginx pod에서 지속적으로 `WARN` 등급의 log가 발생함을 확인하였습니다. 이를 App Insight등을 통해 capture 할 수 있는 방안이 필요해보입니다.

  또한 Azure에서 이런 문법에 대해 fail fast를 제공해주지 않기에 CI/CD 파이프라인에 유효성 검증을 하는 절차를 추가하는 것도 좋은 방법이 될 수 있을 것입니다.


## 끝으로...

IT 서비스는 언제나 위태롭고 1년간 무사히 움직이다 하필.. 놀러가거나 방심할 때 터지는거 같습니다.

아프지말고 무럭무럭 있어주렴 [서비스](https://globalshop.kt.com)야...