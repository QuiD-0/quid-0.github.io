# Index With Jpa

## 인덱스 적용하기 전

### Entity

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Diary {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(length = 20)
    private String username;
    
    @Column(length = 100)
    private String title;
    
    @Column(length = 1000)
    private String content;
    
    private LocalDate date;
    
    private LocalDateTime createdAt;
    
    @Builder
    public Diary(String username, String title, String content, LocalDate date) {
        this.username = username;
        this.title = title;
        this.content = content;
        this.date = date;
        this.createdAt = LocalDateTime.now();
    }

}
```

### DTO

```java
@Builder
public record DailyCount (LocalDate date, Long count) {
}
```

### QueryDsl Repository

```java
public List<DailyCount> dailyCount() {
        return jpaQueryFactory
            .select(Projections.constructor(
            DailyCount.class, diary.date, diary.date.count()))
                .from(diary)
                .groupBy(diary.date)
                .fetch();
    }
```

## 인덱스 적용 후

### Entity 추가 사항

```java
@Table(name = "diary", indexes = 
    @Index(name = "idx_diary_date", columnList = "date"))
```







### 실행 결과

총 10만 건 데이터 조회 결과

#### 적용 전

```
Response code: 200; Time: 52580ms (52 s 580 ms); Content length: 3455 bytes (3.46 kB)
```

#### 적용 후

```
Response code: 200; Time: 57ms (57 ms); Content length: 3455 bytes (3.46 kB)
```



인덱스 적용 전 <mark style="color:green;">52 s</mark> ->  적용 후 <mark style="color:green;">57 ms</mark>







