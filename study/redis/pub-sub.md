# Pub / Sub

## Pub / Sub 패턴

메시징 모델의 하나로 발행과 구독 역할로 개념화 한 형태

발행자와 구독자는 서로에 대한 정보 없이 특정 주제를 매개로 송수신



## 메시징 미들웨어 사용의 장점

* 비동기 : 통신의 비동기 처리
* 낮은 결합도 : 송신자와 수신자가 직접 서로 의존하지 않고 공통 미들웨어에 의존
* 탄력성 : 구성원들간에 느슨한 연결로 인해 일부 장애가 생겨도 영향이 최소화 됨

ex) Kafka, RabitMQ, ActiveMq 등등..



## Redis pub / sub 특징

* 메시지가 큐에 저장되지 않음
* Kafka의 컨슈머 그룹같은 분산 처리 개념이 없음
* 메시지 발행시 push 방식으로 subscriber들에게 전송
* subscriber가 늘어날수록 성능이 저하



## Redis의 pub / sub 유즈케이스

* 실시간으로 빠르게 전송되어야 하는 메시지
* 메시지 유실을 감내할 수 있는 케이스
* 최대 1회 전송 패턴이 적합한 경우
* subscriber들이 다양한 채널을 유동적으로 바꾸면서 한시적으로 구독하는 경우
