# 메시지 브로커

당장 메시지 처리를 못하더라도 보낸 메시지를 어딘가에 쌓아 둔 뒤 나중에 처리할 수 있는 채널



**메시징 큐**

* 생산자(producer)와 소비자(consumer)로 지정&#x20;
* 2개의 서비스에 같은 메시지를 보내야 할 때 2개의 메시지 큐에 2개의 데이터를 푸시&#x20;
* 소비자가 데이터를 읽어갈 때 큐에서 데이터를 삭제 (카프카는 제외)
* 1:1 상황에서 유리

**이벤트 스트림**

* 발행자(publisher)와 구독자(subscriber)로 지정
* 특정 저장소에 하나의 메시지를 보내고 여러 서비스의 소비자들은 같은 메시지를 풀
* 구독자가 읽어간 데이터는 바로 삭제되지 않고 특정 기간 동안 저장
* N:N 상황에서 유리



**레디스를 메시지 브로커로 사용**\
pub/sub을 통해 제공

```
PUBLISH hello world

SUBSCRIBE hello
```

* 모든 데이터는 한 번 채널 전체에 전파된 뒤 삭제된다.
* 메시지 전달을 보장하지 않는다. (fire-and-forget 패턴이 필요한 알림에서 주로 사용)
* PSUBSCRIBE mail-\* 라는 커맨드를 사용하면 mail-track, mail-album 등 \
  앞부분이 mail- 로 시작하는 모든 채널에 전파된 메시지를 모두 수신할 수 있다.

**클러스터를 위한 pub/sub**

레디스 클러스터에서 pub/sub을 사용할 때 , 메시지를 발행하면 클러스터에 속한 **모든 노드에 자동으로 전달** \
아무 노드에 연결하여 SUBSCRIBE 하면 데이터를 수신할 수 있다.&#x20;

불필요한 노드까지 전파되어 비효율적이다.

**Sharded pub/sub**

각 채널은 슬롯에 매핑, 같은 슬롯을 가지고 있는 노드간에만 pub/sub 메시지를 전달 \
다른 슬롯을 사용하는 노드에서 SPUBLISH / SSUBSCRIBE 하면 알맞은 노드로 **리다이렉트** 된다.&#x20;









