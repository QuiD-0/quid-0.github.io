# Security세팅

### SecurityConfig

```kotlin
@Configuration
@EnableWebSecurity
class SecurityConfig(
    private val jwtAuthenticationEntryPoint: JwtAuthenticationEntryPoint,
    private val userAuthService: UserAuthService,
    private val jwtTokenDecoder: TokenDecoder,
) {

    @Bean
    fun filterChain(http: HttpSecurity): SecurityFilterChain =
        http.httpBasic { it.disable() }
            .csrf { it.disable() }
            .formLogin { it.disable() }
            .exceptionHandling{
                it.authenticationEntryPoint(jwtAuthenticationEntryPoint)
            }
            .addFilterBefore(
                JwtAuthenticationFilter(
                    jwtTokenDecoder,
                    userAuthService
                ),
                UsernamePasswordAuthenticationFilter::class.java
            )
            .sessionManagement { it.sessionCreationPolicy(SessionCreationPolicy.STATELESS) }
            .authorizeHttpRequests{
                it.requestMatchers(*allow()).permitAll()
                it.requestMatchers(*admin()).hasRole("ADMIN")
                it.anyRequest().authenticated()
            }
            .build()

    @Bean
    fun passwordEncoder(): PasswordEncoder = BCryptPasswordEncoder()

    private fun allow(): Array<RequestMatcher> = arrayOf(
        AntPathRequestMatcher("/", "GET"),
        AntPathRequestMatcher("/api/user/login", "POST"),
        AntPathRequestMatcher("/api/user/signup", "POST"),
        AntPathRequestMatcher("/api/token/refresh", "POST"),
    )

    private fun admin(): Array<RequestMatcher> = arrayOf(
        AntPathRequestMatcher("/api/admin/**/", "POST"),
    )

}
```

로그인, 회원가입, 토큰 리프레시 API는 모두 접근 가능 하며

admin API는 ADMIN권한이 있어야 접근 가능하다.

### JwtAuthenticationFilter

```kotlin
class JwtAuthenticationFilter(
    private val tokenDecoder: TokenDecoder,
    private val userAuthService: UserAuthService
): OncePerRequestFilter() {
    override fun doFilterInternal(
        request: HttpServletRequest,
        response: HttpServletResponse,
        filterChain: FilterChain
    ) {
        try {
            val accessToken = getToken(request)
                .run { tokenDecoder(this) }
            require(accessToken.isNotExpired()) { "access token is expired" }

            val user = userAuthService.loadUserByUsername(accessToken.username)

            UsernamePasswordAuthenticationToken(user, null, user.authorities)
                .also { SecurityContextHolder.getContext().authentication = it }

        } catch (e: Exception) {
            request.setAttribute("Exception", e)
            SecurityContextHolder.clearContext()
        }
        filterChain.doFilter(request, response)
    }

    private fun getToken(request: HttpServletRequest): String {
        val header = request.getHeader("Authorization")
        require(header.startsWith("Bearer ")){ "Authorization header must start with Bearer" }
        return header.substring(7)
    }

}
```

### JwtAuthenticationEntryPoint

```kotlin
@Component
class JwtAuthenticationEntryPoint(
    @Qualifier("handlerExceptionResolver")
    private val resolver: HandlerExceptionResolver
): AuthenticationEntryPoint {
    override fun commence(
        request: HttpServletRequest?,
        response: HttpServletResponse?,
        authException: AuthenticationException?
    ) {
        resolver.resolveException(request!!, response!!, null, request.getAttribute("Exception") as Exception)
    }
}
```

`JwtAuthenticationEntryPoint` 를 `SecurityConfig` 에 등록하여 필터에서 발생하는 Exception을  ExceptionHandler에서  핸들링 할수 있게 한다.



### Error handler&#x20;

```kotlin
@RestControllerAdvice
class ErrorHandling {

    @ExceptionHandler(Exception::class)
    fun handleException(ex: Exception): ResponseEntity<Error> =
        ResponseEntity<Error>(
            Error{ ex.message ?: "Unknown Error" },
            HttpStatusSelector { ex }
        ).also { ex.printStackTrace() }
}
```

```kotlin
interface HttpStatusSelector {
    operator fun invoke(ex: () -> Exception): HttpStatus

    class HttpStatusSelectorImpl : HttpStatusSelector {
        override fun invoke(ex: () -> Exception): HttpStatus =
            when (ex) {
                is IllegalArgumentException -> HttpStatus.BAD_REQUEST
                is NoSuchElementException -> HttpStatus.NOT_FOUND
                is IllegalStateException -> HttpStatus.CONFLICT
                is SignatureException -> HttpStatus.UNAUTHORIZED
                is ExpiredJwtException -> HttpStatus.UNAUTHORIZED
                is MalformedJwtException -> HttpStatus.UNAUTHORIZED
                else -> HttpStatus.INTERNAL_SERVER_ERROR
            }
    }
    companion object: HttpStatusSelector by HttpStatusSelectorImpl()
}
```

















