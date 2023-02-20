# Redis Serializer Error

## 직렬화 오류

Spring-data-redis @Cacheable java.lang.ClassCastException: java.util.LinkedHashMap cannot be cast to MyObject



## 해결방법

ValueSerializer : GenericJackson2JsonRedisSerializer() 사용&#x20;

```java
    @Bean
    public RedisTemplate<String, Object> redisTemplate(LettuceConnectionFactory lettuceConnectionFactory, ObjectMapper objectMapper) {
        final RedisTemplate<String, Object> redisTemplate = new RedisTemplate<String, Object>();
        redisTemplate.setConnectionFactory(lettuceConnectionFactory);
        // value serializer
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        // hash value serializer
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());

        logger.info("wiring up Redistemplate...");
        return redisTemplate;
    }
```
