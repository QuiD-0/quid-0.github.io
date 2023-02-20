# Page\<Obj> Error

## Pageable 캐싱

페이징한 결과를 캐싱하여야하는 경우&#x20;

@Cacheable을 사용하여 Page\<Dto>를 캐싱하려한다.

동일하게 어노테이션을 사용하여 캐싱을 진행 해 보면

```
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: 
Cannot construct instance of `org.springframework.data.domain.PageImpl` 
(no Creators, like default constructor, exist): 
cannot deserialize from Object value (no delegate- or property-based Creator)
```

위 에러가 발생하게 된다.



## 해결방법

Page 클래스를 Warpping 해서 사용한다

```java
@JsonIgnoreProperties(ignoreUnknown = true, value = {"pageable"})
public class RestPage<T> extends PageImpl<T> {

    @JsonCreator(mode = JsonCreator.Mode.PROPERTIES)
    public RestPage(@JsonProperty("content") List<T> content,
        @JsonProperty("number") int page,
        @JsonProperty("size") int size,
        @JsonProperty("totalElements") long total) {
        super(content, PageRequest.of(page, size), total);
    }

    public RestPage(Page<T> page) {
        super(page.getContent(), page.getPageable(), page.getTotalElements());
    }
}
```
