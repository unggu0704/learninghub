---
author: "unggu"
title: "[AZ-204] Azure Container Registry"
date: 2024-12-28 18:48:15 +0800
categories: [Azure, Azure 204]
tags: [Azure, AKS, AZ204, Docker]
render_with_liquid: true
comments: true
image:
  path: assets/img/metaimg/azure204/acr.png
---

# Azure Container Registry

> *Azure 컨테이너를 저장해놓는 Dokcer Registry 2.0 기반의 관리형 레지스트리 서비스*
> 

ACR(Azure Container Registry)는 배포 파이프라인을 통해 Azure에서 컨테이너 등을 빌드 할 수 있는데, 주문형 또는 소스 코드 커밋등의 *트리거* 기능을 통해 완전 자동화가 가능하다.

또한 `YAML` 등을 사용하여 다단계 작업의 기능 또한 제공하는데 

```yaml
1. 웹 어플리케이션 이미지 빌드
2. 웹 어플리케이션 컨테이너 실행
3. 웹 어플리케이션 테스트 이미지 빌드
4. 웹 어클리케이션 테스트 컨테이너 실행 
5. 테스트 정상일시 ACR에 이미지 push
```

위와 같은 시나리오로 작성이 가능하다. 

### 일반적인 ACR 시나리오 (Git Action을 사용한)

![alt text](https://miro.medium.com/v2/resize:fit:1048/1*wBcz95q3xz_-L9DBgsuToA.png)_Git Action을 사용한 파이프라인_

1. ACR 생성
2. DockerFile에서 이미지 푸쉬(Git Action) 
3. 결과 확인
4. ACR 이미지 실행
5. 리소스 정리 



### ACR 서비스 계층

기본적으로 **Basic**은 비용 최적화된 진입점으로 표준적인 프로그래밍 기능을 제공한다. 하지만 스토리지나 이미지 처리량은 낮기에 이를 향상시킨 **Standard** 계층이 존재한다. 

**Premium** 계층은 동시 작업 수가 가장 높으며 고용향 시나리오를 제공하며, 지역 복제, 이미지 태그, 그리고 엑세스를 제한하며 PE(Private Endpoint)를 사용한 프라이빗 링크를 제공한다.  

### 스토리지 기능

모든 ACR 계층은 이미지를 저장하는 스토리지의 기능을 수행하는데 저장소 및 태그의 수가 증가한다면 ACR 성능에 영향을 줄 수 있다. 

ACR은 일반적으로 미사용시 암호화 기능을 제공하는데 서비스 이미지가 끌어올려 질때, 이미지의 암호를 해독하는 방식을 사용한다. 필요에 따라 고객관리형 키를 사용하여 추가적인 암호화 계층을 적용이 가능하다. 

지역 가용성과 관련해서는 **지역 스토리지**를 사용해 동일한 지역의 쌍을 이루는 지역에 레지스트리 데이터를 저장할 수 있지만, 지역 중단이 발생할 경우 복구되지 못한다. 이를 방지하기 위해 **지역 복제** 라는 시나리오를 사용할 수 있는데 이는 지역 장애 발생시에 엑세스 권한 손실을 방지하는데 도움을 준다. 

**영역 중복**은 각 지역에서 최소 3개의 개별 영역에 복제하여 가용성을 최대한으로 끌어올리는 기능이다.

 

> *일반적으로 ACR은 휘발성이 있는 저장소이다. 그렇기에 이미지 자체의 무언가를 저장할려면 Azure Storage 같은 외부 저장소를 사용하는것이 필수적이다.*
> 

## Azure Container Instances

### Azure Container Instances란

가상머신을 관리할 필요 없이 더 빠르게 컨테이너를 실행 할 수 있다. DockerHub나 ACR에서 이미지를 가져와 독립적으로 실행하는 pod 생성 가능 

### Azure Container Instances의 구조

![스크린샷 2024-10-15 오후 2.54.54.png]({{ site.baseurl }}{{ page.url }}/img/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2024-10-15_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_2.54.54.png)

**컨테이너 그룹(myContainerGroup)** : 최상위 리소스 컨테이너 그룹
- 컨테이너들의 수명 주기, 리소스들을 관리
**배포**
- ARM 템플릿
- YAML 파일
**리소스 할당**
- 인스턴스에 CPU, GPU 등 할당
**정책**
- 컨테이너 그룹을 만들때 아래와 같은 정책 생성 가능
    - Always
    - Never
    - OnFailure
- 이외에도 환경 변수 설정 가능

## Azure Contaier Apps

![alt text]({{ site.baseurl }}{{ page.url }}/img/ACA.png)

Azure Container Apps는 AKS기반 Serverless 플랫폼에서 컨테이너나 어플리케이션을 실행할 수 있다. 주로 API 엔드포인트 배포, MSA 실행, 이벤트 기반 처리 수행등의 기능을 수행하여 HTTP 트래픽이나 Event 기반으로 동적 확장이 가능하다.

또한 Azure service app처럼 배포 slot을 지원하며 로그 기반 모니터링이 가능하다. 

### Azure Container Apps 환경

컨테이너 앱 그룹을 중심으로 단일 Container Apps 환경에 배포하거나 다른 환경에 배포할 수 있는데 동일한 환경에 배포시, 관련 서비스나 동일한 로그 분석이 가능하고 다른 환경에서 배포시 두 어플리케이션은 통신이 어려우며 컴퓨팅 리소스를 공유하지 않는다. **특히 PRD/DEV의 분리에 있어 다중 환경 관리는 필수적이다.**

**Azure Container App 만들기**
`az containerapp up` , `--source .` -> DockerFile로 만들어진 image의 Azure Container App으로 배포

### Azure Container Apps 컨테이너

![image.png]({{ site.baseurl }}{{ page.url }}/img/acaarch.png)

Azure Container Apps는 Linux기반 컨케이너 이미지를 지원한다. 이러한 단일 컨테이너 앱에서 여러 컨테이너를 추가로 정의하여 *사이드카* 패턴을 구현할 수 있다.

또한 MS ID를 포함한 다양한 ID 공급자를 지원한다.

**Dapr와 연동되어 안정적인 상호 통신 기능을 제공한다.**

### Azure Container Apps의 수정모드(Revision) 및 비밀 관리

수정 버전(**Revision 모델**)을 만들어 컨테이너의 앱 버전을 관리한다. 이런 revision을 **Label**을 사용하여 관리할 수 있는데 
특정 리비전으로 트래픽의 라우팅이 필요할때 사용될 수 있다. 


`az containerpp update` 명령어를 사용하여 만드는 예제 코드
```yaml
az containerapp update \
  --name <APPLICATION_NAME> \
  --resource-group <RESOURCE_GROUP_NAME> \
  --image <IMAGE_NAME>
```

이러한 수정모드는 두가지로 이루어져있다.

**단일 수정 모드**

하나의 리비전만 활성화 되며 새 리비전이 배포되면 기존 리비전은 비활성화 되는 방식

**단일 수정 모드에서도 무중단 배포는 가능하다!!!**

**다중 수정모드**

여러 리비전을 동시에 실행하고 트래픽을 분산시킨다. 

배포시에는 점진적 배포를 통해 안정성이 높다 

만약 App이 사용중에 비밀을 추가 업데이트 되는 경우 새 수정본을 배포하거나, 기존 버전을 재시작한다. 삭제되는 비밀은 앱에 영향을 주지 않는다.

또한 Azure Key Vault를 지원하지는 않지만 관리ID를 통해 SDK를 사용해 비밀을 엑세스할 수 있다. 

### 비밀  정의

컨테이너 앱을 만들 때 `secureValue` 속성을 지정하는 환경변수를 만들거나 `--secrets` 매개 변수를 사용하여 비밀을 정의한다. 주로 이름/값 쌍세트로 허용되며 쌍은 `=` 로 구분된다. 

```yaml
az containerapp create \
  --resource-group "my-resource-group" \
  --name queuereader \
  --environment "my-environment-name" \
  --image demos/queuereader:v1 \
  --secrets "queue-connection-string=$CONNECTION_STRING"
```

---

### 요약

이 포스트를 통해 아래와 같은 기능을 배웠다.

- ACR의 기능 및 이점
- ACR의 배포 자동화 시나리오
- ACI의 구조
- Azure Container Apps