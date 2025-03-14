# 병렬 프로그래밍

### <mark style="background-color:yellow;">스레드</mark>&#x20;

프로세스 내에서 실행되는 작업 단위\
병렬로 실행할 수 있는 최대 스레드 수는 시스템의 물리적 코어 수와 같다.

* 하나의 프로세스는 **여러 개의 스레드**를 가질 수 있음(멀티스레딩).
* 같은 프로세스 내 스레드는 **코드, 데이터, 힙을 공유**하지만, **각자의 스택을 가짐**.
* **멀티스레딩(Multi-Threading)**: 하나의 프로세스가 여러 작업을 동시에 실행.
* 스레드의 실행 순서는 OS가 결정하고 **실질적으로 무작위**로 결정된다.



**생성**&#x20;

* 메인 스레드가 하나 이상의 작업자 스레드를 생성
* 프로세스와 동시에 자체 실핼 컨텍스트 내에서 실행

**조인**&#x20;

* 단일 스레드 프로세스로 진행하기 전에 모든 작업 스레드가 완료되기를 기다리는 상태
* 종료된 스레드를 조인하면 실행 컨텍스트와 리소스가 해제



### 스레드 동기화

스레드가 특정 순서로 실행되는것, 프로그램의 실행 시간을 늘리기도 하지만 정확성을 보장한다.&#x20;



**Race Condition / 경쟁 상태**

**하나의 데이터**를 **두개이상의 스레드**가 **읽기 - 수정 - 쓰기** 패턴을 통해 수정할 경우 잘못된 결과를 낸다.



**뮤텍스**&#x20;

한 번에 단 하나의 스레드만이 임계 구역 내의 코드를 실행 하도록 보장&#x20;

```java
class SharedResource {
    private int count = 0;
    
    public synchronized void increment() {  // 뮤텍스 역할
        count++;
    }
}
```

**세마포어**&#x20;

여러 개의 스레드가 접근 가능하지만 개수 제한

```java
import java.util.concurrent.Semaphore;

class SharedResource {
    private Semaphore semaphore = new Semaphore(2); // 동시에 2개 스레드 허용

    public void useResource() {
        try {
            semaphore.acquire();  // 자원 획득 (카운트 감소)
            System.out.println(Thread.currentThread().getName());
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            semaphore.release();  // 자원 반환 (카운트 증가)
        }
    }
}
```

**데드락**

여러 스레드나 프로세스가 서로 다른 자원을 서로 기다리며 영원히 멈추는 상태











