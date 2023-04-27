# 여러개의 db를 연결하는 쉬운 방법

## 이슈

springboot + jpa를 사용하며 개발을 진행할때 여러개의 DB를 사용해야 하는 경우가 있다.

실제 운영하는 DB와  해당  DB를 복제한  Repl DB에쿼리를 날리는 경우가 내가 겪은 경우이다.

해당 이슈를 구글링 해보면 multisource 라는 키워드로 검색이 되는데 이는 설정이 복잡하고 불편하다.

## 해결방법

jdbc 를 사용하여 처리하면 된다.

기존의 datasource 설정을 추가하고

```java
@Configuration
public class DefaultJdbcConfig {

    @Value("${spring.datasource.username}")
    private String username;

    @Value("${spring.datasource.password}")
    private String password;

    @Value("${spring.datasource.url}")
    private String jdbcUrl;

    @Value("${spring.datasource.driver-class-name}")
    private String driverClassName;

    @Bean
    JdbcTemplate jdbcTemplate() {
        return new JdbcTemplate(dataSource());
    }

    private DataSource dataSource(){
        return new HikariDataSource(){{
            setUsername(username);
            setPassword(password);
            setJdbcUrl(jdbcUrl);
            setDriverClassName(driverClassName);
        }};

    }

}
```

추가로 사용할 db의 커넥션 정보를 추가한다.

```java
@Configuration
public class ReplJdbcConfig {

    @Bean(name = "replJdbcTemplate")
    JdbcTemplate replJdbcTemplate() {
        return new JdbcTemplate(replDataSource());
    }

    private DataSource replDataSource(){
        return new HikariDataSource(){{
            setUsername("userName");
            setPassword("password");
            setJdbcUrl("jdbc:sqlserver://host;databaseName=repldb");
            setDriverClassName("드라이버명");
        }};

    }

}
```

해당 설정을 사용하게 되면 일반적인 경우에는 기본 datasource 를 사용하게 되고 필요에  따라서 replJdbcTemplate 을 사용하면 된다.

```java
@Service
public class MultiDBService {

    private final JdbcTemplate jdbcTemplate;

    public MultiDBService(@Qualifier("procedureExecutor") JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public List<Map<String, Object>> getMultiDBData() {
        return jdbcTemplate.queryForList("select * from table");
    }
}
```

jdbcTemplate를 사용하는 로직들이 모두 repl db를 통한 쿼리라면 defaultJdbcConfig 를 사용하지 않고 replJdbcConfig 만 사용하면 된다.

하지만 나는 기본 db와 repl db를 모두 사용해야 했기 때문에 defaultJdbcConfig 를 사용하고 replJdbcConfig 를 사용하고 싶은 로직에서 @Qualifier 를 통해 replJdbcTemplate 를 사용하도록 하였다.
