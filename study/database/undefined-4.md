# 동시성 이슈

## 동시성 이슈를 제거하기 위한 방법

### 1. 동시성 이슈란?

동시성 이슈는 여러 스레드 또는 서버에서 동시에 같은 데이터를 변경하려고 할 때 발생하는 문제입니다.\
이러한 문제는 데이터의 일관성을 해칠 수 있습니다.

### 2. 동시성 이슈를 해결하는 방법

서버가 하나일 경우 동시성 이슈를 해결하는 방법은 간단합니다. syncronized 키워드를 사용하면 됩니다.

````
```java
public class Counter {
    private int count = 0;

    public syncronized void increment() {
        count++;
    }

    public int getCount() {
        return count;
    }
}
```
````

하지만 서버가 여러개일 경우 syncronized 키워드를 사용하면 안됩니다.

데이터 베이스를 활용하여 데이터의 정합성을 유지하는 방법이 있습니다.

### 3. 데이터 베이스를 활용한 동시성 이슈 해결

#### 1. optimistic lock

lock 을 걸지않고 문제가 발생할 때 처리합니다.\
대표적으로 version column 을 만들어서 해결하는 방법이 있습니다.

#### 2. pessimistic lock (exclusive lock)

다른 트랜잭션이 특정 row 의 lock 을 얻는것을 방지합니다.\
A 트랜잭션이 끝날때까지 기다렸다가 B 트랜잭션이 lock 을 획득합니다.\
특정 row 를 update 하거나 delete 할 수 있습니다.\
일반 select 는 별다른 lock 이 없기때문에 조회는 가능합니다.

#### 3. named lock

이름과 함께 lock 을획득합니다.\
해당 lock 은 다른세션에서 획득 및 해제가 불가능합니다.



### 4. redis lock

redis 는 메모리 기반의 key-value 저장소입니다. redis 에서는 lock 을 획득하고 해제하는 기능을 제공합니다.

#### 1. lettuce

spring data redis를 이용하면 lettuce가 기본이기 때문에 별도의 라이브러리를 추가할 필요가 없습니다. spin lock 을 이용하여 lock 을 획득하고 해제하는 방법입니다.

#### 2. redisson

redisson 은 redis 를 이용하여 lock 을 획득하고 해제하는 기능을 제공합니다. pub-sub 을 이용하여 lock 을 획득하고 해제하는 방법입니다.

