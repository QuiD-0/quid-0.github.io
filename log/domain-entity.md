# Domain과 Entity 분리하기

## 더 나은 코딩생활을 위해.

Java + spring으로 백엔드에 처음 입문했을때의 이야기이다.

JPA라는 ORM 기술을 접하고 처음 느낀 감정을 이야기 하자면 "이정도로 간편하게 모든 걸 할수있다고?" 였다.

실제로 꽤나 오랫동안 JPA를 통해 모든것을 하려했엇고 아직도 JPA를 아주 열심히 사용중이다.

하지만 예전과 지금의 코드 스타일은 아주많이 바뀌게 되었다.

무엇이 정답이고 최고라는 생각은 하지않는다. 기술을 사용하는 사람과 그룹이 모두 편하고 잘 사용할 수 있는것이 최고의 정도라고 생각한다.

나는 이 글의 제목과 같이 도메인과 엔티티를 분리하는것이 깨끗하게 코드를 작성하고 유지보수 해 나가는 방법이라고 생각 하여 이렇게 글을 쓰게 되었다.



예전에 나는 도메인 = 엔티티 라고 생각 하고 모든 엔티티를 서비스단에서 이리저리 바꾸고 더티체킹을 통해 변경사항을 저장하곤 했다. 이렇게 작성된 간단해 보이는 코드를 지향했다.

하지만 이런 풀JPA코드는 문제가 있다.

모든 서비스 코드, 레파지토리, 엔티티가 강하게 결합되어있으며 이것은 곧 레이어간의 경계가 사라지는 듯한 효과를 가져온다.

또한 다른 엔티티와 매핑이되어있을 경우 문제는 더욱 커지게 된다. JPA에서 지원하는 연관관계 매핑을 사용하여 다른 엔티티와 함께 수정하고 저장하고 연관관계의 주인만을 통해 객체를 수정하는등의 일련의 과정들을 누군가는 실수 할 수 있다.

당연하게도 잘 이해하고 사용하는 사람도 있을태지만 모든 개발자가 잘 이해하고 사용하나에 대한 질문은 단연코 "아니요" 리고 답 할 수 있다.

당연히 N+1문제와 같은  연관관계에서 발생하는 이슈도 포함이다.



도메인과 엔티티를 분리함으로써 얻을 수 있는 이점중 가장 큰 것은 바로 의존성의 분리라고 생각한다.

"헥사고날 아키텍처"를 읽어보았다면 각각의 레이어별로 역할과 그 레이어들간의 의존성을 분리하여 유지보수하기 좋은 코드를 작성하고 인터페이스를 통해 각각의 레이어들이 실제로 어떻게 동작하는지 전혀 몰라도 코드가 작동하도록 한다.

나또한 엔티티가 도메인과 분리됌으로써 JPA를 레포지토리 레이어에서만 사용되도록 사용하는 객체로 존재하게 한다.

data Jpa를 통해 객체를 간단하게 CRUD하는 기능으로만 사용하고 실제 서비스에서는 toDomain()과 같은 코드를 통해 도메인으로 바꾼 객체를 사용하여 서비스 로직을 완성한다.

모든 로직을 처리 후 도메인 코드를 엔티티로 변환시켜 save해주면 일련의 과정이 끝나게 된다.

이를 위해서는 레파지토리를 한단계 더 추상화 시키면 더욱 효과적인 분리를 할 수있다.



### 예시를 위한 코드

```kotlin
// Entity.kt
@Entity
class Entity(
    @Id
    @Gernerated(strategy = IDENTITY)
    val id: Long? = null,
    val name: String
) {
    constructor(domain: Domain) : this(domain.id, domain.name)
    fun toDomain() = Domain(id, name)
}

// Domain.kt
data class Domain(
    val id: Long? = null,
    val name: String
) {
}

// DomainRepository
interface DomainRepository {
    fun create(domain: Domain): Domain
    
    @Repository
    class DomainRepositoryImpl(
        private val jpaRepository: DomainJpaRepository,
        private val jdbcRepository: DomainJdbcRepository,
        private val dslRepository: DomainDslRepository,
    ){
        override fun create(domain: Domain): Domain = 
            jpaRepository.save(Entity(domain)).toDomain() 
    }
}
```

도메인 객체와 엔티티 객체를 분리하고 엔티티는 레포지토리 계층에서만 사용되는 객체로서 활용을 한다.

또한 위에서 말했던것처럼 레포지토리를 한번 더 추상화하여 구현된 여러 레포지토리들을 유기적으로 활용한다.



이것은 도메인과 엔티티 뿐만이 아니라 모든 외부 의존성이 필요한 코드에서 활용할 수 있다.

Redis에서 사용되는 '@RedisHash', gRPC에서 사용되는 proto 객체들 모두 도메인과 분리하여 사용하면

엔티티나 도메인 객체가 다른 외부 설정들이나 어노테이션으로 더렵혀지지 않도록 관리할 수 있다.

&#x20;







