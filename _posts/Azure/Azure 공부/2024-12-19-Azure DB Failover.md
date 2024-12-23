---
title: Azure DB Failover란?
author: "unggu"
date: 2024-12-19 15:34:21 +0800
categories: [Azure, Azure 공부]
tags: [Azure, AKS, Kubernets, Container, DevOps, DB]
render_with_liquid: true
comments: true
image:
  path: https://www.azure365pro.com/wp-content/uploads/2024/05/34.png
---

## Failover란?
![image1](https://blog.kakaocdn.net/dn/byuAa7/btqEdkma4CJ/gSqgAatLXSucGXBTtes6F1/img.png)

Failover는 시스템이나 서버 또는 DB가 실패할 경우 일종의 백업 서비스이다.
주 시스템의 장애나 문제로 인해 다른 물리적 지역에 존재하는 백업시스템으로 자동 전환되는 시스템을 제공한다. -> 고가용성 보장


### Failover 시스템의 유형 
- **핫 스탠바이** :  상시적으로 가동되고 있음 문제 발생시 즉시 대체되는 방식
- **콜드 스탠바이** : 대기 상태로 존재하지만 가동하진 않름, 장애 발생시 수동 개입 또는 지연이 존재
- **웜 스탠바이** : 코어 서비스 준비 상태로 주 시스템의 모든걸 동기화 하지 않음 문제 발생시 약간의 설정과 함께 전환 될 수 있는 준비 상태


## Azure DB failver

**아키텍쳐**
![image2](https://learn.microsoft.com/ko-kr/azure/azure-sql/database/media/failover-group-overview/failover-group.png?view=azuresql)

Azure SQL Database의 장애조치 그룹을 통해 데이터베이스의 가용성을 높이기 위해 설계 
기본 인스턴스의 장애가 감지되거나 수동으로 클릭(?)시 보조 인스턴스로 전환 일반적으로 read와 write를 위한 Endpoint가 동일하다 -> **AP 코드 단의 수정이 필요 없음!**

주 서버와 보조 서버는 서로 다른 지역(Region)에 위치해야함

failover가 일어나는 동안에 데이터베이스 서버가 정상작동되지 않음 -> 데이터 손실 가능성 존재


### failover 테스트
> 정상적으로  failover가 되는지 테스트하는 것은 중요하다.

**Force failover**
- 강제 장애조치라고 불리는 방식은 기본 데이터베이스를 보조 데이터베이스로 전환하고, 보조 데터베이스를 기본 데이터베이스로 승격 시키는 작업
- Azure Portal에서 버튼을 클릭!
	- CLI 방법도 있다..
- 이러한 방법은 승격 과정에서 데이터 손실 가능

**기동 확인**
- Azure SQL DB에서 활동 로그 클릭 
	- Restart PostgreSQL server
	- Health Event Updated
- DBMS에서 데이터가 정상적으로 인입되는지 확인
	- 1초마다 데이터를 집어 넣고 얼마나 데이터가 손실 되는지 체크
> *실제 데이터가 인입된 시간과 해당 시간이 뭔가 맞지 않는다... *
