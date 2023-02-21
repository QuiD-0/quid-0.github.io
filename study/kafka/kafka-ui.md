# Kafka UI



## 상세 정보

[https://github.com/provectus/kafka-ui](https://github.com/provectus/kafka-ui)

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

## docker-compose.yaml

```yaml
  kafka-ui:
    image: provectuslabs/kafka-ui
    container_name: kafka-ui
    ports:
      - "9000:8080"
    restart: always
    environment:
      - KAFKA_CLUSTERS_0_NAME=local
      - KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=kafka-1:29092,kafka-2:29093,kafka-3:29094
      - KAFKA_CLUSTERS_0_ZOOKEEPER=zookeeper-1:22181
    depends_on:
      - kafka-1
      - kafka-2
      - kafka-3
      - zookeeper-1
```

kafka cluster with docker-compose에 있는 파일 하위에 작성

