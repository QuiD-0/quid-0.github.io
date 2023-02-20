# 동시성 제어

## Build.gradle

```bash
dependencies {
    ...
    //레디스
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    //redisson을 사용 하려면 추가
    implementation 'org.redisson:redisson-spring-boot-starter:3.17.7'
    ...
}
```



## Stock entity

```java
@Entity
@Getter
@ToString
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Stock {

    @Id
    @GeneratedValue(strategy = IDENTITY)
    private Long id;

    private Long productId;

    private Long quantity;


    @Builder
    public Stock(Long productId, Long quantity) {
        this.productId = productId;
        this.quantity = quantity;
    }

    public void decreaseStock(Long quantity) {
        if(this.quantity < quantity) {
            throw new RuntimeException("Not enough stock");
        }
        this.quantity -= quantity;
    }
}

```

## Service

```java
    @Override
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void decreaseStock(Long productId, Long quantity) {
        Stock stock = stockJpaRepository.findByProductId(productId)
            .orElseThrow(() -> new RuntimeException("Product not found"));
        stock.decreaseStock(quantity);
        stockJpaRepository.saveAndFlush(stock);
    }
```

## Lettuce

### repository

```java
@Repository
@RequiredArgsConstructor
public class StockRedisRepository {

    private final RedisTemplate<String, String> redisTemplate;

    public Boolean lock(Long key) {
        return redisTemplate.opsForValue().setIfAbsent(generateKey(key), "lock", Duration.ofMillis(3_000));
    }

    public void unlock(Long key) {
        redisTemplate.delete(generateKey(key));
    }

    private String generateKey(Long key) {
        return "productId::"+key.toString();
    }
}
```

### usage

```java
public void decreaseStockWithRedisLettuce(long productId, long quantity)
            throws InterruptedException {
                while (!stockRedisRepository.lock(productId)) {
                    Thread.sleep(100);
                }
                try {
                    stockService.decreaseStock(productId, quantity);
                } finally {
                    stockRedisRepository.unlock(productId);
                }
        }
```

## Redisson

RedissonClient를 주입받아 사용한다.

### usage

```java
public void decreaseStockWithRedisRadisson(Long productId, long quantity) {
            RLock lock = redisonClient.getLock("productId::"+productId.toString());
            try {
                if(!lock.tryLock(5,1, TimeUnit.SECONDS)) {
                    throw new RuntimeException("Lock is not available");
                }
                stockService.decreaseStock(productId, quantity);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                lock.unlock();
            }
        }
```

