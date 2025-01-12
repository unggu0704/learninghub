---
author: "unggu"
title: "[AZ-204] Azure Functions"
date: 2024-12-26 21:30:01 +0800
categories: [Azure, Azure 204]
tags: [Azure, AKS, AZ204]
render_with_liquid: true
comments: true
image:
  path: assets/img/metaimg/azure204/azure_functions.png
---

# Azure Functions

### Architect

![스크린샷 2024-10-14 오후 2.25.49.png]({{ site.baseurl }}{{ page.url }}/img/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2024-10-14_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_2.25.49.png)

### Azure Functions의 장점

더 적은 코드와 작은 인프라를 사용함으로써 비용을 줄이는 serverless 솔루션으로 어플리케이션이 정상적으로 돌아가도록 모든 필요한 인프라를 클라우드 인프라에서 제공함으로써 서버 배포 및 유지 관리에 용이성을 가질 수 있다.

일반적으로 웹 API, DB의 변경을 감지, 메세지 큐를 관리하는 방법으로 사용된다.

Azure Functions는 코드 실행을 시작하는 **트리거** 방식으로 작동하고 코딩을 간소화하는 **바인딩**의 방식을 지원한다. 

### Serverless?

- 서버의 완정한 추상화
- event-driven 확장성
- 사용한만큼의 요금

**Serverless vs.  PaaS**

- 서버관리 필요성 없고, 코드 위주로 작성 하는것은 비슷
- 개발환경 제어력은 PaaS > Serverless
    - PaaS는 스케일링 구성이 필수적, Serverless는 스케일링 조차 자동으로 해줌

**장점**

- 비즈니스 문제에 집중
- 출시기간의 단축, 비용 절감(호출시에만 지출)
- 손쉬운 프로토타이핑, 마이크로서비스

**Serverless의 중심 FaaS**

- 단일 책임
- 짧은 수명
- 상태 비 저장
- Event driven & scalable

### Azure Functions 정의 및 특징

- 통합 프로그래밍 모델
- 향상된 개발 모델
- 유연한 호스팅 옵션

### Azure Functions의 기능

- 다양한 언어의 지원 → 언어간의 이종 호환은 불가함
- 호스팅 옵션 → 코드 실행 시간에 대해서만 청구
- 라이브러리 사용가능
- Stateful 처리 가능
- Azure 포탈 뿐만 아닌 주류 개발 도구 사용 가능
- 다른 SaaS 솔루션과 간편하게 통합 가능

### Azure Functions의 동작

![스크린샷 2024-10-14 오후 2.57.58.png]({{ site.baseurl }}{{ page.url }}/img/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2024-10-14_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_2.57.58.png)

1. Trigger를 통해 객체를 던짐
2. 코드를 실행하고 필요한 경우 Input Binding 등을 통해 객체를 만듬
3. 처리가 끝나고 Output Binding을 통해 산출물 생성

### Azure Functions의 제약 사항

- 함수는 최대 10분 이상 실행이 불가능함
- 컴퓨팅 자원또한 1.5GB까지의 램과 하나의 CPU 파워만 사용가능
- 실행 주기에 따른 제약을 검

### Azure Functions을 통한 Azure 서비스 통합

![스크린샷 2024-10-14 오후 3.00.21.png]({{ site.baseurl }}{{ page.url }}/img/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2024-10-14_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_3.00.21.png)

### Azure Functions VS Azure Logic Apps

둘다 servless를 사용하는 Azure 서비스이지만 Azure Logic Apps는 serverless 워크플로 통합 플랫폼이다. Azure Functions이 코드 중심적인 반면, Logic Apps는 선언적(디자이너)의 느낌이 강하다.

## Azure Functions 호스팅 계획

### 소비 계획

Azure Functions의 기본 요금제로 함수가 실행 중일 때만 비용을 지불하는 방식이다. 
경제적이지만 소비계획은 함수가 트리거되는데 몇분이 걸릴수도 있다.

### Flex 사용량 계획

미리 프로비전된 인스턴스 수를 지정하면서 높은 확장성을 얻을 수 있는 계획

### 프리미엄 계획

Functions들이 유후 상태로 대기 후 즉시 실행되는 방식, VNet에 이용가능하다. 또한 일반적인 코드 소비계획 보다 높은 사용량이 요구 된다.

> *모든 요금제에서 함수 앱 실행하기 위해서는 일반 Azure 저장소 계정이 필요*
> 

## Azure Functions 크기 조정

> Azure Functions은 *크기 조절 컨트롤러* 를 통해 모니터링 및 확장과 축소를 결정한다.
> 

### 크기 조정

**최대 인스턴스 수** : 단일 function은 200까지 확장 가능, 동시 실행에는제한 없음

**새 인스턴스 수** : HTTP 트리거의 경우 1초에 한개, 나머지는 30초에 한개 

## Azure Functions 개발 과정

![스크린샷 2024-10-14 오후 3.36.28.png]({{ site.baseurl }}{{ page.url }}/img/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2024-10-14_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_3.36.28.png)

**중요한 요소 두가지**

- 코드 : 프로그래밍 언어
- 구성 : *function.json*
    - 트리거, 바인딩 및 기타 구성 설정을 정의

**모든 함수에는 하나의 트리거만 존재!!**

- Function을 만들기 위해서는 일반적인 개발 도구를 사용해 함수를 만들고 테스트 할 수 있다.

### Function 앱

함수가 실행되는 컨텍스트 

- 함수의 배포 단위 (관리 단위)
- 하나 이상의 개별 함수로 구성되어 모든 기능은 동일한 정책으로 통일 되어 있음

 

## 트리거 및 바인딩 만들기

- 트리거 → 함수가 실행되는 원인 (함수와 트리거는 일대일 대응)
- 바인딩 → 다른 리소스를 함수에 연결하는 것 (입력 바인딩, 출력 바인딩)
    - 하드코딩 엑세를 방지

### 바인딩 방향

- 트리거에는 방향 속성이 존재
    - 트리거의 경우 `in`
    - 입력 출력 바인딩의 경우 `in` 과 `out`

### 트리거 바인딩 예제

![스크린샷 2024-10-14 오후 3.49.34.png]({{ site.baseurl }}{{ page.url }}/img/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2024-10-14_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_3.49.34.png)

- Azure Queue Storage 메세지 표시 → Azure Table Storage에 새 행을 작성하는 예제

## Azure Durable Function

Azure Functions의 확장 기능으로 비동기 작업을 쉽게 관리할 수 있게 해주는 기능이다.

**함수 체이닝**
![imaeg](https://learn.microsoft.com/ko-kr/azure/azure-functions/durable/media/durable-functions-concepts/function-chaining.png)
복잡한 비즈니스 프로세스를 정의하는데 있어 여러 Functions들을 조합하며 순차적 또는 병렬로 실행이 가능하다.

**팬아웃/팬인**
![image](https://learn.microsoft.com/ko-kr/azure/azure-functions/durable/media/durable-functions-concepts/fan-out-fan-in.png)
여러 함수의 병렬 실행하고 모든 함수의 완료를 기다린다. 

**비동기 HTTP API**
![image](https://learn.microsoft.com/ko-kr/azure/azure-functions/durable/media/durable-functions-concepts/async-http-api.png)
외부 트리거에 반응하여 실행될 수 있으며, HTTP 엔드포인트에서 장기 실행 작업을 트리거한다. 이러한 방식으로 작업의 분할처리가 가능해 뻐른 처리가 가능하다. 
Durable Functions는 이 패턴에 대해 기본 제공 지원을 제공하며 이러한 오케스트레이션을 관리하는 HTTP API를 기본적으로 제공한다.




