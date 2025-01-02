---
author: 김규형
date: 2023-09-12 14:10:00 +0800
categories: [NHN Academy, CS]
tags: [NHN Acedemy, Java]
render_with_liquid: true
---

# Command Line Arguments(23. 10. 30)

- Java 응용 프로그램은 명령줄에서 여러 인수를 받아들일 수 있고, 이를 통해 사용자는 응용 프로그램이 시작될 때 구성 정보를 지정할 수 있다.
    
    ```java
    public static void main(String[] args) {
            for (String arg : args) {
                try {
                    int value = Integer.parseInt(arg);
                    System.out.println("int : " + value);
                } catch (NumberFormatException ignore) {
                    System.out.println("String : " + arg);
                }
            }
      }
    
    % java Main 2023 NHN Academy backend
    int : 2023
    String : NHN
    String : Academy
    String : backend
    ```
    

## 빌더형(Builder style)

빌더형은 객체를 생성하고 명령줄에서 사용 가능한 인수들을 객체에 추가하여 함으로써 구분 분석에 필요한 인수들의 묶음을 만드는 것이다.

### Apache Commons CLI

Apache Commons CLI 라이브러리는 프로그램에 전달된 명령줄 옵션을 구문 분석하기 위한 API를 제공한다. 또한 명령줄 도구에 사용할 수 있는 옵션을 자세히 설명하는 도움말 메시지를 출력할 수도 있다.

옵션 만들기는 크게 2가지로 구분할 수 있다.

- Boolean Option
- Argument Option

### 모듈 추가하기

```java
<dependency>
    <groupId>commons-cli</groupId>
    <artifactId>commons-cli</artifactId>
    <version>1.5.0</version>
</dependency>
```

### Boolean option 추가하기

**org.apache.commons.cli** 에서 제공하는 `Options` 를 적용

```java
// create Options object
Options options = new Options();

// add t option
options.addOption("v", "version", false, "print the version");
```

- `addOption` 메소드에는 두가지 매개변수가 있다.
    - 첫 번째 매개변수는 String 타입의 옵션을 나타낸다.
    - 두 번째 매개변수는 옵션에 대한 설명

### 값을 갖는 option 추가하기

```java
// create options object
Options options = new Options();

//add logfile option
Option logfile = Option.builder("logfile")
                       .argName("file")
                       .hasArg()
                       .desc("use given file for log")
                       .build();
options.addOption(logfile);
```

- 인수를 갖는 옵션은 Option 클래스의 빌더를 이용해 생성 가능
- `builder` 를 이용해 빌더 객체를 생성, 여기에 선택적으로 설정하여 원하는 옵션 생성

### 명령줄 인수 구문 분석

```java
CommandLineParser parser = new DefaultParser();
CommandLine cmd = parser.parse(options, args);
```

- CommandLineParser의 parse 메소드는 명령줄 인수를 구문 분석하는 데 사용된다.
- CommandLineParser 인터페이스 구현은 여러 가지가 있을 수 있으며 권장되는 것 중 하나는 DefaultParser이다.

### Option 값 가져오기

- command line argument에 ‘c’ 를 추가
- `options.addOption("c", true, "국가 코드");`
    - `hashArg` 가 true → 인자가 필요하다.
    - -c 10 인 경우
        - `cmd.getOptionValue()` 로 옵션을 가져올 수 있다.
- 만약 명령어에 대한 정보를 알고 싶은 `-?` 를 추가하여 설명서를 보이고 싶다면
    
    ```java
    if (cmd.hasOption("?")) {
         HelpFormatter formatter = new HelpFormatter();
         formatter.printHelp(Exam03.class.getSimpleName(), options);
     }
    ```
    
    - 이렇게 처리 할 수 있다.
