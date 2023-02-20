# Cacheable

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
    public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
        return RedisCacheManager.builder(redisConnectionFactory)
            .cacheDefaults(defaultCacheConfiguration())
            .withInitialCacheConfigurations(customCacheConfiguration()).build();
    }

    private RedisCacheConfiguration defaultCacheConfiguration() {
        return RedisCacheConfiguration.defaultCacheConfig()
        .entryTtl(Duration.ofSeconds(10))
            .serializeKeysWith(SerializationPair.fromSerializer(
            new StringRedisSerializer()))
            .serializeValuesWith(SerializationPair.fromSerializer(
            new GenericJackson2JsonRedisSerializer()));
    }

    private Map<String, RedisCacheConfiguration> customCacheConfiguration() {
        Map<String, RedisCacheConfiguration> cacheConfigurations = new HashMap<>();
        cacheConfigurations.put("user",
            defaultCacheConfiguration().entryTtl(Duration.ofMinutes(10)));
        return cacheConfigurations;
    }

}
```



## Cacheable

```java
    @Override
    @Transactional(readOnly = true)
    @Cacheable(value = "postList", 
    key = "{#userName, #pageable.pageNumber, #pageable.pageSize}")
    public Page<PostDto> list(Pageable pageable, String userName) {
        userRepository.findByUserNameOrThrow(userName);
        return new RestPage(postRepository.findAll(pageable).map(Post::toDto));
    }

    @Override
    @Transactional(readOnly = true)
    @Cacheable(value = "postByUser", 
    key = "#user.id, #pageable.pageNumber, #pageable.pageSize")
    public Page<PostDto> myFeed(Pageable pageable, String userName) {
        User user = userRepository.findByUserNameOrThrow(userName);
        return new RestPage(postRepository.findByUser(user, pageable)
        .map(Post::toDto));
    }

    @Override
    @Transactional(readOnly = true)
    @Cacheable(value = "postSearch", 
    key = "{#keyword, #pageable.pageNumber, #pageable.pageSize}")
    public Page<PostDto> search(Pageable pageable, String keyword) {
        return new RestPage(postRepository.searchPost(keyword, keyword, pageable)
        .map(Post::toDto));
    }
```

@Cachable 어노테이션 사용

value와 key 가 value::key의 형태로 redis에 저장된다. / "postList::quid,0,20"







