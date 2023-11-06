# Refresh

```kotlin
fun interface RefreshAccessToken {
    operator fun invoke(accessTokenString: String, refreshTokenString: String): TokenResponse

    @Service
    class RefreshAccessTokenImpl(
        private val tokenEncoder: TokenEncoder,
        private val tokenDecoder: TokenDecoder,
        private val refreshTokenRepository: RefreshTokenRepository
    ) : RefreshAccessToken {
        override fun invoke(accessTokenString: String, refreshTokenString: String): TokenResponse {
            val accessToken: Token = tokenDecoder(accessTokenString)
            val refreshToken: Token = tokenDecoder(refreshTokenString)
            require(accessToken.isExpired()) { "access token is not expired" }
            require(refreshToken.isNotExpired()) { "refresh token is expired" }

            val refreshJti = refreshTokenRepository.findByUsername(accessToken.username)
            require(refreshToken.payload.jti == refreshJti) { "refresh token is not matched" }

            val newAccessToken = getNewAccessToken(accessToken.username)
            val newRefreshToken = getNewRefreshToken(accessToken.username)

            return TokenResponse(newAccessToken, newRefreshToken)
        }

        private fun getNewRefreshToken(username: String) =
            RefreshToken()
                .also {
                    refreshTokenRepository.save(
                        UserToken(
                            username,
                            it.payload.jti
                        )
                    )
                }.let { tokenEncoder(it) }

        private fun getNewAccessToken(username: String) =
            AccessToken(username)
                .run { tokenEncoder(this) }
    }
}
```

1. 엑세스 토큰이 만료되었는지 확인
2. 리프레시 토큰이 만료되었는지 확인&#x20;
3. refreshToken id와 UserName을 매핑한 테이블에서 리프레시 토큰이 일치하는지 확인
4. 새로운 토큰 발급 후 refreshToken id를 저장
