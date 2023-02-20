# Sharded Cluster

> 모든 Shard는 Replica Set으로 구성되어 있다.

## 장단점

### 장점

* 용량의 한계를 극복할 수 있다.
* 데이터 규모와 부하가 크더라도 처리량이 좋다.
* 고가용성을 보장한다.
* 하드웨어에 대한 제약을 해결할 수 있다.

### 단점

* 관리가 비교적 복잡하다.
* Replica Set과 비교해서 쿼리가 느리다.



## Sharding vs Partitioning

`shard`  - 하나의 큰 데이터를 여러 서브셋으로 나누어 여러 인스턴스에 저장&#x20;

`partitioning`  - 하나의 인스턴스에 여러 테이블로 나누어 저장하는 기술&#x20;



> Collection 단위로 Sharding이 가능하다.
>
> 꼭 Router를 통해 접근한다.
>
> Range와 Hash Sharding이 있지만 가능한 Hash Sharding을 사용
