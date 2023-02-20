# Logger

## Custom Logger

```java
@Slf4j
@RequiredArgsConstructor(staticName = "of")
public class FeignLogger extends Logger {

    private static final int SLOW_API_TIME = 3_000;
    public static final String SLOW_API_MESSAGE = "Slow API call.";

    @Override
    protected void log(String configKey, String format, Object... args) {
        System.out.printf(methodTag(configKey) + format + "%n", args);
    }

    @Override
    protected void logRequest(String configKey, Level logLevel, Request request) {
        log.info("Request: " + request.toString());
    }

    @Override
    protected Response logAndRebufferResponse(String configKey, Level logLevel, Response response,
        long elapsedTime) throws IOException {
        if (elapsedTime > SLOW_API_TIME) {
            log.warn(SLOW_API_MESSAGE);
        }
        return super.logAndRebufferResponse(configKey, logLevel, response, elapsedTime);
    }
}
```

`feign.Logger`를 상속 받아야 한다.
