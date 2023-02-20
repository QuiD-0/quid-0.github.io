# Interceptor

## Request Interceptor

```java
@Slf4j
@RequiredArgsConstructor(staticName = "of")
public class FeignInterceptor implements RequestInterceptor {

    @Override
    public void apply(RequestTemplate template) {
        if (template.body() == null) {
            log.info("Request body is null");
            log.info("Request method: {}", template.method());
            log.info("Request queries: {}", template.queries());
            return;
        }

        String body = StringUtils.toEncodedString(template.body(), UTF_8);
        if (Objects.equals(template.method(), HttpMethod.GET.name())) {
            log.info("[GET] [FeignInterceptor] query params: {}", template.queries());
            log.info("[GET] [FeignInterceptor] body: {}", body);
            return;
        }
        if (Objects.equals(template.method(), HttpMethod.POST.name())) {
            log.info("[POST] [FeignInterceptor] body: {}", body);
            //필요한 로직 추가
        }
    }
}
```

## Config

```java
@Configuration
public class FeignConfig {

    @Bean
    public FeignInterceptor feignInterceptor() {
        return FeignInterceptor.of();
    }
}
```

## Result

### Request

```json
GET http://localhost:8080/feign/dummy/10

HTTP/1.1 200 
Content-Type: application/json
Transfer-Encoding: chunked
Date: Thu, 05 Jan 2023 02:05:43 GMT
Keep-Alive: timeout=60
Connection: keep-alive

{
  "id": 10,
  "title": "HP Pavilion 15-DK1056WM",
  "description": "HP Pavilion 15-DK1056WM Gaming Laptop 10th Gen Core i5, 8GB, 256GB SSD, GTX 1650 4GB, Windows 10",
  "price": 1099,
  "discountPercentage": 6,
  "rating": 4,
  "stock": 89,
  "brand": "HP Pavilion",
  "category": "laptops",
  "thumbnail": "https://i.dummyjson.com/data/products/10/thumbnail.jpeg",
  "images": [
    "https://i.dummyjson.com/data/products/10/1.jpg",
    "https://i.dummyjson.com/data/products/10/2.jpg",
    "https://i.dummyjson.com/data/products/10/3.jpg",
    "https://i.dummyjson.com/data/products/10/thumbnail.jpeg"
  ]
}
Response file saved.
> 2023-01-05T110543.200.json

Response code: 200; Time: 1876ms (1 s 876 ms); Content length: 543 bytes (543 B)

```

### Console

```bash
2023-01-05 13:38:35.848  INFO 33708 --- [nio-8080-exec-2] c.q.f.intercepter.FeignInterceptor       : Request body is null
2023-01-05 13:38:35.849  INFO 33708 --- [nio-8080-exec-2] c.q.f.intercepter.FeignInterceptor       : Request method: GET
2023-01-05 13:38:35.849  INFO 33708 --- [nio-8080-exec-2] c.q.f.intercepter.FeignInterceptor       : Request queries: {}
```

현재의 코드에는 아무 파라미터가 없는것을 확인 할 수있다.
