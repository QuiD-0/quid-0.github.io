# TOKEN <-> JWT

우선JWT를 사용하기 위해 의존성을 추가해 준다.

```gradle
dependencies {
	...
	implementation("io.jsonwebtoken:jjwt-api:0.11.2")
	runtimeOnly("io.jsonwebtoken:jjwt-impl:0.11.2")
	runtimeOnly("io.jsonwebtoken:jjwt-jackson:0.11.2")
	...
}
```

### Token -> JWT

```kotlin
fun interface TokenEncoder {

    operator fun invoke(token: Token): String

    @Service
    class JwtTokenEncoder(
        @Value("\${jwt.secret}")
        private val secret: String
    ) : TokenEncoder {

        override fun invoke(token: Token): String = Jwts.builder().apply {
            setHeader(token.header.value)
            setSubject(token.payload.sub.name)
            setIssuer(token.payload.iss)
            setExpiration(Date(token.payload.exp.toEpochSecond(ZoneOffset.UTC) * 1000))
            setIssuedAt(Date(token.payload.iat.toEpochSecond(ZoneOffset.UTC) * 1000))
            setId(token.payload.jti)
            claim("username", token.username)
        }.signWith(
            Keys.hmacShaKeyFor(secret.toByteArray(StandardCharsets.UTF_8)),
            SignatureAlgorithm.HS256
        ).compact()

    }
}
```

claim("username", token.username)

이 부분을 통해 필요한 claim을 JWT에 추가해 준다



### JWT -> TOKEN

```kotlin
interface TokenDecoder {

    operator fun invoke(token: String): Token

    @Service
    class JwtTokenDecoder(
        @Value("\${jwt.secret}")
        private val secret: String
    ): TokenDecoder {

        override fun invoke(token: String): Token =
            try {
                Jwts.parserBuilder().setSigningKey(secret.toByteArray(StandardCharsets.UTF_8)).build()
                    .parse(token)
                    .let { it.body as Claims }
                    .let { Payload(it) }
                    .let { AccessToken(it) }
            } catch (exp: ExpiredJwtException){
                Payload(exp.claims)
                    .run { AccessToken(this) }
            }
    }
}
```

토큰을 JWT로 만들때 발생하는 ExpiredJwtException는 무시하여&#x20;

JWT필터에서 체크한다.
