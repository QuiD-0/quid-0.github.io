# Transaction

데이터베이스 관리 시스템 또는 유사한 시스템에서 상호작용의 단위

## 트랜잭션의 특성

**원자성**(Atomicity): 트랜잭션과 관련된 작업들이 부분적으로 실행되다가 중단되지 않는 것을 보장

**일관성**(Consistency): 트랜잭션이 실행을 성공적으로 완료하면 언제나 일관성 있는 데이터베이스 상태로 유지하는 것을 의미

**격리성**(Isolation): 트랜잭션을 수행 시 다른 트랜잭션의 연산 작업이 끼어들지 못하도록 보장

**영구성**(Durability): 한 번 저장된 트랜잭션을 영구히 보존

## 동시성과 일관성

트랜잭션의 일관성이 높을수록 동시성이 적어지게 되는데&#x20;

이는 다중의 접속이 발생하는 시스템에서 응답속도의 문제가 발생 할 수 있다.

이것을 **트랜잭션 격리 수준**을 통해 적절히 조절 할 수 있다.



## Isolation Level

<table><thead><tr><th> </th><th width="122">Dirty Read</th><th width="216">Non Reapeatable Read</th><th>Phantom Read</th></tr></thead><tbody><tr><td>Read Uncommitted</td><td>O</td><td>O</td><td>O</td></tr><tr><td>Read Committed</td><td></td><td>O</td><td>O</td></tr><tr><td>Repeatable Read</td><td></td><td></td><td>O</td></tr><tr><td>Serializable Read</td><td></td><td></td><td></td></tr></tbody></table>



Serializable Read로 갈수록 동시 처리량이 낮다. 대신 이상현상이 적어진다.
