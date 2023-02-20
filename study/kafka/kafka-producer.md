# kafka producer

```java
@Slf4j
@Component
@RequiredArgsConstructor
public class AlarmProducer {

    private final KafkaTemplate<Long, AlarmEvent> kafkaTemplate;
    //원하는토픽 지정 
    private String topic = "alarm";

    public void send(AlarmEvent event) {
        kafkaTemplate.send(topic, event.getReceiveUserId(), event);
        log.info("AlarmEvent sent to kafka topic: {}", event);
    }

}
```



## 사용

```java
 @Override
@Transactional
public void createComment(CommentCreateRequest request, Long userId, Pageable pageable) {
    Post post = postRepository.findByIdOrThrow(request.getPostId());

    commentRepository.saveById(request.getPostId(), userId, request.getContent());

    AlarmEvent alarmEvent = AlarmEvent.builder().receiveUserId(post.getUser().getId())
        .type(AlarmType.NEW_COMMENT_ON_POST).args(
            AlarmArgs.builder().fromUserId(userId).targetId(post.getId()).build())
        .build();
    
    alarmProducer.send(alarmEvent);
}
```
