---
author: 김규형
date: 2023-06-27 14:10:00 +0800
categories: [Study, MIT6.005-Software Construction]
tags: [sw-engineering, SW공학]
render_with_liquid: true
---

### Recursive functions

  재귀적 함수가 자기 자신의 관점에 따라 정의되는 것과 같이 재귀적 자료형 또한 자기 관점에서 정의된다.

## Immutable lists

- immutable list 인 `ImList<E>` 이것이 4가지의 작동을 한다고 생각
    - *empty: void → ImList*
    - *cons: E × ImList → ImList*
    - *first: ImList → E*
    - *rest: ImList → ImList*
- 위 규칙들은 *List* 나 *Scheme* 의 기본적인 요소로 현재에서도 함수적 프로그래밍에 있어 널리 사용됨
- 이 데이터 유형을 구현하기 위해 Java 클래스를 비트 연사자를 사용한다면 아래와 같은 의미를 가진다.
    
    > 
    > 
    > 
    > *empty() = [ ]*
    > 
    > *cons(0, empty() ) = [ 0 ]*
    > 
    > *cons(0, cons(1, cons(2, empty() ) ) ) = [ 0, 1, 2 ]*
    > 
    > *x ≡ cons(0, cons(1, cons(2, empty() ) ) ) = [ 0, 1, 2 ]*
    > 
    > *first(x) = 0*
    > 
    > *rest(x) = [ 1, 2 ]*
    > 
    > *first(rest(x) ) = 1*
    > 
    > *rest(rest(x) ) = [ 2 ]*
    > 
    > *first(rest(rest(x) ) = 2*
    > 
    > *rest(rest(rest(x) ) ) = [ ]*
    > 
    > first(cons(elt, lst) ) = elt
    > 
    > rest(cons(elt, lst) ) = lst
    > 

### Immutable lists in Java

- 자바의 인터페이스 예시
    
    ```java
    public interface ImList<E> {
        // TODO: ImList<E> empty()
        public ImList<E> cons(E e);
        public E first();
        public ImList<E> rest();
    }
    ```
    
- 해당 인터페이스는 두가지 기능을 나타낸다.
    - `Empty` 빈 연산의 결과
    - `Cons` 축소 작업이 결과
    
    ```java
    public class Empty<E> implements ImList<E> {
        public Empty() {
        }
        public ImList<E> cons(E e) {
            return new Cons<>(e, this);
        }
        public E first() {
            throw new UnsupportedOperationException();
        }
        public ImList<E> rest() {
            throw new UnsupportedOperationException();
        }
    }
    ```
    
    ```java
    public class Cons<E> implements ImList<E> {
        private final E e;
        private final ImList<E> rest;
    
        public Cons(E e, ImList<E> rest) {
            this.e = e;
            this.rest = rest;
        }
        public ImList<E> cons(E e) {
            return new Cons<>(e, this);
        }
        public E first() {
            return e;
        }
        public ImList<E> rest() {
            return rest;
        }
    }
    ```
    
- `empty` 를 구현하는 방법 중에 Empty 생성자를 호출하는 방식이 있다. 하지만 이러한 방식은 독립성의 손산을 유발
- 이러한 메서드를 팩토리로 구현 → ???
    
    `List<String> z = new ArrayList<>();`
    
- 업데이트된 `ImList`
    
    ```java
    public interface ImList<E> {
        public static <E> ImList<E> empty() {
            return new Empty<>();
        }
        public ImList<E> cons(E e);
        public E first();
        public ImList<E> rest();
    }
    ```
    
- 우리는 모든 연산을 구현하였고 아래와 같은 스냅샷 다이어그램을 확인 가능.

| Java syntax | Functional syntax | Result |
| --- | --- | --- |
| ImList<Integer> nil = ImList.empty(); | nil = empty() | [ ] |
| nil.cons(0) | cons(0, nil) | [ 0 ] |
| nil.cons(2).cons(1).cons(0) | cons(0, cons(1, cons(2, nil))) | [ 0, 1, 2 ] |
| ImList<Integer> x = nil.cons(2).cons(1).cons(0); | x = cons(0, cons(1, cons(2, nil))) | [ 0, 1, 2 ] |
| x.first() | first(x) | 0 |
| x.rest() | rest(x) | [ 1, 2 ] |
| x.rest().first() | first(rest(x)) | 1 |
| x.rest().rest() | rest(rest(x)) | [ 2 ] |
| x.rest().rest().first() | first(rest(rest(x))) | 2 |
| x.rest().rest().rest() | rest(rest(rest(x))) | [ ] |
| ImList<Integer> y = x.rest().cons(4); | y = cons(4, rest(x)) | [ 4, 1, 2 ] |
- 이 구조에서 `x` 와 `y` 는 [1,2] 를 서로 공유 하고 있다는 것을 알고 있을것.

## Recursive datatype definitions

- 추상 데이터 타입의 `ImList` 는 두 구체적인 클래스 `Empty` , `Cons` 로 구성되어 있다.
- 하지만 `ImList` 또한 자기 자신을 재귀적으로 사용할 수 있다
    
    `ImList<E> = Empty + Cons(first:E, rest:ImList)`
    
- 이러한 형식의 정의는 다음과 같다.
    - 왼쪽이 추상 자료형, 오른쪽이 표현
    - 포현은 + 로 구분된 데이터의 유형의 변형으로 구성
    - 각각의 변형은 인자를 가진 constructor이다.
- 재귀형 데이터 타입의 다른 예인 이진트리
    - `Tree<E> = Empty + Node(e:E, left:Tree<E>, right:Tree<E>)`

## Functions over recursive datatypes

- 변형이 있는 추상 데이터타입의 재귀적 정의는 무한 구조 처리, 각 타입에 대한 변형의 트리 구조 성립등에 이점을 갖는다.
- 예를 들어 사이즈를 측정하는 경우
    - ***size : ImList → int***  // returns the size of the list
- 그리고 size의 크기를 정의한다.
    - size(Empty) = 0
    - size(Cons(first: E, rest: ImList)) = 1 + size(rest)
- 이러한 식을 다음과 같이 재귀적으로 볼 수 있다.

```java
size(Cons (0, Cons (1, Empty)))
 = 1 + size(Cons (1, Empty))
 = 1 + (1 + size(Empty))
 = 1 + (1 + 0)
 = 1 + 1
 = 2
```

이 정의로 자바 코드 작성

```java
public interface ImList<E> {
    // ...
    public int size();
}

public class Empty<E> implements ImList<E> {
    // ...
    public int size() { return 0; }
}

public class Cons<E> implements ImList<E> {
    // ...
    public int size() { return 1 + rest.size(); }
}
```

- 이러한 재귀적 패턴을 구현하는법
    - 추상 데이터 타입 인터페이스에서 연산을 선언
    - 각각의 구체적인 변형에서 작동 실행
- 이러한 재귀적 패턴의 몇가지 예시
    
    **isEmpty : ImList → boolean**
    
    > isEmpty(Empty) = true
    > 
    > 
    > isEmpty(Cons(first: E, rest: ImList)) = false
    > 
    
    **append: ImList × ImList → ImList**
    
    > append(Empty, list2: ImList) = list2
    > 
    > 
    > append(Cons(first: E, rest: ImList), list2: ImList) = cons(first, append(rest, list2))
    > 

### Tuning the rep

- 위의 `size()` 는 O(n) 시간이 소모  하지만 처음 계산 할때 크기 계산하는 방식으로 구성하면 O(1)
    
    ```java
    public class Cons<E> implements ImList<E> {
        private final E e;
        private final ImList<E> rest;
        private int size = 0;
        // rep invariant:
        //   e != null, rest != null, size >= 0
        //   size > 0 implies size == 1+rest.size()
    
        // ...
        public int size() {
            if (size == 0) size = 1 + rest.size();
            return size;
        }
    }
    ```
    
    - 이러한 형식은 얼핏 보면 `size` 의 변화로 mutable하게 보인다. 하지만 이러한 방식은 beneficent mutation이다.

### Rep independence and rep exposure revisited

- 우리는 이러한 클래스들을 숨겨오는것을 배웠고 pacakage-private를 활용해 더 잘 숨길수 있을것 이다.
- 오히려 `size()` 내 부에 배열을 숨기는 식으로 공간을 희생하고 더 빠른 계산의 가능

## Null vs empty

- null 을 사용하고 싶은 욕구 주의
- null 을 사용하면 매번 null 체크를 해야함
    - `if (lst != null) n = lst.size();`
- 코드가 더러워질 수가 있음
    - `n = lst.size();`

## Declared type vs. actual type

- type checking에는 컴파일 타임, 프로그램 실행 타임, 런타임 3가지로 나뉜다.
    - 컴파일 시간에는 선언된 타입을 가진다 모든 메소드에 유형을 추론하여 검사한다.
    - 런타임에는 실제 유형을 바탕으로 객체를 만든다. → 인터페이스를 선언할려고 하면 잡아낸다.

## Another example: Boolean formulas

> (P [∨](https://en.wikipedia.org/wiki/Logical_disjunction) Q) [∧](https://en.wikipedia.org/wiki/Logical_conjunction) ([¬](https://en.wikipedia.org/wiki/Logical_negation)P [∨](https://en.wikipedia.org/wiki/Logical_disjunction) R)
> 

이거를 아래와 같이 적기가 가능하다.

```php
Formula = Variable(name:String)
          + Not(formula:Formula)
          + And(left:Formula, right:Formula)
          + Or(left:Formula, right:Formula)
```

### Backtracking search with immutability

- 서로 다른 인스턴스를 관리 하기위해 불변 리스트로 시작했지만 리스트의 끝만 공유 하는 방식으로 우린 수정했다.
- backtracking 방식은 이런 목록에서 매우 우수하다.
- 단 mutable 리스트에는 이러한 백트래킹이 좋진 않다.
- 공유가 없는 immutable 또한 좋진 않다.
- 공유적인 immutable 리스트에 backtracking이 좋다는걸 기억하자.
