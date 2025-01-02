---
author: 김규형
date: 2023-09-11 14:10:00 +0800
categories: [NHN Academy, ETC]
tags: [NHN Acedemy]
render_with_liquid: true
---

# Junit 5, Package, jar…(23.08.30)

- 자바 프로그래밍 언어용 유닛 테스트 프레임 워크
- 왜 Junit을 사용하는가?
    - 함수 및 메소드가 의도한 대로 잘 동작하는지 검증이 필요함
    - 단위 테스트를 통해 안전성을 높일 수 있음
- 대표적인 사용 예시
    
    ```java
    public class Junit5Test {
        @Test
        @DisplayName("DisplayName 테스트")
            // DisplayName 어노테이션을 통해서 더 우아하게 생성 가능
        void test_문자열_출력_테스트() { //test 이름을 정하는게 중요하다.
            System.out.println("test");
        }
    }
    ```
    
- 테스트의 순서
    
    ```java
    package NHN_weeks_1;
    
    import org.junit.jupiter.api.Test;
    
    public class Junit5Test {
        @Test
        void one() {
            System.out.println("1");
        }
    
        @Test
        void two() {
            System.out.println("2");
        }
    
        @Test
        void three() {
            System.out.println("3");
        }
    
        @Test
        void four() {
            System.out.println("4");
        }
    
        @Test
        void five() {
            System.out.println("5");
        }
    }
    ```
    
    해당 코드의 순서는 위에서 아래가 아니다 
    
    `@Order` 를 사용해서 순서를 조절 가능 
    

### 다양한 Annoations들

- `@Test`
    - 테스트 코드 실행 가능
- `@DisplayName`
    - 테스트 이름 지정
- Assertions.assertTrue()
- Assertions.assertFalse()
- Assertions.assertArrayEquals()
- Assertions.assertInstanceOf()
- Assertions.assertEquals()
- Assertions.assertThrows()

## Assertions

- Assertions를 활용하여 테스트 코드를 작성 하는 법 예시
    
    **Dice** 
    
    ```java
    public class Dice {
        private int number;
    
        Dice(int n) {
            if (n <= 0 || n >= 7) {
                throw new IllegalArgumentException("유효하지 않은 값");
            }
            this.number = n;
        }
    
        public int getNumber() {
            return number;
        }
    }
    ```
    
    **DiceTest**
    
    ```java
    public class DiceTest {
        @Test
        @DisplayName("Dice 객체 생성 테스트 ")
        void diceCreat() {
            Dice dice = new Dice(4);
    
            Assertions.assertEquals(4, dice.getNumber());
        }
    
        @Test
        @DisplayName("Dice 객체 생성 반복 테스트 ")
        void DiceIteratorTest() {
            int[] arr = {1, 2, 3, 4, 5, 6};
    
            for (int i = 0; i < arr.length; i++) {
                Dice dice = new Dice(arr[i]);
                Assertions.assertEquals(arr[i], dice.getNumber());
            }
        }
    
    		@ParameterizedTest // 파라미터 주입 
        @ValueSource(ints = {1, 2, 3, 4, 5, 6})
        @DisplayName("Dice 객체 Param 테스트")
        void diceParamTest(int number) {
            Dice dice = new Dice(number);
            Assertions.assertEquals(number, dice.getNumber());
        } // ints의 모든 인덱스에 대해 테스트 실행 
    
        @Test
        @DisplayName("Dice 객체 생성 -1 입력 테스트 ")
        void negativeDiceCreate() {
            Assertions.assertThrows(IllegalArgumentException.class,
                    () -> new Dice(-1)); // 예상되는 예외를 받는지 확인
        }
    
    }
    ```
    
    `Assertions.assertEquals` 를 통해 생성자를 통해 만들어지는 input되는 값과 객체의 멤버 변수 비교 
    
    `Assertions.*assertThrows`* 를 통해 음수가 들어갈시 적절한 예외를 내보내는지 확인 
    
    `@ParameterizedTest` 와  `@test` 는 같은 뜻을 가진 어노에이션 중복해서 쓰지 않게 주의 
    
     - 스트림을 사용한다면 더욱 효율적인 테스트 케이스 생성이 가능 하다.
    
- 왜 테스트 코드를 작성하고 실행하는 이유?
    - 테스트 코드를 통해 대규모 코드의 안전성을 높일 수 있다. (에러 확률 감소)
    - 코드를 작성하기 전에 테스트 코드를 먼저 작성하면 더 좋을수도?
    - 코딩 테스트에서 TC 채점 할떄 이런식으로 하는 걸까 생각…
    

# Package

## Package란?

- class들의 모음

## 고유성

- package를 사용하는 이유는 클래스명의 고유성을 보장하기 위해서 사용
- 만약 다른 java프로그래머가 협업과정에서 서로 다른 목적을 가진 클래스를 생성 했는데 이름이 같다고 가정할 때, 서로 다른 패키지를 사용함으로써 충돌을 피할 수 있다.

## 최상위 패키지명은 도메인

- 최상위 패키지명에 도메인 주소를 앞뒤로 바꿔서 사용
- 이미 인터넷 도메인은 고유성을 가지고 있음으로 충돌에 대한 걱정할 필요가 없음
- 일종의 관례
    
    `com.nhnacademy.study ...`
    
    ex)
    
    ```
    package com.google.gson; // 도메인 뒤집어서 사용해서 고유성을 보장 
    
    public final class Gson {
     //...
    }
    ```
    

### 폴더형식의 package 구조

- Springframework 페키지 구조
- 폴더구조형태로 계층적으로 구성되어 있음
- org.springframework.spring-core

# jar

- java에서 제공하는 압축 파일, `zip 압축`
- `class 파일`, `resource(텍스트,이미지)`, .. 메타데이터를 하나로 모아서 Java 플랫폼에 응용소프트웨어나 라이브러리를 배포하기위한 소프트애어 패키지 파일
- 컴파일된 class 파일을 jar 묶어서 배포하면 , 경로나 파일의 위치에 상관없이 프로그램 실행 가능
- jar 파일 생성
    
    `jar --create --file  hello.jar --main-class com.nhnacademy.hello.Main -c ./com`
    
    → `--main-class com.nhnacademy.hello.Main` main 함수가 다수 존재 할떄 main의 위치를 지정 
    
    → ./com 가장 루트 디렉 토리 
    
    → `hello.jar` 라는 실행 가능한 파일을 생성 
    
    → `unzip` 을 활용하여 다시 압축 해제도 가능하다.
