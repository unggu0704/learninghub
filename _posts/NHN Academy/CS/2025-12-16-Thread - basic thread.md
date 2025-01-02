---
author: 김규형
date: 2023-09-12 14:10:00 +0800
categories: [NHN Academy, CS]
tags: [CS, NHN Acedemy, Thread]
render_with_liquid: true
---

# Thread - basic thread(23.10.17)

- [x]  Process vs Thread
- [x]  Thread 클래스 만들기
- [x]  Runnable 인터페이스 활용
- [x]  Pipe 구현

### Process vs Thread

- **Process**
    - 실행 중인 프로그램
    - 스케줄링의 대상이 되는 작업과 같은 의미
    - 프로그램이 실행하는데 필요한 모든 자원들을 개별적으로 가지고 있음
    - 개별 메모리로 통신에 어려움
- **Thread**
    - 각 thread별로 Stack과 Register 를 가지고 있음
    - 개별적으로 분리 가능한 최소한의 자원만 가지고 있음
    - 공유 메모리 사용으로 통신에 용이

### Single Thread Vs. Multi Thread

- **Single Tread**
    - Main 쓰레드에서 작업 진행
    - 하나의 Process에 오직 하나의 Thread
    - 동시성에 관리
- **Multi Thread**
    - 프로그램 내에서 두 개 이상의 동작을 동시에 실행
    - 하나의 프로세스를 다수의 실행 단위로 구분하여 자원 공유
    - 수행 능력 향상

## Thread 클래스 확장

> 목표
1. 단일 스레드 환경에서 여러 작업 실행해 문제점을 확인
2. Thread 클래스를 확장 시켜 스레드로 동작하는 클래스 생성
3. 확장된 Thread 클래스를 통해 여러 작업을 동시 실행
> 

### 실습

---

- **연습 1-1. Counter클래스인 인스턴스를 동시에 2개 실행해 확인하기**
    
    ```jsx
    public class myThread {
        public static void main(String[] args) {
            Counter counter1 = new Counter("Counter1");
            Counter counter2 = new Counter("Counter2");
    
            counter1.run();
            counter2.run();
            
        }
    }
    ```
    
    ```jsx
    public class Counter  {
      
            String name;
            int count = 0;
    
            public Counter(String name) {
                this.name = name;
                count++;
            }
    
            public void run() {
                while (!Thread.interrupted()) {
    
                    delay(1);
                    System.out.println(this.name + " : " + count++);
                  }
            }
    
            public void delay(int time) {
                try {
                    Thread.sleep(time);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        
    }
    ```
    
    ```jsx
    Counter1 : 1346
    Counter1 : 1347
    Counter1 : 1348
    Counter1 : 1349
    ...
    ```
    
    - 왜 Counter1 만 실행될까?
        - 하나의 프로그램은 하나의 main()만 실행된다고 한다. Counter1.run()의 메소드가 점유하고 있기에 이러한 현상이 나타난다고 생각…
- **실습 1-2. CounterThread 클래스를 만들어 스레드 동작 확인하기**
    
    ```jsx
    public class myThread {
        public static void main(String[] args) {
            Counter counter1 = new Counter("Counter1");
            Counter counter2 = new Counter("Counter2");
    
            counter1.start(); // run() -> start()로 변경
            counter2.start();
        }
    }
    ```
    
    ```jsx
    public class Counter extends Thread {
      
            String name;
            int count = 0;
    
            public Counter(String name) {
                this.name = name;
                count++;
            }
    
            public void run() {
                while (!Thread.interrupted()) {
    
                    delay(1);
                    System.out.println(this.name + " : " + count++);
                  }
            }
    
            public void delay(int time) {
                try {
                    Thread.sleep(time);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        
    }
    ```
    
    ```jsx
    Counter2 : 1616
    Counter1 : 1616
    Counter2 : 1617
    Counter1 : 1617
    ...
    ```
    
    - 두 쓰레드가 병행 실행됨을 확인
- **실습 1-3. Counter 클래스의 인스턴스와 CounterThread 클래스의 인스턴스를 동시에 실행해 확인하기**
    
    ```jsx
    public class myThread {
        public static void main(String[] args) {
            CounterThread counterThreead = new CounterThread("CounterThread");
            Counter counter = new Counter("no Thread");
            counterThreead.start();
            counter.run();
        }
    }
    ```
    
    ```jsx
    public class Counter {
      
            String name;
            int count = 0;
    
            public Counter(String name) {
                this.name = name;
                count++;
            }
    
            public void run() {
                while (!Thread.interrupted()) {
    
                    delay(1);
                    System.out.println(this.name + " : " + count++);
                  }
            }
    
            public void delay(int time) {
                try {
                    Thread.sleep(time);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        
    }
    
    class CounterThread extends Thread {
      
            String name;
            int count = 0;
    
            public CounterThread(String name) {
                this.name = name;
                count++;
            }
    
            public void run() {
                while (!Thread.interrupted()) {
    
                    delay(1);
                    System.out.println(this.name + " : " + count++);
                  }
            }
    
            public void delay(int time) {
                try {
                    Thread.sleep(time);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        
    }
    ```
    
    ```jsx
    no Thread : 1072
    CounterThread : 1073
    no Thread : 1073
    no Thread : 1074
    CounterThread : 1074
    no Thread : 1075
    CounterThread : 1075
    no Thread : 1076
    CounterThread : 1076
    no Thread : 1077
    CounterThread : 1077
    no Thread : 1078
    CounterThread : 1078
    ...
    ```
    
    - `CounterThread`와 `no Thread`가 병행 실행됨.
    

### 정리. Thread 클래스의 활용

- Thread 클래스를 확장하여 스레드를 생성할 수 있다.
- Thread 클래스 인스턴스는 실행시 개별 스레드를 생성하여 동작하여, 하나 이상의 스레드 생성이 가능하다.
- 프로그램이 시작되는 main 함수도 하나의 스레드로 동작한다

---

## Runnable 인터페이스 구현

- 클래스 상속 외에 Runnable 인터페이스로도 구현 가능
- 자바는 단일상속이기에  Thread 사용시 다른 클래스를 상속 받지 못함

- **실습 2-1 RunnableCounter 인터페이스 구현**
    
    ```java
    public class RunableCounter implements Runnable {
        int counter = 0;
        int maxCount = 0;
        String str = "";
    
        RunableCounter(String str, int counter) {
            this.str = str;
            this.maxCount = counter;
        }
    
        @Override
        public void run() {
            while (counter < maxCount) {
                System.out.println(str + " : " + counter++);
            }
        }
    }
    ```
    
    ```java
    public class myThread {
        public static void main(String[] args) {
            System.out.println("Thread 생성! ");
            Thread thread = new Thread (new RunableCounter("runnableCounter" , 10));
            thread.start();
        }
    }
    ```
    
- **실습 2-1 CounterRunnable 클래스의 인스턴스를 동시에 2개 실행해 확인하기**
    
    ```jsx
    public class myThread {
        public static void main(String[] args) {
            CounterRunnable counterRunnabl1 = new CounterRunnable("CounterRunnable1");
            CounterRunnable counterRunnabl2 = new CounterRunnable("CounterRunnable2");
    
            Thread thread1 = new Thread(counterRunnabl1);
            Thread thread2 = new Thread(counterRunnabl2);
    
            thread1.start();
            thread2.start();
        }
    }
    ```
    
    ```jsx
    public class CounterRunnable implements Runnable  {
        String name;
        int count;
    
        public CounterRunnable(String name) {
            this.name = name;
        }
    
        public void run() {
            while (!Thread.interrupted()) {
                delay(1000);
                System.out.println(this.name + " : " + count++);
            }
        }
    
        public void delay(int time) {
            try {
                Thread.sleep(time);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    ```
    
    ```jsx
    CounterRunnable1 : 0
    CounterRunnable2 : 0
    CounterRunnable1 : 1
    CounterRunnable2 : 1
    CounterRunnable1 : 2
    ...
    ```
    
- **연습 2-2. CounterRunnable 클래스를 CounterThread 클래스와 동일하게 구성 후 결과 확인하기**
    
    ```jsx
    public class myThread {
        public static void main(String[] args) {
            CounterRunnable counterRunnabl1 = new CounterRunnable("CounterRunnable1");
            CounterRunnable counterRunnabl2 = new CounterRunnable("CounterRunnable2");
    
            counterRunnabl1.start();
            counterRunnabl2.start();
        }
    }
    ```
    
    ```jsx
    public class CounterRunnable implements Runnable  {
        String name;
        int count;
        Thread thread;
    
        public CounterRunnable(String name) {
            this.name = name;
            thread = new Thread(this);
        }
    
        public void start() {
            thread.start();
        }
    
        @Override
        public void run() {
            while (!Thread.interrupted()) {
                System.out.println(this.name + " : " + ++count);
                delay(1000);
            }
        }
    
        public void delay(int time) {
            try {
                Thread.sleep(time);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    ```
    
    ```jsx
    CounterRunnable1 : 1
    CounterRunnable2 : 1
    CounterRunnable1 : 2
    CounterRunnable2 : 2
    ...
    ```
    
    - Thread를 Runnable 객체에 Thread 인스턴스 변수를 사용해서 관리
    - Start는 객체 메소드로 선언 후 해당 Thread를 start() 하게 관리

### 문제점

- main() 이 먼저 종료 한다.
    - `thread.join()` 을 활용해서 `main` 을 대기시킨다.
        
        ```java
        try {
                thread.start();
                thread.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
        ```
        
- Thread의 실행 순서는 어떻게 결정되는걸까?
    
    ```java
    thread1.start();
    thread2.start();
    thread3.start();
    thread4.start();
    
    mycounter4 : 0
    mycounter3 : 0
    mycounter2 : 0
    mycounter1 : 0
    mycounter4 : 1
    mycounter1 : 1
    mycounter2 : 1
    mycounter3 : 1
    ```
    
    아마 랜덤 인것 같다….
    
    - `join` 사용
    
    ```java
    try {
            thread1.start();
            thread1.join();
            thread2.start();
            thread2.join();
            thread3.start();
            thread3.join();
            thread4.start();
            thread4.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
    
    mycounter1 : 0
    mycounter1 : 1
    mycounter1 : 2
    mycounter1 : 3
    mycounter1 : 4
    ```
    
    → 실행 순서 보장 완료!

### Class 내에서 Thread를 field로 포함

- Runnable interface를 구현하는 class가 필요함 이걸 class내에 Thread Instance로 관리
- 자신이 Thread를 관리하므로 더 많은 처리가 가능
- **예시 소스 코드 (`SelfRunnableCounter.java`)**
  ```java
  public class SelfRunnableCounter implements Runnable {
    int count;
    int maxCount;
    Thread thread;

    SelfRunnableCounter(String name, int maxCount) {
        this.maxCount = maxCount;
        count = 0;
        thread = new Thread(this, name);
    }

    public void start() {
        thread.start();
    }

    @Override
    public void run() {
        while (count < maxCount) {
            try {
                System.out.println(count++);
                thread.sleep(1000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}



### Thread의 Interrupt

- **Interrupt에 대해서 더 알아보기…**
    - Thread는 여러 상황에서 `Interrupt` 를 통해 통제한다.
    - 특정 스레드에게 작업을 멈춰달라고 요청하는 기능
    - `Interrupt`를 다룰 수 있는 메소드
        - `interrupt` : 해당 스레드에게 인터럽트를 검.
        - `isInturrupted` : 해당 스테드가 인터럽트가 걸려 있는지를 확인
        - `static interrupted` : 현재 해당 스레드의 인터럽트를 해제하고 해제하기 이전의 값이 무엇인지를 return 또한 인터럽트를 해제할 수 있음.
    - `while (!Thread.interrupted())` 같은 경우 이전에 Thread가 인터럽트 상태였다면 True를 반환 그리고 스레드의 인터럽트를 해제하고 반복문은 종료, 하지만 인터럽트 상태가 아니였다면 False를 반환 반복문은 멈추지 않는다.
    - 만약에 어디서 누군가가 `interrupt`를 건다면…?
        
        ```jsx
        public class CounterRunnable implements Runnable  {
            String name;
            int count;
            Thread thread;
        
            public CounterRunnable(String name) {
                this.name = name;
                thread = new Thread(this);
            }
        
            public void start() {
                thread.start();
            }
        
            @Override
            public void run() {
                while (!Thread.interrupted()) {
                    System.out.println(this.name + " : " + ++count);
                    delay(1000);
                }
                System.out.println("Interrupted!");
            }
        
            public void delay(int time) {
                try {
                    Thread.sleep(time);
                    thread.interrupt(); // 쓰레드에게 인터럽을 걸어버림! 작업을 멈춰라.
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
        ```
        

### 정리. Runnable 인터페이스의 활용

- CounterRunnable 인터페이스를 이용해 스레드 활용이 가능하다.
- 실제 활용은 인터페이스 구현시 Thread 클래스와 동일하게 run() 함수 재정의로 구현한다.
- Thread 클래스 확장과는 달리 별도의 Thread 인스턴스 생성이 필요하지만, 클래스 구현 단계에서 해결 가능하다.

---

## 스레드 상태

| 상태 | 설명 |
| --- | --- |
| NEW | 생성되었지만 아직 start() 메소드가 호출되지 않은 상태 |
| RUNNABLE | 실행 대기 중인 상태 |
| BLOCKED | 락(lock)을 획득하지 못해 실행을 멈춘 상태 |
| WAITING | 다른 스레드에 의해 특정 조건이 충족될 때까지 대기하는 상태 |
| TIMED_WAITING | 일정 시간 동안 대기하는 상태 |
| TERMINATED | 실행이 종료된 상태 |
- 스레드간의 상태 확인은 Pipe로 연결된다.
- Pipe를 통해 데이터는 임의의 시간 간격으로 데이터를 집어넣는다.
    - 만약 Pipe가 채워져 있다면 대기한다.
- Pipe를 통해 데이터를 받는다.
    - 만약 Pipe가 비워져 있다면 대기한다.

- 두 스레드간 Pipe 로 데이터를 전송하는 클래스
    - *Pipe.java*
    
    ```java
    package Thread;
    
    import java.util.LinkedList;
    import java.util.Queue;
    
    public class Pipe {
        private int data;
        private boolean empty = true;
    //    Queue<Integer> queue = new LinkedList<>();
    
        public synchronized int receive() {
    
            if (empty) { // 비어있으니 wait()
                try {
                  wait();
                } catch (InterruptedException e) { }
            } 
    
            empty = true;
            notifyAll();
    
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            System.out.println("파이프의 " + this.data + " 내보냄 ");
          	return data;
        }
    
        public synchronized void send(int data) {
    
            if (!empty) { // 뭐가 있으니깐 wait()
                try {
                  wait();
                } catch (InterruptedException e) { }
            }
            
            empty = false;
            notifyAll();
    
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            this.data = data;
            System.out.println("파이프에 " + this.data + " 저장 ");
        }
    }
    ```
    
    - *Reciver.java*
    
    ```java
    package Thread;
    
    import java.lang.Thread.State;
    import java.time.LocalDateTime;
    import java.util.concurrent.ThreadLocalRandom;
    import javax.sound.midi.Track;
    
    public class Receiver implements Runnable {
        final Pipe pipe;
    		int endData = 10;
        Thread thread;
      
        public Receiver(Pipe pipe, int endData) {
            this.pipe = pipe;
            thread = new Thread(this);
        }
    
        public void start() {
            thread.start();
        }
    
        public State getState() {
          return thread.getState();
        }
    
        public void run() {
          // Pipe에서 데이터를 기다리며, 수신된 데이터가 endData이면 종료한다.
         	// 수신된 데이터가 endData가 아닌 경우, 임의의 시간을 기다린다. 
    
          while (!Thread.interrupted()) {
              int data = pipe.receive();
              if (endData == data) {
                  /* 
                  try {
                    System.out.print("모든 데이터 수신 완료");
                    thread.join();
                    
                  } catch (InterruptedException e) {
                    e.printStackTrace();
                  }
                  */
                  thread.interrupt();
                  System.out.print("reciver는 모든 데이터를 수신하였습니다. reciver 스레드 종료.");
              } else {
                    try {
                      System.out.println("데이터 " + data + " 도착");
                      Thread.sleep(1000);
                    } catch (InterruptedException e) {
                      Thread.currentThread().interrupt();
                    }
                }
            }
      
        }
    }
    ```
    
    - *Sender.java*
    
    ```java
    package Thread;
    
    import java.lang.Thread.State;
    import java.util.concurrent.ThreadLocalRandom;
    
    public class Sender implements Runnable {
        final Pipe pipe;
        int[] data = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
        int dataIndex = 0;
        Thread thread;
    
        public Sender(Pipe pipe) {
            this.pipe = pipe;
            thread = new Thread(this);
        }
    
        public void start() {
            thread.start();
        }
    
        public State getState() {
          return thread.getState();
        }
    
        public void run() {
          // pipe를 통해서 데이터를 전송한다.
    
          while (!Thread.interrupted()) {
            
              pipe.send(data[++dataIndex]);
              if (dataIndex == 10) {
                thread.interrupt();
                System.out.println("sender는 모든 데이터를 전송하였습니다. sender 쓰레드 종료");
              }
            try {
                 Thread.sleep(1000);
            } catch (InterruptedException e) {
                 Thread.currentThread().interrupt();
            } 
          }
        
          // 전송에 성공하면 일정시간 기다린다.
        }
    }
    ```
