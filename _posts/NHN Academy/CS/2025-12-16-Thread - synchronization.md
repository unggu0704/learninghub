---
author: 김규형
date: 2023-09-12 14:10:00 +0800
categories: [NHN Academy, CS]
tags: [CS, NHN Acedemy, Thread]
render_with_liquid: true
---


# Thread - synchronization(23. 10. 18)

## Thread 멈추기

- `Thread()` 는 `run()` 에 의해 시작되지만 종료 명령어는 없다.
- 과거에는 `stop()` 이 있었지만 데드락, 내부 리소스 정리 등 문제로 삭제
- Thread의 안전한 종료를 위해 안전하게 스스로 종료할 수 있게 구현

### 상태 제어 이용하기

- Thread 안에서 내부 상태 제어가 가능하다
- 안전하게 종료하는법
    - `flag` 변수로 제어하기
        
        ```java
        boolean runnuingflag = true;
        
            RunableCounter(String str, int counter) {
                this.str = str;
                this.maxCount = counter;
            }
        
            @Override
            public void run() {
                while (runnuingflag) {
                    try {
                        if (counter == 59)
        						...
        				}
        	}
        ```
        
    - interrupt() 를 활용
        
        ```java
        public void run() {
                while (!Thread.currentThread().isInterrupted()) {
                    try {      
                        System.out.println(str + " : " + counter++);
        								...
        			}
        }
        ```
        

### Thread 동시성 제어

- 하나의 Process내에서 자원을 공유하는 여러 스레드의 문제점은 어떤게 있을까
- **Race Condition**
    - 둘 이상의 thread가 동시에 공유 자원에 접근 할 시 발생
    - `[SharedCounter.java](http://SharedCounter.java)`
        
        ```java
        public class SharedCounter extends Thread {
            SharedCount sharedCount;
            int count;
            int maxCount;
        
            public SharedCounter(String name, int maxCount, SharedCount sharedCount) {
                setName(name);
                this.sharedCount = sharedCount;
                this.maxCount = maxCount;
                count = 0;
            }
        
            @Override
            public void run() {
                while (count < maxCount) {
                    count++;
                    sharedCount.increment();
                }
            }
        }
        ```
        
        ```java
        public class SharedCount {
            int count;
        
            public int getCount() {
                return count;
            }
        
            public void setCount(int count) {
                this.count = count;
            }
        
            public void increment() {
                setCount(getCount() + 1);
            }
        }
        ```
        
        ```java
        SharedCount sharedCount = new SharedCount();
                SharedCounter counter1 = new SharedCounter("counter1", 10000, sharedCount);
                SharedCounter counter2 = new SharedCounter("counter2", 10000, sharedCount);
        
                counter1.start();
                counter2.start();
        
        sharedCount : 14047
        ...
        sharedCount : 14375
        ```
        
    - 결과는 20000을 예상하지만 실행할때마다 다른 결과가 나타난다.
    - counter1 이 1을 추가하고 메모리에 적었지만 counter2 가 곧이어 다시 써버려서 counter1이 덮어진다
    - 이러한 구역을 **임계 구역** 이라고 부른다.
    - 이러한 문제를 해결하기 위해 **상호 배제** 를 사용 → 공유 자원의 접근을 나의 process나 thread로 제한
- **Deadlock**
    - 공유 자원의 접근 권한 등으로 통제하던 도중 반환 관계가 꼬임으로써 발생
    - **Hold and Wait**
        - P1이 R1 을 기다리고 있는 도중 P2가 R1를 실행하던 도중 R2도 사용할 필요가 있음 하지만 P3이 R3을 사용하고 있을 경우… R1은 언제 P1을 사용할 수 있을까…
    - **Circular Wait**
        - 위의 Hold and Wait가 순환 형태를 이루고 있는 경우
        - 식사하는 철학자가 대표적인 문제
    - **Starvation**
        - 다른 process나 thread가 지속적으로 한 resource를 가짐으로써 다른 thread가 계속해서 접근 권한을 얻지 못할 경우에 발생 → 무한한 대기 상태
    - **Licelock**
        - 접근이 없는 권한의 경우 해당 자원 대신에 다른 자원을 수행하고 있는 것
        - 해당 작업만 처리되지 못하고 나머지 작업은 처리 되는 것의 차이를 가지고 있다.

### Synchronized

Java에서 thread 동기화를 위해 synchronized 적용 가능한 곳

- Instance methods
- Static methods
- Code blocks

위의 SharedCount.java에 `Synchronized` 키워드를 적용

```
synchronized public void increment() {
        setCount(getCount() + 1);
    }

sharedCount : 20000
```

- 정상적으로 sharedCount가 나타남을 볼 수 있다.
- `static` 을 사용해서 싱글톤 패턴 흉내
    
    ```java
    public class SharedCount {
        static int count;
    
        public static int getCount() {
            return count;
        }
    
        public static void setCount(int count) {
            SharedCount.count = count;
        }
    
        static synchronized public void increment() {
            setCount(getCount() + 1);
        }
    }
    ```
    
    thread safety가 더 강화됨을 확인할 수 있다.
    

### Synchronization Control

- 이러한 병행성의 제어를 위해 `wait` 과 `notify` 지원
- *wait()*
    - 어떠한 thread가 자신의 제어권을 양보하고 WATING 상태에서 대기하는 명령어
    - notify() 를 호출함으로써 RUNNABLE 상태로 변경
- *notify()*
    - WAITING 상태에 있는 영역을 RUNNABLE로 변경 하는 역할

### 멀티 스레드 환경에서는

- Java는 multi-thread 언어이므로, 이러한 동기화는 상호배제를 달성하는 좋은 방법이다.
- 변수에는 synchronized 를 적용할 수 없다.
- synchronized block의 object가 null인 경우는 null point exception을 발생
- Java 동기화는 동시 읽기를 허용하지 않는다.
- synchronized method는 매우 느리다.
