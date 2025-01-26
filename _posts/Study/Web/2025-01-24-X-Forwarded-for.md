---
title: "[Web] X-Forwarded-For & X-Forwarded-By"
author: 김규형
date: 2025-01-24 17:51:53 +0800
categories: [Study, Web]
tags: [Web, Http]
render_with_liquid: true
comments: true
image:
    https://blog.wearedrew.co/hubfs/1.%20Im%C3%A1genes%20de%20dise%C3%B1o%20%5BNO%20TOCAR%5D/DDC1641701725.jpg
---

## X-Forwarded-For

L7 로드밸런서 또는 L4 로드밸런서를 거쳐 들어온 트래픽은 웹 서버에서 장비 IP 또는 Proxy Server의 IP로 보이기도 한다. 
![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fw0iYZ%2FbtsdzBTv9Id%2FkWABhIvQraporzX256wWYK%2Fimg.png)

이를 추적하기 위해 **X-Forwarded-For Header**를 사용하면 Client의 IP를 유지할 수 있다. 

**일반적인 X-Forwarded-For**
```
X-Forwarded-For: client1, proxy1, proxy2
```
원래 IP주소, 거쳐온 proxy1, proxy2의 주소가 담겨있다.

이를 통해 백엔드 서버는 이 Header를 이용해 클라이언트의 실제 IP 주소를 받아 AP적 요구사항을 달성할 수 있다.(Sticky Session...)


이는 Apache나 Nginx 같은 Web서버 단에서 설정을 해야한다.

--- 

## X-Forwarded-By

X-Forwarded-By는 Client의 IP를 제외한 Proxy의 IP나 서버 정보를 가지고 있다.
또한 플랫폼 및 프레임워크의 사용과 정보를 알려주는 유무를 가지는데 이는 개발적으로 유용한 정보를 제공하지만 보안상 여러 취약점을 유발할 수 있다.

**Server의 정보를 알려주는 예시**
```
X-Powered-By: JSP/3.1
```
위 정보는 해당 서버가 JavaServer Pages를 사용하고 있다는 점과 3.1 버전을 사용하고 있다는걸 제공한다. 
이를 통해 공격자는 특정 프레임워크나 기술 및 특정 버전에 대한 취약점을 악용할 수 있다.

이를 제거하기 위해서는 was 설정 단에서 이러한 Header를 노출하지 않도록 조치해야한다. 

**Wildfly에서 X-Forwarded-By 헤더 정보 숨기기**
```
<subsystem xmlns="urn:jboss:domain:undertow:14.0"...>
	... 중략
	<servlet-container name="default" default-encoding="utf-8" use-listener-encoding="true">
	...
		<jsp-config  x-powered-by="false"></jsp-config>    <!-- x-powered-by disable -->
	</servlet-container>

</subsystem>
```
X-Forwarded-By는 표준화된 방식이 아니기에 사용이 지양되는 편

