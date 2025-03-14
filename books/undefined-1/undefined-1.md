# 레디스와 캐시

### <mark style="background-color:yellow;">캐시란?</mark>

데이터의 원본보다 더 빠르고 효율적으로 액세스할 수 있는 임시 데이터 저장소&#x20;

<div align="left"><figure><img src="../../.gitbook/assets/스크린샷 2025-03-14 오후 9.20.57.png" alt=""><figcaption></figcaption></figure></div>

**성능을 효과적으로 개선할 수 있는 상황**

* 원본 데이터 저장소에서 **쿼리 시간이 오래걸리거나**, **매번 계산**을 통해 데이터를 가져와야하는 경우
* 캐시에 저장된 데이터가 **잘 변하지 않는** 데이터일 경우
* 캐시에 저장된 데이터가 **자주 검색**되는 데이터일 경우&#x20;

**캐시로서의 레디스의 장점**

* 모든 데이터가 메모리 위에 존재하기 때문에 데이터 접근이 빠르다.&#x20;
* 자체적으로 고가용성 기능을 가지고 있는 솔루션&#x20;
* 클러스터를 사용하면 수평확장이 굉장히 간단해진다.&#x20;



### <mark style="background-color:yellow;">캐싱전략</mark>

#### 읽기 전략

**Look aside**&#x20;

* 캐시를 우선 조회한 뒤 있으면(캐시 히트) 가지고 오고 없으면 DB를 조회한다.(캐시 미스)&#x20;
* 캐시 워밍 작업을 통해 도입 초기 효율성을 높일수 있다.&#x20;

#### 쓰기 전략&#x20;

**Write through**

* 데이터를 업데이트 시킬때마다 Redis와 DB에 함께 업데이트&#x20;
* 쓰기 과정에서 시간이 더 소요될 수 있다.
* 다량의 쓰기 작업이 발생하면 그때 그때 DB에 업데이트를 해주어야한다.

**Write back**

* 레디스에만 우선적으로 업데이트를 한 뒤 일정 주기마다 비동기적으로 DB에 업데이트
* 캐시에 문제가 발생하면 일정 주기안의 데이터 유실 가능성이 존재

**Cache invalidation**

* DB에 값을 업데이트할 때마다 캐시에서는 데이터를 삭제
* Write through의 단점을 보완한 방법

### <mark style="background-color:yellow;">메모리 관리와 maxmemory-policy</mark>

제한적인 메모리가 가득 차는 상황이 발생했을경우 어떤 키를 삭제할지 결정

**Noeviction**

* 임의로 데이터를 삭제하지 않고 에러를 반환
* 데이터 관리를 캐시에게 맡기지 않고 애플리케이션 측에서 관리하겠다는 것을 의미
* 권장하지 않는 설정값&#x20;

**LRU eviction**

* 가장 **오래된** 데이터부터 제거, 효율성을 위해 근사치로 사용&#x20;
* Volatile-LRU : 만료 시간이 설정되어 있는 키에 한해서 LRU방식으로 제거\
  모두 만료 시간이 지정돼 있지 않다면 noeviction과 동일
* Allkeys-LRU  : 모든 키에 대해 LRU 알고리즘을 이용해 제거&#x20;

**LFU eviction**

* 가장 **사용되지 않는** 데이터부터 제거, 효율성을 위해 근사치로 사용
* Volatile-LFU : 만료 시간이 설정되어 있는 키에 한해서 LFU방식으로 제거\
  마찬가지로 만료시간이 없다면 noeviction
* Allkeys-LFU  : 모든 키에 대해 LFU 알고리즘을 이용해 제거&#x20;

**Random eviction**

* **임의로** 골라서 제거, 삭제할 키를 계산하지 않아도 된다는 점에서 부하를 줄이는 방법&#x20;
* Volatile-random : 만료 시간이 설정되어 있는 키에 한해서 랜덤으로 제거
* Allkeys-random  : 모든 키에 대해 랜덤으로 제거

**Volatile-ttl**&#x20;

* 만료시간이 가장 작은 키를 제거, 근사치 사용&#x20;



**캐시 스탬피드 현상**

look aside 패턴을 통해 읽기를 하는 상황에서 특정 키가 만료 된다면 \
여러 어플리케이션에서 동시다발적으로 DB를 중복으로 읽고 레디스에 중복으로 쓰게된다.

<div align="left"><figure><img src="../../.gitbook/assets/스크린샷 2025-03-14 오후 10.33.12.png" alt=""><figcaption></figcaption></figure></div>

* 만료 시간을 **너무 짧지 않게** 설정
* 키가 만료되기 전에 값을 **미리 갱신**

**PER 알고리즘**

```
currentTime - ( timeToCompute * beta * log(rand()) ) > expiry
```

만료 시간이 다가올 때 더 자주 만료된 캐시 항목을 확인하게 된다.





