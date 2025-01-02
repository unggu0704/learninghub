---
author: 김규형
date: 2023-09-12 14:10:00 +0800
categories: [NHN Academy, CS]
tags: [CS, NHN Acedemy, Thread]
render_with_liquid: true
---

# Thread - pool, daemon (23. 10. 19)

## Thread Pool

- 자바 thread pool은 여러개의 thread를 만들어 놓고 재사용되는 thread 그룹을 말한다.
- task가 있다면 pool 에 넣고 모든 pool의 thread가 가득 찼다면 빈 thread가 생길 떄까지 대기한다.
- Thread pool 생성 및 실행
    
    `ExecutorService threadpool = Executors.newCachedThreadPool();`
    
    `threadpool.execute(w);`
    

### Thread Pool의 장점

- Thread 생성과 삭제에 따른 시간과 자원을 줄일 수 있음

### Thread Pool 사용 시 주의할 점

- Deallock
    - 다양한 thread가 돌아가기에 교착 상태 발생 가능성
- Thread Leakage
    - 비정상 종료에 의한 종료
- Resource Thrasing
    - 너무 큰 thread pool의 경우 context 전환 비용 증가

### Thread Group

- Java는 단일 Object에서 여러 thread를 그룹화 가능
- group 전체를 pasue, resume 가능
    
    ```java
    ThreadGroup threadGroup = new ThreadGroup("Group");
    
    RunableCounter runableCounter1 = new RunableCounter(threadGroup, "run1", 5);
    RunableCounter runableCounter2 = new RunableCounter(threadGroup, "run2", 5);
    
    runableCounter1.start();
    runableCounter2.start();
    
    Thread.sleep(2999);
    threadGroup.interrupt(); // 그룹의 모든 스레드 일괄 종료 
    ```
    

## Daemon

## User Thread vs Daemon Thread

- java에서 thread는 user thread 와 daemon thread로 나뉜다.
- User thread는 특별히 daemon thread로 설정하지 않은 thread로서 일반적으로 생성해서 사용되는 thread이다.
- JVM은 모든 user thread가 종료될 때까지 프로그램을 실행한다.
- Daemon thread는 백그라운드에서 동작하는 thread로, user thread가 종료될 때 자동으로 종료된다.
- 주로 main thread나 다른 user thread의 보조 역할을 수행하거나, 특정 작업을 주기적으로 처리하는 thread 등에 사용된다.
- JVM은 모든 user thread가 종료되면 daemon thread를 강제로 종료합니다.
    
    ```java
    ThreadGroup threadGroup = new ThreadGroup("Group");
    
    RunableCounter runableCounter1 = new RunableCounter("run1", 5);
    RunableCounter runableCounter2 = new RunableCounter("run2", 5);
    
    runableCounter2.getThread().setDaemon(true);
    runableCounter1.start();
    runableCounter2.start();
    
    runableCounter1.getThread().interrupt();
    ```
    
    - `runableCounter2` 를 daemon으로 설정 한다.
    - `runableCounter1` 은 유일한 thread가 되었고 곧이어 interrupt 할시 유일한 user thread가 종료되었기에 데몬도 종료된다.
        
        ```java
        RunableCounter runableCounter1 = new RunableCounter("run1", 5);
        RunableCounter runableCounter2 = new RunableCounter("run2", 5);
        RunableCounter runableCounter3 = new RunableCounter("run3", 5);
        
        runableCounter2.getThread().setDaemon(true);
        runableCounter1.start();
        runableCounter2.start();
        runableCounter3.start();
        ```
        
    - 이렇게 수정할 경우 user thread가 아직 존재하기에 daemon thread가 죽지 않는다.

### 정리

- 모든 thread의 생성과 종료를 직접 관리하지 않을 수도 있다.
- Deamon thread의 우선 순위는 낮으나 조정 가능하다.
