---
description: security에서 사용되는 UserDetails와 UserDetailsService를 정의 한다.
---

# USER

### UserDetails&#x20;

```kotlin
data class UserDetail (
    val user: User,
    val authority: UserAuthority
): UserDetails{
    val name: String
        get() = user.name

    override fun getAuthorities(): MutableCollection<out GrantedAuthority> {
        return mutableListOf(GrantedAuthority { "ROLE_" + authority.authorityName })
    }

    override fun getPassword(): String {
        return user.password
    }

    override fun getUsername(): String {
        return user.username
    }

    override fun isAccountNonExpired(): Boolean {
        return !user.deleted
    }

    override fun isAccountNonLocked(): Boolean {
        return true
    }

    override fun isCredentialsNonExpired(): Boolean {
        return true
    }

    override fun isEnabled(): Boolean {
        return true
    }

}
```

```kotlin
data class User(
    val userSeq: Long? = null,
    val username: String,
    val password: String,
    val email: String,
    val name: String,
    val authorityId: Long = 1,
    val deleted: Boolean = false,
) {
    init {
        require(username.length in 4..10) { "ID는 4자 이상 10자 이하 여야 합니다." }
        require(email.matches(emailRegex)) { "이메일 형식이 올바르지 않습니다." }
    }

    fun encodePassword(encodedPassword : String): User {
        return this.copy(password = encodedPassword)
    }

    companion object{
        private val emailRegex = Regex("^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,6}$")
    }
}
```

```kotlin
data class UserAuthority(
    val authoritySeq: Long? = null,
    val authorityName: AuthorityType,
    val regDate: LocalDateTime = LocalDateTime.now(),
    val deleted: Boolean = false,
) {
}
```

```kotlin
enum class AuthorityType {
    ADMIN, USER
}
```



### UserDetailsService

```kotlin
@Service
class UserAuthService(
    private val userRepository: UserRepository,
    private val userAuthorityRepository: UserAuthorityRepository
): UserDetailsService {
    override fun loadUserByUsername(username: String): UserDetails {
        val user = userRepository.findByUsername(username)
        val authority = userAuthorityRepository.findById(user.authorityId)
        return UserDetail(user, authority)
    }
}
```
