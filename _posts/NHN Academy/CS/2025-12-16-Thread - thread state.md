---
author: 김규형
date: 2023-09-12 14:10:00 +0800
categories: [NHN Academy, CS]
tags: [CS, NHN Acedemy, Thread]
render_with_liquid: true
---


# Thread - Thread State (23. 10. 19)

## Thread의 상태 6가지

| 상태 | 설명 |
| --- | --- |
| NEW | thread가 실행 준비를 완료한 상태로, start() 함수를 호출하기 전 상태 |
| RUNNABLE | start()가 호출되어 실행될 수 있는 상태 |
| RUNNING | 해당 thread가 동작중인 상태 |
| WAITING | 다른 thread가 통지할 때까지 기다리는 상태 |
| TIMED_WAITING | 정해진 시간 동안 기다리는 상태 |
| BLOCKED | 사용하고자 하는 객체의 잠금(lock)이 풀릴 때까지 대기하는 상태 |
| TERMINATED | 실행이 종료된 상태 |
- **new**

새로운 Thread 객체가 생성 되었지만 실행되기전

```java
Thread thread = new Thread();
System.out.println(thread.getState());

NEW
```

- **RUNNABLE**

언제든지 실행준비가 된 thread

r을 `start()` 시키고 `getState()` 한다.

```java
RunableCounter r = new RunableCounter("ddd", 5);
r.start();
System.out.println(r.getThread().getState());

RUNNABLE
ddd : 1
ddd : 2
ddd : 3
ddd : 4
...
```

- **RUNNUNG**

한 thread가 RUNNING이라면 다른 thread는 RUNNABLE, WATING등을 실행이 아닌 상태를 갖는다.

- **BLOCKED**

Synchronized block을 다른 thread가 점유하고 있는 경우 기다리고 있는 상태

- **TERMINATED**

Thread가 종료된 상태

- **TIMED_WAITING**

WAITING과 동일하지만, 제한시간 설정이 가능

### BLOCKED vs WATING

| WAITING | BLOCKED |
| --- | --- |
| thread는 wait() 또는 join()을 호출할 때 WAINTING 상태가 된다. thread는 다른 thread가 notify() 또는 notifyAll()을 호출할 때까지 WAITING 상태를 유지한다. | thread는 다른 thread에서 알림을 받았지만, 아직 object 잠금이 설정되지 않은 경우 이 상태가 된다. |
| WAITING thread는 다른 thread의 알림을 기다리고 있다. | BLOCKED thread는 다른 thread가 잠금을 해제하기를 기다리고 있다. |
| WAITING thread가 중단될 수 있다. | BLOCKED thread는 중단될 수 없다 |

### Thread 상태 제어

- interrupt() :  *WAITING → RUNNABLE*
- notify() : *wait() 에 의한 WAITING → RUNNABLE*
- sleep() : 주어진 시간동안 thread를 TIMED_WAITING
- join() : join()을 요청할 thread가 종료 할 때까지 대기
    
    ```java
    counter1.start();
    counter2.start();
    counter1.getThread().join();
    counter2.getThread().join();
    ```
    
- wait() : Synchonized block에서 thread를 WAITING 상태로 만든다.
