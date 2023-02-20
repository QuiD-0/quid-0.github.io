# Record



## Record란?

* 불변(immutable) 데이터 객체를 쉽게 생성할 수 있도록 해주는 클래스
* JDK16에서 정식 스펙으로 포함&#x20;



## 간단한 사용법

```java
@Builder
public record DiaryDto(Long id, String username, String title, 
String content,LocalDate date, LocalDateTime createdAt) {
}
```



## 기존의class 를사용할 경우

```java
@Getter
public class DiaryDto(){
    private Long id;
    private String username;
    private String title;
    private String content;
    private LocalDate date;
    private LocalDateTime createdAt;

    @Builder
    public DiaryDto(Long id, String username, String title, String content, 
    LocalDate date, LocalDateTime createdAt) {
        this.id = id;
        this.username = username;
        this.title = title;
        this.content = content;
        this.date = date;
        this.createdAt = createdAt;
    }
}
```
