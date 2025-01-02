---
author: 김규형
date: 2023-06-27 14:10:00 +0800
categories: [Study, MIT6.005-Software Construction]
tags: [sw-engineering, SW공학]
render_with_liquid: true
---

- Objects
    - 체계적인 디버깅
    - 디버깅이 선택되어 질 때가 있다. (전체 시스템이 함께 연결되거나, 배포 후에 사용자 보고)
    - 버그가 현지화가 불가능 할때에는 디버깅이 좋은 전략이 될 수도 있다.

## Reproduce the Bug

- 고장을 야기하는 테스트케이스를 반복해서 시도한다.
- regression tesing을 통해 버그를 찾는데 성공 했다면, 운이 좋은편
- 사용자가 버그를 제보 했다면 이에 반해 약간의 노력이 필요하다.
- 하지만 이 환경이 그래픽 사용 인터페이스 또는 멀티 스레드 환경이라면 버그를 찾는 것은 더 어려워진다.
- 물론 작은 테스트 케이스를 만들어 반복적으로 실행하는 것은 도움이 된다.
- 반복해서 실행 하고 버그를 찾아 수정하고 이를 통해 regression test에 추가하는 것은 추후 이러한 버그를 지속해서 방지할 수 있기 때문이다.
- 다음과 같은 예시를 보자
    
    ```java
    /**
     * Find the most common word in a string.
     * @param text string containing zero or more words, where a word
     *     is a string of alphanumeric characters bounded by nonalphanumerics.
     * @return a word that occurs maximally often in text, ignoring alphabetic case.
     */
    public static String mostCommonWord(String text) {
        ...
    }
    ```
    
- 유저에게 세익스피어의 텍스트를 전달하는 메소드이다.
- `"the"` 나`"a"` 를 반환 하는 대신에 `"e"` 를 반환 해버리는 문제가 발생한다면?
- 세익스피어는 100,000 줄과 800.000개의 단어가 있다.
- 일반적인 디버깅 방식으로는 매우 디버깅 난이도가 높음
- 만약에 버그가 작은 범위에서도 유의미하게 발생한다면..?
    - 세익스피어의 전반부에서도 발생하는가?
    - 연극 하나에도 같은 버그가 발생하는가?
    - 하나의 문장에서도 같은 버그가 발생하는가?
- 작은 테스트 케이스를 찾았다면 이를 통해 수정한 후 전체에서도 같은 버그가 발생하는지 확인한다!

## ****Understand the Location and Cause of the Bug****

- 버그를 지역화 하는데 사용할 수 있는 과학적 방법
    - **Study the data :** 버그를 야기시키는 테스트 케이스를 확인하고 잘못된 결과로부터 재추적
    - **Hypothesize :** 버그가 어디에서 발생할 수 있는지, 어디서 발생할 일은 없는지에 대한 데이터에 기반한 가설을 세운다.
    - **Experiment :** 가설을 검증하는 실험을 생각한다. 프로그램을 최소한으로 방해하지 않으면서 정보를 수집한 뒤 이를 관찰한다.
    - **Repeat :** 실험에서 데이터를 수집하고 이를 바탕으로 새로운 가설을 만든다. 반복을 통해 버그가 발생할 곳을 점차적으로 줄여 나간다.
- 이러한 방식을 바탕으로 다음과 같은 예시
    
    ```java
    /**
     * Find the most common word in a string.
     * @param text string containing zero or more words,
     *     where a word is a string of alphanumeric
     *     characters bounded by nonalphanumerics.
     * @return a word that occurs maximally often in text,
     *         ignoring alphabetic case.
     */
    public static String mostCommonWord(String text) {
        ... words = splitIntoWords(text); ...
        ... frequencies = countOccurrences(words); ...
        ... winner = findMostCommon(frequencies); ...
        ... return winner;
    }
    
    /** Split a string into words ... */
    private static List<String> splitIntoWords(String text) {
        ...
    }
    
    /** Count how many times each word appears ... */
    private static Map<String,Integer> countOccurrences(List<String> words) {
        ...
    }
    
    /** Find the word with the highest frequency count ... */
    private static String findMostCommon(Map<String,Integer> frequencies) {
        ...
    }
    ```
    

### Study the Data

- Data의 중요한 것 중 하나는 exceoption으로 부터 발생한 Stack trace이다.
- Stack trace하는 능력을 연습한다면 버그를 찾을 수 있는 엄청난 능력을 갖게 되는것
- 작은 테스트 케이스를 조정하는 사이에 특정 입력에 대한 버그의 발생 차이를 알게 될 수도 있다.
    - `mostCommonWords("c c, b")` → 버그 발생
    - `mostCommonWords("c c b")` → 버그가 발생하지 않음

### ****Hypothesize****

- 프로그램을 모듈로써 생각하고 알고리즘의 단계로 생각하는 것은 프로그램의 나머지 전체를 배제하는데 도움이 된다.
- 전체적인 데이터의 흐름
    
    - 버그의 증상이 `countOccurrences()` 의 Exception인 경우 `findMostFrequent()` 가 배제 가능하다.
    - 그렇다면 `splitIntoWords()` 에 버그가 있다고 가정하고 실험을 진행해볼 수도 있다.
    - 만약 위의 가정이 true라면 `countOccurrences()` 또한 배제가 가능하다.
    - 만약 false 라면 `splitIntoWords()` 를 배제 가능하다.

### Experiment

- 좋은 experiment는 최대한 시스템을 방해하지 않고 진행한다.
    - Run a **different test case** : 윗 단계에서 진행 한것이 테스트 케이스를 감소시키는 과정
    - Insert a **print statement** or **assertion :** 프로그램 실행중에 내부 상태를 체크
    - Set a **breakpoint :** 중단점을 설정해 코드를 한 단계씩 체크(내부 변수 값, 개체 값)
- 증명 없이 가설에 기반한 수정을 사용하는 것은 매우 매력적이다
- 하지만 이러한 시도는 결과적으로 이해하기 어려운 코드를 생성하는 원인이 된다.
- 설사 버그가 수정되어 진 것처럼 보이더라도 이는 전체적인 수정이 아니라 버그가 단순히 숨었을 수도 있다.
- `ArrayOutOfBoundsException` 의 버그 발생 시 실제 문제를 해결하지 않고 단순히 exception을 피할 수 있는 코드만을 추가하는 것과 비슷한 문제

### Other tips

- **Bug localization by binary search**
    - 버그를 찾는것 또한 결과적으로 search라고 볼 수 있다.
    - binary search를 사용한다면 이러한 속도를 높일 수 있다.
    - `mostCommonWords` 를 예로 든다면 workflow를 절반으로 나누고 버그가 발생 되는 부분을 다시 한번 반으로 나누는 식으로 속도를 증가 시킬 수 있다.
- **Prioritize your hypotheses**
    - 가설의 우선순위를 세우는 것은 중요하다
    - 테스트가 오래 진행된 예전의 코드에 비해 새로 추가된 코드에서 버그가 발생 할 확률이 높다.
    - 또한 자바 라이브러리는 일반적으로 사용자가 적는 코드 보다는 신뢰성이 높다
    - 마찬가지로 자바의 컴파일러, 운영체제, 하드웨어 등도 버그의 가능성은 낮다.
- **Swap components**
    - 모듈을 만족하는 다른 구현이 있다면 swap 또한 좋은 시도가 될 수 있다.
    - 예를 들어 binary search()에서 문제가 발생했다면 단순한 linear search()로 대체도 가능하다.
    - 마찬가지로 java.util.ArrayList가 문제가 발생한 경우 java.util.LinkedList로 대체도 가능하다.
    - 여전히 문제가 발생한다면 Java 버전을 변경해보고, OS 자체가 의심된다면 다른 OS 환경에서도 테스트 해볼 수 있다.
    - 하지만 이러한 방식은 많은 시간을 요구하기에 충분한 이유가 있어야하다.
- **Make sure your source code and object code are up to date**
    - git 등을 통해 최신 버전을 새로 받아오고 새로 컴파일한다
    - `Clean`  명령어도 비슷한 기능을 실행한다.
- **Get help**
    - 다른 사람에게 문제를 설명하고 도움을 요청해본다.
- **Sleep on it**
    - 자고 내일 다시 본다.

## ****Fix the Bug****

- 버그를 발견한다면 어떻게 해결해야할까
- 단순히 빨리 패치를 하지 않고 버그가 코딩 에러인지 잘못된 오타인지 생각해보자
- 디자인 에러, 불충분한 인터페이스 등등 다른 문제가 있을 수도 있다는걸 명심하자
- 또한 비슷한 버그가 있는지도 항상 명심하자
- 이러한 방법은 미래의 버그를 방지할 수 있는 기회를 제공한다.
- 마지막으로 회귀 테스트를 통해 해당 버그가 다음에 나타나지 않도록 주의한다.

## Summary

- 효율적인 디버깅을 위해서는
    - 버그를 회귀테스트 케이스에 추가
    - 과학적 방법 및 신중한 버그 수정
