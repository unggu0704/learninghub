---
author: "unggu"
title: "[AZ-204] Azure Cache for Redis"
date: 2025-01-05 17:31:04 +0800
categories: [Azure, Azure 204]
tags: [Azure, App Redis, AZ204, Redis]
render_with_liquid: true
comments: true
image:
  path: assets/img/metaimg/azure204/azure_redis.png
---

## Azure Cahe for Redis

시스템의 성능 및 확장성을 개선하는데 사용되는 **캐싱**기술 Application의 응답 시간을 훨씬 향상하는데 도움을 준다.

Azure에서는 Redis를 사용하여 이러한 서비스를 제공한다.

### Azure Redis 주요 사용 패턴

![image.png]({{ site.baseurl }}{{ page.url }}/img/image32.png)

**데이터 캐시**

데이터베이스는 너무 커서 캐시로 직접 로드하지 않고 필요한 부분(*자주 사용하는*) 부분을 메모리에 저장하여 빠르게 읽을 수 있게 하는 전략이다.

주로 **캐시 배제 전략**을 사용하여 캐시에 데이터를 로드한다. 

> 캐시 배제 패턴이란?
*Cahce Miss가 발생하면 데이터를 cache에 저장하고 데이터에 변화가 감지되면 cache를 무효화하는 전략*
> 

**콘텐츠 캐시**

해더나 바닥글 같은 곳은 정적 항목으로 변화가 없기에 캐시화한다.

**세션저장소**

사용자의 세션을 저장하는 기록 데이터로 사용될 수 있다. 일반적으로는 쿠키에 저장하지만 너무 큰 쿠키 저장소는 성능 저하를 유발한다. Azure Cahe for Redis를 사용해 저장하는 방식이 사용된다.

**작업 및 메세지 큐**

AP의 요청에 작업을 추가하고 다른 서버에 순서대로 처리되도록 큐를 구성한다. 

**분산 트랜잭션**

AP의 원자성 작업을 보장하기 위해 일련믜 명령을 저장하고 롤백하여야한다. 이러한 명령의 일괄 처리 실행을 위해 단일 트랜잭션을 지원한다.

### 서비스 계층

Azure Cache for Redis는 아래와 같은 계층으로 사용이 가능하다.

- Basic : 단일 VM을 통해 OSS Redis 캐시
- Standard : 복제된 구성의 두가지 VM OSS Redis 캐시
- Premium : 고성능 OSS Redis 캐시, 강력한 VM
- Enterprise : 높은 가용성, redis enterprise sw로 구동되는 캐, **Redis 모듈 지원** (*RediSearch, RedisBloom* 등의 기능을 지원)
- Enterprise Flash : 대용량 캐시 VM의 DRAM보다 저렴한 비휘발성 메모리로 확장

> *일반적으로 캐시는 휘발성 메모리이지만 Azure의 데이터 지속성을 제공하여 데이터 손실을 방지할 수 있다.*
> 

**Redis 모듈이란?**
일반적인 Redis 명령 이외에 모듈을 통해 복잡한 연산을 수행 할 수 있다. 거기에는 RediSearch, RedisBloom 등의 모듈이 존재하며 
대규모 AP의 데이터 인덱싱, 고속 JSON I/O, 머신 러닝의 사용 등에 기용된다. 


### 실습

아래 코드는 Java에서 Redis를 사용하고 이를 Azure Redis에 저장하는 방식이다.

```jsx
import redis.clients.jedis.Jedis;

public class AzureRedisExample {
    public static void main(String[] args) {
        // Azure Redis Cache 연결 
        String redisHost = "{Azure Redis 주소}"; // Azure Redis Cache 호스트
        String redisKey = "{엑세스키}"; // 액세스 키

        // Redis 연결
        try (Jedis jedis = new Jedis(redisHost, 6380, true)) {
            jedis.auth(redisKey); // 인증
            jedis.ping(); // 연결 확인

            System.out.println("Azure Redis Cache에 연결되었습니다.");

            // 데이터 저장
            jedis.set("exampleKey", "Hello, Azure Redis!");
            System.out.println("데이터 저장 완료: " + jedis.get("exampleKey"));

            // 데이터 갱신
            jedis.set("exampleKey", "Updated Value");
            System.out.println("데이터 갱신 완료: " + jedis.get("exampleKey"));

            // 데이터 삭제
            jedis.del("exampleKey");
            System.out.println("데이터 삭제 완료: " + jedis.get("exampleKey"));
        } catch (Exception e) {
            System.err.println("Redis 연결 실패: " + e.getMessage());
        }
    }
}

```

### 값에 만료 시간 추가

캐싱된 값은 일반적으로 시간이 흐르면 사라져야한다. 이를 위해 TTL을 적용하는 것은 중요하다!
만약 Redis 서버가 중지된 상태로 남아 있으면 만료에 대한 정보까지 복제 되어 디스크에 보관되어진다. 

**Java의 TTL 설정**

```jsx
jedis.setex("tempKey", 300, "This will expire in 300 seconds");
```

---

## Azure Content Delivery Network

전 세계에 걸쳐 배포된 노드에 콘텐츠를 캐시하여 사용자 성능 및 사용자 환경을 향상한다. 이러한 에지 서버 방식을 사용해 원본 서버로 전송되는 트래픽의 양을 감소 시킨다. (이러한 CDN 프로필은 구독 유형에 따라 제한됨)

![image.png]({{ site.baseurl }}{{ page.url }}/img/image31.png)

어느날 Alice는 하나의 File을 요청하였다. DNS는 이를 사용해서 가장 성능이 좋은 PoP위치로 라우팅 하였다. 

POP의 Edge 서버에 캐시 파일이 없다면 POP은 이를 원본 서버에 요청을한다. 원본 서버로부터 정보를 받은 POP은 이 파일을 캐싱하고 요청에 대한 응답을 Alice에게 반환한다. 이 파일을 TTL 기간 동안 유지되며(7일) 추후 다른 사용자가 이 파일을 요청하면 원본서버를 참조하지 않고 POP에서 전송해준다.

### 캐싱 동작 제어

Azure CDN의 캐시되는 방식은 두 가지 방식을 기준으로 제공되어진다.

**캐싱 규칙**은 사용자가 지정한 캐싱 규칙으로 **전역 캐싱 규칙**은 엔드포인트에 영향을 주는 전역 캐싱 규칙을 설정한다. **사용자 지정 캐싱 규칙**은 각 엔드 포인트마다 설정하며 특정 경로 및 확장명과 일치되어 순서대로 처리한다. 

**쿼리 문자열 캐싱**은 쿼리 문자열을 사용해 캐싱을 처리하는 방식이다.