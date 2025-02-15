---
author: 김규형
title: [Spring] Static 필드와 Instance 필드에서 객체 주입
date: 2025-02-12 18:31:16 +0800
categories: [Framework, Spring Essentials]
tags: [BackEnd, SpringBoot, Spring]
render_with_liquid: true
comments: true
image: 
  path: assets/img/metaimg/spring.png
---

### Spring DI와 Static 필드

반년간 Azure와 k8s에서 공부를 하며 프로젝트를 진행하며 어느덧 프로젝트도 막바지에 다가와 보안성 검증 단계에 있다. 
보안성 검증을 진행하면서 소스에 하드코딩된 부분을 수정하며 Spring Framewokr 지식의 부족으로 조금 헤맨 것에 대해 간략히 적어볼려고 한다. 

`instance` 필드와 `static` 필드에서 각자 `.properties` 의 값을 가져오는 코드가 있다고 가정한다. (*본래 하드코딩 되어 있었지만 보안을 위해 암호화 된 값을 `.properties`에 저장함*)
일반적으로 `instance` 필드에서 `.properties` 에서 값을 가져오기 위해 Spring에서 지원하는 두 가지 방법을 통해 값을 가져온다.

1. `@Value` 어노테이션을 사용
2. 별도의 서비스 객체 **PropertyService** 객체를 `@Autowired`를 통해 객체 주입해서 사용

하지만 `static` 필드에서 이런 방법 사용하기 위해서 아래와 같이 코드를 작성한다면 문제가 발생한다.

**1. `@Value` 어노테이션을 사용**
```
@Value("#{prop['ONLINE_URL']}")   
static private String ONLINE_URL;  
```

**2. `@Autowired` 통해 PropertyService 객체 주입 받아 사용**
```
@Autowired
static private PropertyService propertyService;
```

해당 코드들은 Spring 관리 아래에 있는 인스턴스에 대해 의존성을 주입하는 형식이지만 , `static`필드는 클래스가 메모리에 로드될 시기에 생성되기에 **Spring**과의 관계성을 설정할 수 없다. 
두 가지 어노테이션의 방식은 컴포넌트 스캐닝을 통해 인스턴스 변수에 주입받지만 `static`은 이러한 범위 밖에 있기에 **의존성이 설정되지 못해 `null`이 주입되어진다.** (*디버깅의 어려움*)

이를 해결하기 위해 아래와 같은 식으로 코드를 작성하였다.

**static 영역과 일반 영역 모두에서 사용할 수 있도록 두 개의 service 객체 생성**
```
@Autowired
private PropertyService propertyService;

private static PropertyService staticPropertyService;
    
@PostConstruct
protected void init() {
	AdminLoginServiceImpl.staticPropertyService = this.propertyService;
}
```
`@PostConstruct` 어노테이션을 통해 `static` 필드의 초기화를 지연시키면서 `propertyService`가 확실히 주입된 후에 작업이 진행하도록 설정하였다. 

### 완벽한 방법일까?
완벽한 방법이라고는 할 수 없다고 생각한다. 내가 생각하는 해당 코드는 요구사항은 만족할지언정  하나의 class에 두 필드(Instance, Static)에 대해 객체를 생성하는 것은 코드를 모호하게 하고 가독성이 떨어진다고 느낀다. 

`PropertyService` 객체를 직접 **sigleton** 객체로 만드는 방식도 생각을 했지만 이미 다른 곳에서 `@Autowired`로 주입 받으면서 시스템에 대한 영향도의 파악이 힘들어 Service 로직을 수정하지는 못하고 이정도 수준으로 타협해야할 것으로 보인다.

이러한 구현을 거치며 가능한 `static` 필드를 사용하는 것을 지양하고 (이번 case는 이미 static으로 code가 구현된 상태에서 수정되어야만 했음) Spring의 DI를 염두하여 코드를 설계할 필요성을 느낀다.
