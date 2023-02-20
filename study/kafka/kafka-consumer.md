# kafka consumer

```java
@Slf4j
@Component
@RequiredArgsConstructor
public class AlarmConsumer {

    //원하는 service, repository 사용
    private final SseService sseService;

    @KafkaListener(topics = "alarm")
    public void consumeAlarmEvent(AlarmEvent event, Acknowledgment ack) {
        log.info("AlarmEvent consumed from kafka topic: {}", event);
        //원하는 작업 실행
        sseService.send(event.getType(), event.getArgs(), event.getReceiveUserId());
        
        ack.acknowledge();
    }
}
```
