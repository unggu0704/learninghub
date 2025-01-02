---
author: 김규형
date: 2023-06-27 14:10:00 +0800
categories: [Study, MIT6.005-Software Construction]
tags: [sw-engineering, SW공학]
render_with_liquid: true
---

## Exceptions for signaling bugs

- 자바 프로그래밍에는 다양한 예외가 있다.
    - `ArrayIndex-OutOfBoundsExceions` 같은…
    - `NullPointerException` 또한 일반적인 예외이다.
    - `NumberFormatException` 은 Interger.parseInt 정수로 구문 분석할수 없는 문자열을 전달하는 경우의 예외도 있다.

## Exceptions for special results

- 예외는 신호적인 버그뿐만 아니라 그들은 코드의 구조를 향상시킬수 있다. 특별한 결과함께 절차적인 것을 포함하여
- 자바의 **Look up** 은 양의 정수를 기대해거나 또는 객체에 대한 null 참조를 할때 -1의 인덱스를 얻을 수 있다.
- 어느정도는 이러한 방식을 사용해도 괜찮습니다. 하지만 두가지 문제점이 있다.
    - 리턴값을 매번 확인해야한다.
    - 이걸 잊어버리기 쉽다.
- 또한 특별한 값을 찾는것은 쉽지 않다. 아래 `BirthdayBook` 을 가정해보자.
    
    ```jsx
    class BirthdayBook {
        LocalDate lookup(String name) { ... }
    }
    ```
    
- 해당 메소드에 어떤 사람의 생일인지를 고려하지 않는 경우
- 우리는 이런 경우 특별한 결과를 리턴할 수 있다. (ex: 99.9.9) 같은 경우
- 예외를 사용하는 경우
    
    ```java
    LocalDate lookup(String name) throws NotFoundException {
        ...
        if ( ...not found... )
            throw new NotFoundException();
        ...
    ```
    

```java
BirthdayBook birthdays = ...
try {
    LocalDate birthdate = birthdays.lookup("Alyssa");
    // we know Alyssa's birthday
} catch (NotFoundException nfe) {
    // her birthday was not in the birthday book
}
```

## Checked and unChecked exceptions

- 위에서 우리는 특별한 결과와 버그를 위해 예외를 사용했다.
- Checked exception은 다음과 같이 불린다.
    - `RuntimeException` 을 상속하지 않은 클래스
    - 메소드에서 예외를 체크한다면 확인된 예외를 미리 선언을 해야한다. `thowrs` 를 활용
    - 만약 다른 메소드가 호출한다면 마찬가지로 선언해야한다 그렇지 않으면 예외가 전파된다.
- 이런식으로 예외를 미리 선언하지 않으면 컴파일러가 거부한다.  ← 매우 유용
- unChecked exception 같은 경우 버그를 알리는데 사용된다.
    - `RuntimeException` 을 상속하는 클래스
- 이 예외는 아마 가장 높은 수준을 제외하고는 밑에서는 다루지 못한다.
- 우리는 모든 메소드가 낮은 콜 레밸에서 선언되져야 할것을 원하지는 않는다. 다양한 버그의 예외가
- 결과적으로 unchecked exception은 컴파일러가 체크하지 못합니다. `try - catch` 나 a `throws` 같은 선언으로
- 자바는 여전히 허용합니다 당신이 a `throws` 를 사용하는 것을 unchecked exception의 부분으로써 그러나 추천하진 않는다,.

## Throwable hierarchy

- 어떻게 자바가 예외를 결정하는지를 이해하기 위해서는 계층 구조를 봐야한다.
- `Throwable` 클래스는 thrown 하거나 caught할 수 있다.
    - 어디서 예외를 던지는지에 대한 스택 포인터의 추적
    - 예외의 optional string을 묘사한다.
- 어떤 객체가 `throw` 나 `try-cathc` 를 사용한다면 또는 선언한다면 이 클래스의 자식이 된다.
- `Error` 클래스는 `Throwable` 의 자식으로써 자바 런타임 시스템에 의해 생성된 오류가 예약되어 있다.
    - `stackoverflow` 나 `OutOfMemory` 같은…
    - `AssertionError` 같은거 또한 Error의 확장이다, 이것이 런타임이 아닌 유저코드에서 나타나는 버그임에도 불구하고 에러는 회복 불가능한걸로 고려되며. 일반적으로 발견되지 않는다
    - 간단하게 처리 할수 없는 에러!
- 확인된 예외와 확인되지 않은 예외를 구분하는 방법
    - `RuntimeException` , `Error` 이것들의 자식은 unchecked Exception이다. 컴파일러는 이것들에게 선언할것을 강요하지 않는다. 물론 호출자 또한
    - `Throwable` 과 `Exception` 의 모든 자식들은 checked Exception이다.  이런것들은 컴파일러가 `throws` 할것을 요구한다. (`RuntimeException` 의 자식이 아니라면)
- 예외를 정의할때, 서브클래스 중 하나를 선택해야한다. `RunTimeException` 또는 `Exception` 들 중에 `Error` 와 `Throwable` 는 자바 자체적으로 예약되어 있기에 하지 않아도 된다.

## Exception design considerations

- 특별한 결과에 대해선 checked exceptions을 버그에 대해서는 unchecked exception을 사용하는 규칙은 좋지만 자바의 예외는 끝이 아니다.
- 성능적인 패널티 외에도 예외에 대한 새로운 클래스를 만든느 것은 더 큰 비용을 요구한다. 만약 checked exception가 야기될수 있는 메소드일 경우 try-catch로 덮어야한다. 만약에 절대 던지지 않을 것이라고 알고 있는 경우 딜레마를 야기
- 예를 들어 큐 추상화를 디자인 할 경우
    - 큐가 비어있을 때 pop의 경우 예외를 던져야 하는건 맞다.
    - 만약 클라이언트가 예외가 아닌 대기열이 비지 않았을때만 pop하게 한다면 메소드를 사용하는 클라이언트 또한 try-catch를 던져야하는가?
- 이러한 규칙에 대해서 제시한다.
    - unchecked Exception은 버그를 알리는데에만 사용되여야 한다.
    - 또한 편리하고 저렴하게 예외를 피할 방법이 있는 경우 에만 uncheckd exception을 작성한다.
    - 그렇지 않으면 checked exception을 사용한다.
- Example
    - 큐에서 pop 할 경우 큐가 비어져있다면 `EmptyQueueException` 을 던질경우 caller 가 큐가 비었음을 먼저 테스트 해서 이를 피할수 있다.
    - `Url.getWebPage()` 같은 경우는 호출자가 예외를 피하기 쉽지 않기 때문에 checked IOException을 활용
    - `int integerSquareRoot(int x)` 깉은 경우 x가 완벽한 제곱근인지 찾는 것은 매우 어렵다 그렇기에 예외를 사용한다.

## Abuse of exceptions

- 예외 남용의 예
    
    ```java
    try {
        int i = 0;
        while (true)
            a[i++].f();
    } catch (ArrayIndexOutOfBoundsException e) { }
    ```
    
    - 이건 무한루프로써 예외를 통해 끝내는 경우
    
    ```java
    for (int i = 0; i < a.length; i++) {
        a[i].f();
    }
    ```
    
    - 이거와 완전하게 동일
- 자바의 Exception은 예외적인 상황에서 동작하게 만들었다 성능적인것은 기대하지 않는다.
- 또한 다른 버그가 숨어 있을 수 도 있다.
- 일반적인 루프를 사용시 예상되지 않은 엘 발생 할 경우 스택을 추적하여 종료한다.
- 하지만 예외기반 루프는 버그 기반 예외일 경우 일반적인 루프의 종료로 해석한다.
