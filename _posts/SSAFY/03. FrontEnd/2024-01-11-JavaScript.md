---
author: "unggu"
date: 2024-03-11 12:17:32 +0800
categories: [SSAFY, FrontEnd]
tags: [SSAFY, FrontEnd, HTML, CSS, JavaScript]
render_with_liquid: true
comments: true
image:
  path: https://www.ssafy.com/swp/images/sns_img.png
---


# JavaScript (24. 03. 07)

- 프로토타입 기반의 스크립트 프로그래밍 언어로 객체 지향 개념 지원
- 웹 브라우저에서 동작하는 유일한 프로그래밍 언어
- 인터프리터 언어이다
- 명령형, 함수형, 프로토타입 기반 객체 지향 프로그래밍을 지원하는 멀티 패러다임 프로그래밍 언어이다.

## JavaScript의 사용법

- HTML에서 JavaScript를 사용하려면 `<script>` 태그를 사용
    - `src` 와 `type` 속성등을 사용하여 js를 선언

![Untitled]({{ site.baseurl }}{{ page.url }}/resources/JavaScript%20(24%2003%2007)%201a1f5b5696aa4185825d6ba0a877097e/Untitled.png)

- HTML 파일 내부의 어디서나 선언이 가능 (body 안에 권장)

### 실습 : 시계 구현

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>시계</title>
    <script type="text/javascript">
      window.onload = function () {
        setInterval(showTime, 1000);
      };

      function showTime() {
        var view = document.getElementById("view");
        var date = new Date();
        view.innerHTML = date.toLocaleTimeString();
      }
    </script>
  </head>
  <body>
    현재 시간 : <span id="view"></span>
  </body>
</html>

```

- Date 내장 객체 활용
- 1초마다 시간을 업데이트
- `<span>` 태그에 시간을 출력하기 위해 innerHTML 프로퍼티에 값을 할당

### 변수

- 타입이 없다 `var keyword` 를 사용해 선언
    - 동적 타입의 언어 → 자동으로 타입이 결정됨

### 자료형

![Untitled]({{ site.baseurl }}{{ page.url }}/resources/JavaScript%20(24%2003%2007)%201a1f5b5696aa4185825d6ba0a877097e/Untitled%201.png)

- `undefined` : 값이 초기화 되지 않음
- 문자열의 경우 ` 을 이용한 경우 $ {} 를 통해 변수 표현 가능

### 변수 호이스팅

- var 키워드를 사용한 변수는 중복해서 선언이 가능
- 선언되기 이전에 참조가 가능하다.

## 객체

- js의 객체는 키와 값으로 구성된 프로퍼티들의 집합
- 객체의 참조는 (.)을 사용하거나 복사되지 않고 참조되는 점은 자바와 비슷하다.

## DOM 사용하기

> Document Object Model의 약자
> 
- 문서 객체
- HTML같은 문서를 JS가 이용 할 수 있는 객체로 만들면 → 문서 객체라고 함
- 넓은 의미로 웹 브라우저악 HTML을 인식하는 방식이라고 함