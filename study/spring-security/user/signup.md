# SignUp

```kotlin
fun interface SignUp {
    operator fun invoke(user: User)

    @Service
    class SignUpUseCase(
        private val userRepository: UserRepository,
        private val passwordEncoder: PasswordEncoder
    ) : SignUp {

        override fun invoke(user: User) {
            isUserExist(user)
                ?.let { throw IllegalArgumentException("Username already exists") }
                ?: passwordEncoder.encode(user.password)
                    .let { user.encodePassword(it) }
                    .let { userRepository.save(it) }
        }

        private fun isUserExist(user: User) =
            takeIf { userRepository.existsByUsername(user.username) }
    }
}
```
