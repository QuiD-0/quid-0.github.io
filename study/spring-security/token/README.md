# TOKEN

JWT 에서 사용되는 토큰은 크게 Header, Payload, Signature 이루어져 있습니다.

### Header&#x20;

```kotlin
data class Header(
    val typ: String,
    val alg: String,
) {
    val value: Map<String, String>
        get() = mapOf("typ" to typ, "alg" to alg)

    companion object{
        fun default() = Header("JWT", "HS256")
    }
}
```

### Payload

```kotlin
data class Payload(
    val jti: String = UUID.randomUUID().toString(),
    val iss: String = "QUID_LAB",
    val sub: TokenType,
    val iat: LocalDateTime = LocalDateTime.now(),
    val exp: LocalDateTime,
    val username: String,
) {
    constructor(claims: Claims) : this(
        jti = claims.id,
        iss = claims.issuer,
        sub = TokenType.of(claims.subject),
        iat = claims.issuedAt.toInstant().atOffset(ZoneOffset.UTC).toLocalDateTime(),
        exp = claims.expiration.toInstant().atOffset(ZoneOffset.UTC).toLocalDateTime(),
        username = claims["username"] as String
    )

    init {
        require(exp.isAfter(iat)) { "토큰 만료시간은 발급시간보다 빠를수 없습니다." }
    }

    companion object {
        fun accessType(username: String) = Payload(
            sub = TokenType.ACCESS,
            exp = LocalDateTime.now().plusMinutes(30),
            username = username
        )

        fun refreshType() = Payload(
            sub = TokenType.REFRESH,
            exp = LocalDateTime.now().plusDays(7),
            username = ""
        )
    }
}

enum class TokenType {
    ACCESS, REFRESH;
}
```

### Signature

시그니쳐는 따로 코드화 시키지 않고 application.yaml에 키값으로 추가하였습니다

```yaml
jwt:
  secret: this_is_token_secret_code_please_replace_this_in_production
```
