---
author: 김규형
title: "[Spring] Spring Security 세션 관리의 숨은 트리거"
date: 2026-09-12 10:14:00 +0800
categories: [Framework, Spring Essentials]
tags: [Spring]
render_with_liquid: true
comments: true
image: 
  path: https://www.dariawan.com/media/images/tech-spring-security.width-1024.png
---

Spring Security는 Spring 내에서 인증 파이프라인을 제공하는, 필수적으로 사용되는 Spring 관련 라이브러리입니다.

Spring Security를 사용하지 않는다면 개발자는 아래와 같이 JWT 토큰을 수동으로 관리하고 검증해야 합니다.

```java
// 인터셉터나 필터에서 직접
public class JwtInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest req, HttpServletResponse res, Object handler) {
        String token = req.getHeader("Authorization");
        if (token == null || !jwtUtil.isValid(token)) {
            res.setStatus(401);
            return false;  // 여기서 직접 막음
        }
        User user = jwtUtil.parseUser(token);
        req.setAttribute("currentUser", user);  // 직접 어딘가에 담아서 넘김
        return true;
    }
}
```

그리고 이걸 `ADMIN`만 수용한다면 아래처럼 매번 if 문을 작성해야 합니다.

```java
if (!user.getRole().equals("ADMIN")) 
    throw new ForbiddenException();
```

하지만 Spring Security를 사용하면 아래와 같이 설정 후 필터를 적용해 매번 검증을 할 필요가 없어집니다.

```java
public class JwtAuthFilter extends OncePerRequestFilter {
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res, FilterChain chain) {
        String token = req.getHeader("Authorization");
        if (jwtUtil.isValid(token)) {
            User user = jwtUtil.parseUser(token);
            var auth = new UsernamePasswordAuthenticationToken(user, null, user.getAuthorities());
            SecurityContextHolder.getContext().setAuthentication(auth);  // 보관 장소 지정
        }
        chain.doFilter(req, res);
    }
}

// 규칙 선언 -> 이 필터를 지나는 모든 요청에 대해 검사
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) {
    http.authorizeHttpRequests(auth -> auth
        .requestMatchers("/api/admin/**").hasRole("ADMIN")
        .requestMatchers("/api/public/**").permitAll()
        .anyRequest().authenticated());
    return http.build();
}

```

뒷단의 컨트롤러는 별도의 검증 로직 없이, 도달한 요청에 대해 비즈니스 로직을 수행할 수 있습니다.

### Spring Security의 세션 관리

```java
@PostMapping("/login")
public void login(@RequestBody LoginRequest req, HttpServletRequest request, HttpServletResponse response) {
    User user = userService.authenticate(req.getUsername(), req.getPassword());
    
    var auth = new UsernamePasswordAuthenticationToken(user, null, user.getAuthorities());
    SecurityContextHolder.getContext().setAuthentication(auth);
}
```

위 코드는 인증 정보인 `auth`를 스레드 로컬(`SecurityContextHolder`)에 저장합니다.

스레드 로컬에만 저장하고, 이 정보를 세션이나 쿠키에 저장하는 로직은 없습니다.

![Untitled]({{ site.baseurl }}{{ page.url }}/img//session.png)

하지만 실제 웹 브라우저에서 확인해보니 세션 ID인 `WEBAPP_SESSION`이 확인됩니다.

이 세션에 뭐가 저장되어 있는지 확인해보기 위해, 이 값을 base64로 디코딩하여 세션 저장소인 Redis에 질의해보겠습니다.

**Redis 세션 키 확인**
redis-cli -h myredis.redis.cache.windows.net -p 6380 --tls
-a "$REDIS_PASS"
HKEYS webapp:session:sessions:<디코딩된세션ID>


**Redis 응답**
"creationTime"
"lastAccessedTime"
"sessionAttr:SPRING_SECURITY_CONTEXT"
"sessionAttr:is_tester"
"maxInactiveInterval"

Spring Security가 인증 정보를 세션에 저장할 때 쓰는 고정 키인 `SPRING_SECURITY_CONTEXT`가 존재함을 확인할 수 있습니다.

소스 코드는 명시적으로 스레드 로컬에만 인증 정보를 저장하고 있었습니다.

Spring Security 공식 문서에서는 Spring Security 6+부터 세션에 자동으로 인증 정보를 저장하지 않도록 바뀌었고, 사용자가 명시적으로 인증 정보를 세션에 저장하라고 안내하고 있습니다.

https://docs.spring.io/spring-security/reference/servlet/authentication/session-management.html

### 왜 이런 동작이?

Spring Security는 내부적으로 다양한 필터를 기본적으로 제공합니다.

- SecurityContextHolderFilter (세션에서 인증 정보를 읽어와 현재 요청 스레드에 채워 넣음)
- ExceptionTranslationFilter (인증/인가 실패 시 예외를 401/403 응답으로 변환)
- AuthorizationFilter (URL별 권한 규칙(`hasRole` 등) 최종 검사)

이 필터들은 Spring Security의 기본 동작을 제공하지만, 일부는 사용자가 특정 설정을 했을 때만 켜집니다.

- SessionManagementFilter (세션 관리, 하이재킹 방지)
- OAuth2LoginAuthenticationFilter (OAuth2 인증)

여기서 SessionManagementFilter의 경우, SessionManagementConfigurer의 메서드를 사용하면 내부적으로 필터가 켜집니다.

SessionManagementFilter는 세션 하이재킹 방지라는 목적으로 사용됩니다. 그렇기에 세션에 인증 정보가 없다면 세션 ID를 바꾸고, 이걸 다시 세션에 저장하는 기능을 수행합니다.

즉, 명시적으로 저장을 하지 않아도 SessionManagementConfigurer의 메서드를 한 번이라도 사용한다면, 의도하지 않게 세션에 인증 정보가 자동으로 저장되는 상황이 됩니다.

### 명시적 저장 적용

Spring 공식 문서에서는 세션 저장이 명시적으로 되어야 한다고 언급하고 있습니다.

> *Understanding Require Explicit Save*

사용자가 세션을 아래와 같이 명시적으로 저장하는 것이 표준적인 방식입니다.

```java
@PostMapping("/login")
public void login(@RequestBody LoginRequest req, HttpServletRequest request, HttpServletResponse response) {
    User user = userService.authenticate(req.getUsername(), req.getPassword());

    var auth = new UsernamePasswordAuthenticationToken(user, null, user.getAuthorities());

    SecurityContext context = SecurityContextHolder.createEmptyContext();
    context.setAuthentication(auth);
    SecurityContextHolder.setContext(context);

    new HttpSessionSecurityContextRepository().saveContext(context, request, response); // 세션에 인증 정보 명시적 저장
}
```

이럴 경우 SessionManagementFilter는 세션에 이미 인증 정보가 있기에, 굳이 세션 ID를 바꾸지도 않으며 별도로 세션에 인증 정보를 저장하지도 않습니다.

그렇다면 현재 방식은 표준은 준수했지만 세션 하이재킹에 취약한 코드라고 볼 수도 있습니다.
(SessionManagementFilter는 로그인되는 순간에 세션 ID를 바꾸어 하이재킹 방지를 수행하는데, 이미 세션에 인증 정보가 저장되어 있다면 취약한 순간이 아니라고 판단합니다.)

### 명시적 저장 + 하이재킹 방지

Spring Security는 `AbstractAuthenticationProcessingFilter`라는, 인증 처리를 위한 특별한 클래스를 제공합니다.

이 필터는 지정한 로그인 URL(기본 `/login`)로 오는 요청만 가로채서 처리하며, 별도의 저장 호출 없이 '검증 시도 → 성공 시 ID 교체 → 저장 → 성공 핸들러 호출, 실패 시 실패 핸들러 호출'로 이어지는 로직이 이미 다 짜여 있는 상태입니다. 우리는 이걸 상속받아 내부의 `attemptAuthentication()`만 채우면 표준적이고 안정적인 로그인 코드가 완성됩니다.

```java
public class LoginFilter extends AbstractAuthenticationProcessingFilter {
    
    @Override
    protected Authentication attemptAuthentication(request, response) {
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        return authenticationManager.authenticate(token);
    }
    
    // successfulAuthentication()은 우리가 오버라이드 안 해도
    // 부모 클래스에 이미 saveContext() 호출이 구현되어 있음
}
```

별도의 `/login` 컨트롤러도 필요 없이 해당 URL로 오는 요청을 필터단에서 처리하고, `saveContext()` 또한 부모 클래스에서 처리해줍니다. (명시적 저장)

이런 흐름에서는 SessionManagementFilter 자체를 등록할 필요도 없습니다. 실제로 Spring Security 공식 문서는 이 필터를 "Moving Away From SessionManagementFilter"라는 별도 섹션으로 다루며, 이 필터에 의존하던 설정들(`sessionAuthenticationErrorUrl`, `sessionAuthenticationStrategy` 등)은 6.x에서 효과가 없거나 예외를 던지도록 바뀌었습니다. 즉 이 필터는 하위 호환을 위해 남아있는 레거시 경로에 가깝습니다.

결과적으로 표준에 따른 명시적 저장과, 보안적으로도 안전한 로그인 처리가 완성되었습니다.

### 정리

명시적으로 로직을 작성하지 않으면 의도치 않은 동작을 야기할 수 있습니다.

예를 들어 `sessionCreationPolicy(IF_REQUIRED)`는 겉으로는 단순히 서비스 내 세션 동작에 대한 기본값입니다.

하지만 뒷단에서는 SessionManagementFilter를 등록하는 트리거이므로, 최종적으로는 로그인 세션을 저장하는 역할까지 가지고 있습니다.

만약 단순 리팩토링이나 다른 설정 작업 중 해당 설정이 지워진다면, 치명적인 버그가 발생할뿐더러 원인을 찾기는 매우 어려울 것입니다.


