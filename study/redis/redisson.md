# Redisson

## Redisson이란

Redisson은 Redis의 Java client 라이브러리로, Redis 기능을 Java API로 제공합니다. 분산 락(Distributed Lock)을 사용하는 경우 아래와 같은 코드 예제를 참고 할 수 있습니다.

```java
import org.redisson.api.RLock;
import org.redisson.api.RedissonClient;

RedissonClient redisson = Redisson.create();

RLock lock = redisson.getLock("myLock");

try {
    lock.lock();
    // 실행 로직
    ...
} finally {
    lock.unlock();
}
```

Redisson은 특정 시간 내에 lock을 얻지 못하면 자동으로 unlock 하도록 할 수 있는 `tryLock()` 을 제공 합니다.&#x20;

```java
import java.util.concurrent.TimeUnit;

try {
    if (lock.tryLock(100, TimeUnit.SECONDS)) {
        // 실행 로직
        ...
    }
} finally {
    lock.unlock();
}
```

여러가지 락 옵션을 설정 할 수 있는 메서드들을 제공 하니 Redisson 공식 문서를 참조하면 자세한 사용법을 확인 할 수 있습니다.
