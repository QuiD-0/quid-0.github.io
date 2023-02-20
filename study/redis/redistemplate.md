# RedisTemplate

## Configuration

```java
@Configuration
@EnableCaching
@RequiredArgsConstructor
public class RedisConfig {

    private final RedisProperties redisProperties;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        RedisURI redisURI = RedisURI.create(redisProperties.getUrl());
        RedisConfiguration configuration = 
        LettuceConnectionFactory.createRedisConfiguration(redisURI);
        LettuceConnectionFactory factory = 
        new LettuceConnectionFactory(configuration);
        factory.afterPropertiesSet();
        return factory;
    }
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(
        RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        return redisTemplate;
    }

}
```



## RedisTemplate

```java
@Slf4j
@Repository
@RequiredArgsConstructor
public class UserCacheRepository {

    private final static Duration USER_CACHE_TTL = Duration.ofDays(3);
    private final RedisTemplate<String, Object> redisTemplate;

    private static String getKey(String username) {
        return "user:" + username;
    }

    public void setUser(UserDto user) {
        String key = getKey(user.getUsername());
        redisTemplate.opsForValue().set(key, user, USER_CACHE_TTL);
    }

    public Optional<UserDto> getUser(String username) {
        String key = getKey(username);
        log.info("getUser: {}", key);
        return Optional.ofNullable(castInstance(redisTemplate.opsForValue().get(key),
            UserDto.class));
    }

}
```

