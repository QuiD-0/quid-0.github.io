# 레디스 기본 개념

### <mark style="background-color:yellow;">자료구조</mark>

1. String

```
SET hello world 

GET hello
"world"
```

NX 와 함께 사용하면 지정한 키가 없을 때만 새로운 키를 저장\
XX 옵션은 키가 있을때만 새로운 값으로 덮어쓴다.

최대 512MB의 문자열을 저장할 수 있으며 이미지와 같은 바이트 값, HTTP 응답값 등의 데이터도 저장 가능



2. list

```
LPUSH myList A
RPUSH myList B
RPUSH myList C

LRANGE myList 0 -1
"A"
"B"
"C"

LPOP myList
"A"

LTRIM myList 0 1 
"B"
"C"
```

LTRIM은 시작과 끝 아이템의 인덱스 외의 아이템은 모두 삭제하지만 아이템을 반환하지는 않는다. \
LPUSH와 LTRIM을 사용하여 고정길이 큐를 유지할 수 있다.&#x20;



3. hash

```
HSET Product:123 Name "Happy Hacking"
HSET Product: 123 TypeID 35

HGET Product: 123 TypeID
"35"
```

필드와 값 모두 문자열 데이터로 저장된다.



3. set

```
SADD mySet A
SADD mySet B B C

SMEMBERS mySet
"A"
"B"
"C"
```

아이템은 중복 저장되지 않으며 집합 연산을 제공한다.

4. sorted set

```
ZADD rank:20240101 100 user:A
ZADD rank:20240101 200 user:B
ZADD rank:20240101 300 user:C

ZRANGE rank:20240101 1 2
"user:B"
"user:C"

ZPOP rank:20240101
"user:A"
```

값에 따라 정렬되는 문자열의 집합\
같은 스코어를 가진 아이템은 데이터의 사전 순으로 정렬된다.&#x20;

인덱스, 스코어, 사전순으로 조회가능하다.



5. 비트맵

```
SETBIT myBitmap 2 1

GETBIT myBitmap 2
(integer) 1
```

비트맵은 String 자료구조에 bit연산을 수행할 수 있도록 확장한 형태

저장 공간을 획기적으로 줄일 수 있다. string과 동일하게 512MB를 저장할 수 있으며 이는 약 40억개의 flag를 저장할 수 있는 수치&#x20;



6. hyperloglog

대량 데이터에서 중복되지 않는 고유한 값을 **집계**할 때 사용할 수 있는 데이터 구조

최대 12KB의 크기를 가지며 최대 2^64 개의 아이템을 저장할 수 있으며 오차는 0.81%이다.&#x20;



7. geospatial

지도의 경도, 위도 데이터의 쌍의 집합

두 아이템 사이의 거리를 쉽게 반환할 수 있다.&#x20;

특정 거리 내에 있는 아이템들 또한 검색할 수 있다.&#x20;



8. stream

레디스를 메시지 브로커로서 사용할 수 있게 하는 자료구조, 데이터를 분산 처리할 수 있다.



### <mark style="background-color:yellow;">키 관리 방법</mark>

1. 키가 존재하지 않을 때 아이템을 넣으면 아이템을 **삽입하기 전에 빈 자료 구조를 생성**
2. 모든 아이템을 삭제하면 **키도 자동으로 삭제** (stream 제외)
3. 키가 없는 상태에서 키 삭제, 아이템 삭제와 같은 커맨드를 수행하면 에러가 아닌 \
   **키가 있으나 아이템이 없는 것처럼 동작**

### <mark style="background-color:yellow;">키와 관련된 커맨드</mark>&#x20;

조회&#x20;

**EXISTS key**\
키가 존재하는지 확인 존재하면 1 없으면 0을 반환&#x20;

**KEYS \[Pattern]**\
레디스에 존재하는 모든 키를 조회하는 커맨드 \
매우 위험한 커맨드로 사용안하는게 좋다. - 싱글 스레드로 동작하는데 실행 시간이 오래걸리기 때문&#x20;

**SCAN cursor \[pattern]**\
커서를 기반으로 특정 범위의 키만 조회 \
우선 스캔 후 필터링을 해서 반환하기 때문에 결과가 적을 수 있다.&#x20;

**SORT key \[LIMIT, ASC | DESC]**\
list, set, sorted set에서만 사용할 수 있는 커맨드, 키 내부를 정렬해서 반환한다.&#x20;

**RENAME key newKey**\
키의 이름을 변경하는 커맨드&#x20;

**COPY source destination** \
소스에 저장된 키를 데스티네이션 키에 복사한다. 이미 존재한다면 에러가 반환되는데 \
REPLACE옵션을 사용하면 destination의 키를 삭제한 뒤 복사한다.

**TYPE** \
키의 자료 구조 타입을 반환

**OBEJCT** \
키에 대한 상세 정보를 반환



삭제

**FLUSHALL  \[async | sync]**\
레디스에 저장된 모든 키를 삭제한다.&#x20;

**DEL key** \
키와 키에 저장된 모든 아이템을 삭제

**UNLINK** \
키와 데이터를 삭제하지만 백그라운드에서 다른 스레드에 의해 처리된다. 우선 연결된 데이터의 연결을 끊는다.\


만료 시간

EXPIRE key seconds \
키가 만료될 시간을 초 단위로 정의&#x20;

* NX: 만료시간이 정의돼 있지 않을 경우만 수행
* XX: 만료시간이 정의돼 있을 때만 수행
* GT: 새로 입력한 초가 더 클때만 수행
* LT: 새로 입력한 초가 더 작을때만 수행&#x20;

**EXPIREAT** \
특정 타임스탬프에 만료&#x20;

**EXPIRETIME** \
키가 삭제되는 시간을 초 단위로 반환, 만료시간이 안정해진 경우 -1, 키가 없을 경우 -2

**TTL** \
키가 몇 초 뒤에 만료되는지 반환&#x20;







