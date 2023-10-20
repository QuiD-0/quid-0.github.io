# @Lock과 @Transactional(isolation)의 차이점

spring boot와 jpa를 사용하여 동시성을 제어할때 자주 나오는 키워드가 있습니다.

@Lock을 통한 비관적락, 낙관적 락을 획득 하는것 \
그리고 @Transactional에 파라미터로 넣을 수 있는 isolation Level 입니다.

### @Lock&#x20;

```kotlin
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("select uc from UserCouponEntity uc where uc.id = :userCouponId")
fun findByIdOrNullForUpdate(userCouponId: Long): UserCouponEntity?
```

### IsolationLevel

```kotlin
@Transactional(isolation = Isolation.READ_COMMITTED)
```

### 차이점

1. @Lock (락):
   * @Lock은 데이터베이스에서 동시에 여러 트랜잭션이 동일한 데이터를 수정하거나 액세스하지 못하도록 막는 메커니즘입니다.
   * @Lock은 데이터베이스에서 다중 트랜잭션 환경에서 데이터 무결성과 일관성을 보장하기 위해 사용됩니다.
   * 락은 트랜잭션을 기다리도록 하거나 충돌을 방지하여 동시성을 제어합니다.
2. 트랜잭션 격리 레벨:
   * 트랜잭션 격리 레벨은 여러 트랜잭션 간의 데이터 액세스 및 수정의 격리 정도를 제어하는 것을 나타내는 개념입니다. 데이터베이스에서 네 가지 주요 격리 레벨이 있습니다. 이것은 다음과 같이 나열됩니다(가장 낮은 격리 레벨에서 가장 높은 격리 레벨 순서로):
   * READ UNCOMMITTED
   * READ COMMITTED
   * REPEATABLE READ
   * SERIALIZABLE

트랜잭션 격리 레벨은 트랜잭션 간의 데이터 액세스 및 수정이 어떻게 상호작용하는지에 대한 규칙을 정의하며, 각 레벨은 동시성과 데이터 일관성 사이의 트레이드오프를 나타냅니다.

두 개념을 연결하면, 트랜잭션 격리 레벨이 락 메커니즘을 구현하는 방식에 영향을 미칩니다. 격리 레벨이 높을수록 락을 더 엄격하게 사용하여 동시성을 줄일 수 있습니다. 따라서, 데이터베이스 시스템은 격리 레벨과 락을 함께 사용하여 데이터의 일관성을 유지하면서 동시성을 제어합니다.

### 그래서..?

즉, @Lock은 동시 액세스를 제어하기 위한 실질적인 메커니즘이며, 트랜잭션 격리 레벨은 이러한 락 메커니즘의 동작을 규제하는 설정 레벨을 나타냅니다.
