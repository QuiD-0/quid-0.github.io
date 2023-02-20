# Feign With Spring

## build.gradle

```groovy
dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:2021.0.4"
    }
}

dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
}
```

버전은 프로젝트의 spring 버전에 따라 호환되는 버전을 이용

## EnableFeignClients

```java
@EnableFeignClients
@SpringBootApplication
public class PlayGroundApplication {
    public static void main(String[] args) {
        SpringApplication.run(PlayGroundApplication.class, args);
    }
}

```

## Client

```java
@FeignClient(name = "dummyJsonClient", url = "https://dummyjson.com/products")
public interface DummyJsonFeignClient {

    @GetMapping("/{id}")
    ResponseEntity<ProductRes> getDummyJson(@PathVariable(name = "id") String id);
}

```

## Service

```java
@Service
@RequiredArgsConstructor
public class FeignService {

    private final DummyJsonFeignClient dummyJsonFeignClient;
    
    public ResponseEntity<ProductRes> callDummyJson(String id) {
        return dummyJsonFeignClient.getDummyJson(id);
    }
}

```

