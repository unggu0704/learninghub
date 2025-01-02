---
author: 김규형
date: 2023-06-27 14:10:00 +0800
categories: [Study, MIT6.005-Software Construction]
tags: [sw-engineering, SW공학]
render_with_liquid: true
---

## Specifications란 무엇인가?

- 심각한 버그는 인터페이스의 동작에 대한 오해에서 나타난다.
- 정확한 명세를 활용해 비난을 사람이 아닌 코드로 옮겨야한다.
- BigIntefer에 대한 명세와 코드
    
    ```java
    /*
     * 명세
     */
    
    public BigInteger add(BigInteger val)
    
    Returns a BigInteger whose value is (this + val).
    
    Parameters: 
    val - value to be added to this BigInteger.
    
    Returns: 
    this + val
    ```
    
    ```java
    /*
     * 실제 코드 
     */
    if (val.signum == 0)
        return this;
    if (signum == 0)
        return val;
    if (val.signum == signum)
        return new BigInteger(add(mag, val.mag), signum);
    
    int cmp = compareMagnitude(val);
    if (cmp == 0)
        return ZERO;
    int[] resultMag = (cmp > 0 ? subtract(mag, val.mag)
                       : subtract(val.mag, mag));
    resultMag = trustedStripLeadingZeroInts(resultMag);
    
    return new BigInteger(resultMag, cmp == signum ? 1 : -1);
    ```
    
- Specifications로써의 `BigInteger.add` 는 일반적인 사용자가 이해하기 쉽게 되어 있다.
- Source Code로 써의 `BingInteger.add` 는 코너케이스에 대한 질문에 적절한 읽을 수있는 문서를 제공한다.
- *Specifications를 사용함으로써 구현에 있어서 변화할 때 클라이언트에게 알리지 않고도 변경이 가능하다. → ?*
- 코드를 더 빠르게 만들게 도와주기도 하는데 Specifications는 더 효율적인 구현을 제공하기도 한다.
- precondition은 특정 범위에 대해 필요 없는 체크를 하지 않게 해준다.
- *Specifications = contract acts* 라고도 불린다. 고객과 설계자와의 방화벽으로써 클라이언트가 작업과 unit의 자세한 부분까지 알게 할 필요가 없다. Specifications가 있다만 고객에게 소스코드를 읽게할 필요가 없다는 뜻
- 구현하는 사람은 장치의 세부 사항으로부터 보호한다.
- 또한 고객으로부터 어떻게 사용할지 물어보는 대신에 명세를 확인한다.
- 이러한 벽은 결과적으로 unit의 코드와 고객의 코드를 독립적으로 변경할 수 있게 해준다.

## Behavioral equivalence

```java
static int findFirst(int[] arr, int val) {
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] == val) return i;
    }
    return arr.length;
}

static int findLast(int[] arr, int val) {
    for (int i = arr.length -1 ; i >= 0; i--) {
        if (arr[i] == val) return i;
    }
    return -1;
}
```

- 위 두 코드는 다르다.  하지만 behavior equivalence에 대한 우리의 질문은 하나의 구현을 다른걸로 대체가 가능한지 에대한 문제이다.
- 단순히 코드가 다른걸 제외하고도 return값에 있어 또한 차이가 있다.
- 투 메소느는 같은 행동을 하지만 고객이 절대 다른 case에 있어서 의존하지 않게 할 수 있다.
- 결국 같은 val과 같은 요소를 출력하기에 서로가 서로로 전환 가능
- 동등성은 고객의 눈으로 봐야한다.
- 한 구현에 있어 대체제를 하기 위해 그리고 그것을 수용하기 위해 우리는 Soecification(명세)에 의존한다.

## Specification structure

- 메소드 명세는 다음과 같은 항으로 구성된다.
    - Precondition : keyword가 무엇을 요구하는지 나타내는것, 고객의 의무 어떤 값을 넣을지
    - Postcondition: keyword가 어떤 영향에 대해 나타내는것, 구현자의 의무 적절한 값을 반환
- precondition이 고정되어 있다면 postcondition도 고정되어 있어야한다.
- 반대로 precondition이 고정되어 있지 않다면 구현은 postcondition에 구속되지 않는다. 예외를 던지거다 임의의 결과를 반환하는 등의 자유가 있다.

> **Precondition** : 입력 조건, 함수가 Call 되기 전에 무엇이 참이 되어짐을 나타냄
> 

> **Postcondition** : 출력 조건, 함수가 실행되고 무엇이 참이 되어져야하는지를 나타냄
> 

### Specifications in Java

- 일부 언어는 전제조건과 사후 조건을 언어의 기본적인 부분으로 포함
- 자바의 Static type은 메소드의 전제조건과 사후조건의 일부, 나머지는 주석으로 기술
- 이런 주석을 잘 활용하기 위해 javadoc의 규칙을 준수
    
    ```java
    /**
     * Find a value in an array.
     * @param arr array to search, requires that val occurs exactly once
     *            in arr
     * @param val value to search for
     * @return index i such that arr[i] = val
     */
    static int find(int[] arr, int val)
    ```
    

## Null reference

- 자바의 객체나 배열은 null 값을 가질수 있다. 즉 참조가 언제나 객체를 가지지 않는다. 자바에서는 이러한 null을 주의해야한다.
- 원시형 타입은 null이 가능하지 않다.
- 비 원시형에는 뭐든지 null이 가능
- 컴파일러는 null을 찾지 못한다. 그렇기에 런타임 시간에 에러가 발생
- null과 빈 문자열은 같지 않다.
- null 값은 불안전하기에 문법적 디자인으로 추천하지 않는다.
- null 값은 암묵적으로 허용되지 않습니다 return values과 파라미터로써 null을 가지는것은
- 그렇기에 모든 전제조건은 null이 아닌것을 암묵적으로 가진다.
- 이는 출력조건 또한 non-null임을 가진다.
- null을 파라미터로 하고싶은 경우 명시적으로 명시해야하는데… null은 피하는게 좋다.

## What a specification may talk about

- 메소드의 명세는 파라미터나 return값에 대한 이야기는 가능하지만 지역변수와 private fields 같은 명세에 보여질 필요가 없을것
- 독자가 spec에서 자바 코드를 읽을수 없는등 이용할수가 없을수도 있습니다
- 자바독이 읽을수 있는 HTML로 코드에서 추출하기 때문에 메소드의 소스코드는 굳이 이용하지 않아도 된다.

## Testing and specifications

- 명세에만 염두를 두고 진행하는 테스트를 블랙박스 테스트라고 한다.
- 화이트박스 테스트는 실제 기능을 보고 구현을 하지만, 화이트 박스도 명세를 따른다.
- 물론 구현보다 더 강력한 경우를 고려할수도 있다.
- 해당 명세를 보면 index는 배열에서 가장 낮은 val과 같은 값만 추출되어진다. → weak postcondition

```java
static int find(int[] arr, int val)
  requires: val occurs in arr
  effects:  returns index i such that arr[i] = val
```

- 이런 테스트코드는…?

```java
int[] array = new int[] { 7, 7, 7 };
~~assertEquals(0, find(array, 7));~~  // bad test case: violates the spec
assertEquals(7, array[find(array, 7)]);  // correct
```

- find 메소드를 val을 찾을수 없을때 특정값을 return 하는게 아닌 예외를 던지게하는다면  precondition을 위반하기 때문에 find는 호출되지 못하고 동작을 테스트하지 못한다.
- 왜 화이트박스 테스트가 의미가 있는지는 명세를 넘어서지 못한는 경우에 새로운 테스트를 찾을수가 없다. 그렇기에 구현 독릭접인 방식으로 여전히 테스트는 실행되어야한다.

### Testing units

- Testing 단원에서 학습한 코드 예시
    
    ```java
    /** @return the contents of the web page downloaded from url */
    public static String getWebPage(URL url) { ... }
    
    /** @return the words in string s, in the order they appear,
     *          where a word is a contiguous sequence of
     *          non-whitespace and non-punctuation characters */
    public static List<String> extractWords(String s) { ... }
    
    /** @return an index mapping a word to the set of URLs
     *          containing that word, for all webpages in the input set */
    public static Map<String, Set<URL>> makeIndex(Set<URL> urls) { 
        ...
        calls getWebPage and extractWords
        ...
    }
    ```
    
- unit testing에 대해 이야기 할 때, 우리의 프로그램에 고립적으로 각각의 모듈에 대해 테스트를 작성하였다.
- 좋은 unit test는 작은 명세에 초첨을 맞춘다.

## Specifications for mutating methods

- 이전의 가변성과 불변성에 대해 이야기했다.
- 우리의 `find` 명세는 고려하지 않아다 어떻게 영향력을 미치는지 데이터를 변경할때 사후조건에서
- 가변 객체에 대한 명세는 다음과 같다.
    
    ```java
    static boolean addAll(List<T> list1, List<T> list2)
      requires: list1 != list2
      effects:  modifies list1 by adding the elements of list2 to the end of
                  it, and returns true if list1 changed as a result of call
    ```
    
- 자바 List interface를 단순화 한 예시 위
- 사후조건의 경우 두가지 제약이 있다.
    - `list1` 이 어떻게 수정되지는지
    - return값이 어떻게 결정되어지는지
- 전제조건을 본다면, 이것은 우리에게 말해줍니다.  리스트에 요소를 자체에 추가할려고 한다면 동작이 정의되지 않음을 알 수 있다. 너는 쉽게 상상할수 있습니다 왜 메소드의 구현자가 이 제약을 원하는지 .
- 이러한 방식은 구현이 쉬워지고 메소드의 유용한 것을 배제할 가능성이 낮아진다.
- 암묵적인 전제 조건인 not null을 적용
