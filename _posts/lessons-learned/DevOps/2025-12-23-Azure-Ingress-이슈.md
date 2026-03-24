---
title: "[Azure AKS] Azure Ingress Annotation 중괄호 문법 제한과 404 에러"
author: "unggu"
date: 2025-11-27 19:12:42 +0800
categories: [lessons-learned, DevOps]
tags: [Azure, AKS, Kubernets, Container, DevOps]
render_with_liquid: true
comments: true
image:
  path: assets\img\metaimg\azure_404.png
---

Azure 환경에서 Ingress의 특정 어노테이션은 예기지 못한 동작 유발할 수 있습니다. (라우팅 실패)

이 글에서는 실제 사례를 통해 Ingress의 Silent Failure가 발생했는지 집중적으로 조사하고, 이 과정에서 얻은 Lesson Learned를 정리해 보겠습니다.


## 구조 및 개념 설명

현재 서비스의 네트워크 구조는 간단하게 아래와 같이 이루어져 있습니다.

![image.png]({{ site.baseurl }}{{ page.url }}/img/azure4042.png)

저희가 이번에 자세하게 알아볼 것은 Ingress라는 쿠버네티스 리소스입니다.

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

이런 Controller에는 Nginx, HAProxy 등 다양한 컨트롤러가 존재하지만, AKS는 기본적으로 **Application Routing Add-on** 플러그인의 **Managed NGINX Ingress Controller**를 기본으로 제공합니다.

이 관리형 Ingress는 기본적으로 내부 nginx Deployment/Pod(Nginx)/ConfigMap(NGINX 설정)/Secret(TLS) 등 라우팅에 필요한 모든 정보를 가지고 있습니다.


#### 설정이 적용되는 원리

> [NGINX Ingress Controller - How it works](https://kubernetes.github.io/ingress-nginx/how-it-works/)

Ingress Controller의 최종 목표는 `nginx.conf`의 최종 조립이라고 볼 수 있습니다.

![image.png]({{ site.baseurl }}{{ page.url }}/img/azure4041.png)

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

실제 장애 발생 이전 Ingress에 Annotaion 추가 작업이 있었습니다.

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

즉 해당 설정은 Invalid syntax로 정상적으로 reload되지 못하였고, Ingress는 이전 `nginx.conf` 기반으로 롤백하여 서비스 되었습니다. 

그리고 시간이 흘러(약 10일 이상) Azure의 App-routing Managed 영역의 작업이 있었고 이로 인해 nginx pod가 강제 재기동(restart) 되어졌습니다. (Azure측 별도 공지X)
 
 Invalid Syntax를 가지고 있던 Ingress는 restart 당시 정상적으로 `nginx.conf`가 생성되지 못하였고, 이는 해당 Ingress 리소스 자체가 생성되지 못하는 영향을 발생시켰습니다.

-> **라우팅 실패로 인한 404 에러 발생**

결국 해당 에러는 긴급하게 Annotation 제거 후 재배포하는 것으로 해결되었습니다.
이 경험을 통해 몇 가지 인상 깊은 점을 정리해보았습니다.

---

**이상 신호는 흘려보내지 말 것**

운영 전 개발 서버에서 먼저 유사한 증상이 나타났었습니다.
작은 이상도 이슈로 남겨두는 습관이 나중에 큰 차이를 만들 수 있을 것 같습니다.

**운영 환경은 항상 의도된 상태여야 한다**

효과가 없다고 판단된 설정을 방치했던 것이 문제의 씨앗이 되었습니다.
"서비스 이상 없음"이 "설정이 안전하다"는 의미는 아닙니다.
불필요한 설정은 즉시 제거하는 것이 원칙인 것 같습니다.

**검증은 자동화로**

파이프라인 단계에서 문법 오류를 잡을 수 있었다면 훨씬 빨리 발견할 수 있었을 것입니다.
사람이 놓치는 부분은 자동화가 잡아줄 수 있도록 검증 체계를 갖추는 것이 필요해보입니다.

결과적으로
완벽한 운영보다 중요한 건 이상 신호를 빠르게 감지하고 복구하는 흐름을 만드는 것이 아닐까 싶습니다.