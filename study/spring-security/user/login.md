# LogIn

```kotlin
fun interface LogIn {
    operator fun invoke(username: String, password: String): TokenResponse

    @Service
    class LoginUseCase(
        private val authenticationManagerBuilder: AuthenticationManagerBuilder,
        private val tokenEncoder: TokenEncoder,
        private val refreshTokenRepository: RefreshTokenRepository,
    ) : LogIn {

        override fun invoke(username: String, password: String): TokenResponse {
            val accessToken = UsernamePasswordAuthenticationToken(username, password)
                .let { authenticationManagerBuilder.getObject().authenticate(it) }
                .let { it.principal as UserDetail }
                .let {
                    AccessToken(it.username)
                }
                .let { tokenEncoder(it) }

            val refreshToken = RefreshToken()
                .also { refreshTokenRepository.save(UserToken(username, it.payload.jti)) }
                .run { tokenEncoder(this) }

            return TokenResponse(accessToken, refreshToken)
        }

    }
}
```

로그인시 access토큰과 refresh토큰을 함께 발급 해 준다.
