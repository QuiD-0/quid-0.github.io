# RedisTemplate 직렬화시 @Class없애기

## Genericjackson2jsonredisserializer

genericjackson2jsonredisserializer 를 사용하면 간단하게 class를 직렬화/역직렬화 할수있다.

하지만 직렬화되어 레디스에 저장된 데이터 값을 보면 @class : {classPath} 타입이 남아있다.

이는 싱글 프로젝트에서는 문제가 되지 않겠지만 분산 서버 환경에서는 모든 서버에서 패키지명을

일치 시켜줘야 하는 치명적인 문제가 발생하게 된다.

```log
127.0.0.1:6379> get user::quid
"{
\"@class\":\"com.quid.sns.user.User\",
\"registeredAt\":[2022,10,25,17,1,32,608662000],
\"updatedAt\":[2022,10,25,17,1,32,608662000],
\"removedAt\":null,
\"id\":3,
\"userName\":\"quid\",
\"password\":\"$2a$10$UQuKxQrwxbJ51ot1k0EhBeG1aUos2IMUAZsGWsipZ1dyTi8bqzzqC\",
\"role\":\"USER\"
}"
```



## 해결 방법

```java
 @Bean
 public RedisTemplate<String, Object> redisTemplate(
        RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer(Object.class));
        return redisTemplate;
 }
```

ValueSerializer를 <mark style="color:green;">Jackson2JsonRedisSerializer(Object.class)</mark>으로 지정 해 준다.

혹은 원하는 클래스 별로 RedisTemplate를 여러개생성해도 되지만&#x20;

나는  범용으로  사용할 수 있는 RedisTemplate를 위해 Object 클래스로 지정 해 주었다.



```java
@Component
@RequiredArgsConstructor
public class RedisBase {

    private final RedisTemplate redisTemplate;

    private final ObjectMapper objectMapper;

    @SneakyThrows
    public <T> Optional<T> getData(String key, Class<T> classType) {
        Object result = redisTemplate.opsForValue().get(key);
        return result == null ? Optional.empty()
            : Optional.of(objectMapper.convertValue(result, classType));
    }

    public <T> void save(String key, T value, Duration expireTime) {
        redisTemplate.opsForValue().set(key, value, expireTime);
    }
    
    public void delete(String key) {
        redisTemplate.delete(key);
    }

}
```

이후  RedisTemplate를 사용하기 위한RedisBase를 만들어 준다.

제네릭 타입을 사용하여 원하는 타입의 데이터로 변환하여 리턴하는 메소드를 작성한다.



```java
@Repository
@RequiredArgsConstructor
public class UserRedisRepository {

    private final RedisBase redisBase;

    private static final Duration TTL = Duration.ofMinutes(5);

    public void save(String key, User value) {
        redisBase.save(getKey(key), value, TTL);
    }

    public Optional<User> getData(String key) {
        return redisBase.getData(getKey(key), User.class);
    }
    
     public void delete(String key) {
        redisBase.delete(getKey(key));
    }

    public String getKey(String userName) {
        return "user::" + userName;
    }
}

```

원하는 Repository에 RedisBase를 주입하여 사용한다.



## 결과

@class 없이 저장된 모습을 확인할 수 있다.

```log
127.0.0.1:6379> get user:quid
"{\"registeredAt\":[2022,10,25,17,1,32,608662000],
\"updatedAt\":[2022,10,25,17,1,32,608662000],
\"removedAt\":null,
\"id\":3,
\"userName\":\"quid\",
\"password\":\"$2a$10$UQuKxQrwxbJ51ot1k0EhBeG1aUos2IMUAZsGWsipZ1dyTi8bqzzqC\",
\"role\":\"USER\"
}"
```



