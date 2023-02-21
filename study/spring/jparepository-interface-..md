# JpaRepository는 어떻게 interface만으로 동작할까.

## 궁금증

Spring Data Jpa를 사용하다 보면 Entity를 정의 하고 JpaRepository를 상속받은 interface를 별다른

구현 없이 자주 사용을 하게 된다.

이런 interface만으로 어떻게 데이터를 CRUD할 수 있도록 해주는걸까?



## 디버깅

실제 공부를 위해 작성해둔 JpaRepository이다.

```java
public interface DiaryJpaRepository extends JpaRepository<Diary, Long> {

    Page<Diary> findAllByDate(LocalDate date, Pageable pageable);
}
```

그리고 해당 repository를 사용하는 서비스의 일부 코드이다.

```java
@Override
public Page<Diary> read(Pageable pageable) {
    log.info("read diary {}", pageable);
    return diaryJpaRepository.findAll(pageable);
}
```

Page를 리턴하는 findAll 메서드는 내가 작성하지 않아도 JpaRepository에 이미 정의되어있는&#x20;

메서드이기 때문에 내가 작성하지 않아도 곧잘 동작한다.&#x20;

&#x20;하지만 구현체는 찾아볼수 없는데 어떻게 잘 작동을 하는걸까?

궁금증을 해결하기 위해 디버깅 모드를 사용하여 Repository를 사용할때를 breakpoint를 찍어 보면

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

SimpleJpaRepository의 프록시가 주입되어 있는 것을 볼 수 있다.

해당 빈을 찾아가 보면&#x20;

```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {

   private static final String ID_MUST_NOT_BE_NULL = "The given id must not be null!";

   private final JpaEntityInformation<T, ?> entityInformation;
   private final EntityManager em;
   private final PersistenceProvider provider;

   private @Nullable CrudMethodMetadata metadata;
   private EscapeCharacter escapeCharacter = EscapeCharacter.DEFAULT;

   /**
    * Creates a new {@link SimpleJpaRepository} to manage objects of the given {@link JpaEntityInformation}.
    *
    * @param entityInformation must not be {@literal null}.
    * @param entityManager must not be {@literal null}.
    */
   public SimpleJpaRepository(JpaEntityInformation<T, ?> entityInformation, EntityManager entityManager) {

      Assert.notNull(entityInformation, "JpaEntityInformation must not be null!");
      Assert.notNull(entityManager, "EntityManager must not be null!");

      this.entityInformation = entityInformation;
      this.em = entityManager;
      this.provider = PersistenceProvider.fromEntityManager(entityManager);
   }
                    
   이하 생략
```

@Repository 어노테이션과 함께 JpaRepository에서 사용할 수 있엇던 기본적인 메서드들이&#x20;

모두 구현이 되어있는것을 볼 수 있다.

이것 덕분에 JpaRepository는 인터페이스만 작성 해 두면 스프링에서 프록시고 구현체를 주입하여

사용할 수 있는것이다.



## 결과

스프링에서 프록시 빈을 생성하여 의존성을 주입해주기 때문에 interface만으로 동작하는 JpaRepository를 사용 할 수 있다.
