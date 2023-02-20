# Redis With Springboot

## docker-compose

```yaml
version: '3.8'

services:
  redis:
    image: redis:alpine
    command: redis-server --port 6379
    container_name: redis_boot
    hostname: redis_boot
    labels:
      - "name=redis"
      - "mode=standalone"
    ports:
      - 6379:6379
```

## build.gradle

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
}
```

## application.yaml

```yaml
spring:
  redis:
    host: localhost
    port: 6379
    url: redis://localhost:6379
```



## logging

```yaml
logging:
  level:
    org:
      springframework:
        cache: TRACE
```

### 로그 내역

```
2022-11-16 14:26:52.875 TRACE 1736 --- [io-8080-exec-10] o.s.cache.interceptor.CacheInterceptor   : Computed cache key 'username' for operation Builder[public com.quid.sns.user.User com.quid.sns.user.repository.UserRepositoryImpl.findByUserNameOrThrow(java.lang.String)] caches=[user] | key='#userName' | keyGenerator='' | cacheManager='' | cacheResolver='' | condition='' | unless='' | sync='false'
2022-11-16 14:26:52.880 TRACE 1736 --- [io-8080-exec-10] o.s.cache.interceptor.CacheInterceptor   : Cache entry for key 'username' found in cache 'user'
2022-11-16 14:26:52.892 TRACE 1736 --- [io-8080-exec-10] o.s.cache.interceptor.CacheInterceptor   : Computed cache key '[username, 0, 20]' for operation Builder[public org.springframework.data.domain.Page com.quid.sns.post.service.PostServiceImpl.list(org.springframework.data.domain.Pageable,java.lang.String)] caches=[postList] | key='{#userName, #pageable.pageNumber, #pageable.pageSize}' | keyGenerator='' | cacheManager='' | cacheResolver='' | condition='' | unless='' | sync='false'
2022-11-16 14:26:52.898 TRACE 1736 --- [io-8080-exec-10] o.s.cache.interceptor.CacheInterceptor   : Cache entry for key '[username, 0, 20]' found in cache 'postList'
```



