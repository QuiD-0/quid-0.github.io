# What is Kafka

## Kafka란

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

Apache Kafka는 분산 스트리밍 플랫폼으로,&#x20;

실시간 데이터 파이프라인과 스트리밍 애플리케이션을 구축하는 데 사용됩니다.&#x20;

Kafka의 주요 특징 중 몇 가지는 다음과 같습니다:

1. 높은 처리량: Kafka는 높은 처리량을 처리할 수 있도록 설계되었으며, 대량의 데이터를 빠르게 처리하고 전송할 수 있습니다.
2. 확장성: Kafka는 가로 방향으로 확장할 수 있어, 클러스터를 추가하여 더 많은 처리량을 처리할 수 있습니다.
3. 메시지 저장: Kafka는 메시지를 저장하고 관리할 수 있는 기능을 제공하며, 이를 통해 데이터를 처리하는 시점을 지연시킬 수 있습니다.
4. 보안: Kafka는 인증, 보안 그리고 권한 관리 기능을 제공하여, 데이터를 안전하게 처리할 수 있도록 보장합니다.



## Kafka 구성 요소

### Producer

Producer는 메시지를 생성하는 역할을 한다.\
Producer는 메시지를 생성하고, 메시지를 큐에 저장한다.\
메시지를 큐에 저장하는 것을 publish라고 한다.

### Consumer

Consumer는 큐에 저장된 메시지를 읽어오는 역할을 한다.

### Topic

Topic은 메시지를 저장하는 큐이다.

### Partition

Partition은 Topic을 분할한 것이다.

### Broker

Broker는 Kafka를 구성하는 서버이다.
