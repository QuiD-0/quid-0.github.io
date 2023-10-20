---
description: Sealed interface를 활용하기
---

# 코틀린으로 API Response정의하기

### ApiResponse.kt

```kotlin
sealed interface ApiResponse<RESPONSE> {
    val timeStamp: String
        get() = LocalDateTime.now().format(ofPattern("yyyy-MM-dd HH:mm:ss"))
}

data class Success<RESPONSE>(
    val data: RESPONSE
) : ApiResponse<RESPONSE>

data class Error<ERROR>(
    val message: String,
) : ApiResponse<ERROR>

```



### ErrorHandling.kt

```kotlin
@RestControllerAdvice
class ErrorHandling {

    @ExceptionHandler(Exception::class)
    fun handleException(ex: Exception): ResponseEntity<Error<String>> =
        ResponseEntity<Error<String>>(
            Error(ex.message ?: "Unknown Error"),
            determineStatusCode(ex)
        ).also { ex.printStackTrace() }

    private fun determineStatusCode(ex: Exception): HttpStatus =
        when (ex) {
            is IllegalArgumentException -> BAD_REQUEST
            is NoSuchElementException -> NOT_FOUND
            is IllegalStateException -> CONFLICT
            is RuntimeException -> INTERNAL_SERVER_ERROR
            else -> INTERNAL_SERVER_ERROR
        }

}
```

