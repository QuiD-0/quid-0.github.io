# 아키텍처

### <mark style="background-color:yellow;">폰 노이만 아키텍처</mark>

1. **처리 장치** - 명령어 실행
2. **제어 장치** - 처리장치에서 명령어를 실행하도록 주도, 처리 + 제어 = CPU
3. **기억 장치** - 명령어 저장
4. **입력 장치** - 데이터와 명령어를 컴퓨터에 로딩
5. **출력 장치** - 실행 결과를 저장하거나 수신
6. **버스** - 장치들을 연결하고 서로에게 정보를 전송

<div align="left"><figure><img src="../../.gitbook/assets/스크린샷 2025-03-14 오전 10.35.54.png" alt=""><figcaption></figcaption></figure></div>

### <mark style="background-color:yellow;">프로그램 명령어 사이클</mark>

**Fetch - Decode - Execute - Store**



### 클럭 주도 실행&#x20;

<div align="left"><figure><img src="../../.gitbook/assets/스크린샷 2025-03-14 오전 10.43.17.png" alt=""><figcaption></figcaption></figure></div>

**클럭 사이클** - 1과 0이 한번씩 이어진것 \
**클럭 엣지** - 1 -> 0 또는 0 -> 1로 변화하는것&#x20;

<div align="left"><figure><img src="../../.gitbook/assets/스크린샷 2025-03-14 오전 10.46.21.png" alt=""><figcaption></figcaption></figure></div>

하나의 명령어를 실행하는데 4 사이클이 걸린다.



### 파이프라이닝

<div align="left"><figure><img src="../../.gitbook/assets/스크린샷 2025-03-14 오전 10.48.06.png" alt=""><figcaption></figcaption></figure></div>

현재 명령 실행을 완료하기 전에 다음 명령을 실행한다.



### <mark style="background-color:yellow;">메모리 계층</mark>

<div align="left"><figure><img src="../../.gitbook/assets/스크린샷 2025-03-14 오전 10.58.38.png" alt=""><figcaption></figcaption></figure></div>

CPU에 가까이 있을수록 비싸고 성능이 좋다.&#x20;



**1차 저장 장치** - CPU 명령이 직접 접근할 수 있다. (레지스터, 캐시, 메인 메모리(RAM))

**정적램 (SRAM)** - 레지스터, 캐시\
작은 전자 회로(래치)에 데이터를 저장, CPU에 장착된다.\
캐시는 레지스터와 메인 메모리의 중간 역할, CPU의 제어 회로가 메인 메모리의 내용 일부를 캐시에 저장한다.&#x20;

**동적램 (DRAM)** - 메인 메모리 \
캐패시터에 데이터를 저장, 메모리 버스를 통해 CPU에 연결 된다.\
저장된 값을 유지하기 위해 캐패시터의 전하를 자주 채워야하기 때문에 '동적'램이다.

<div align="left"><figure><img src="../../.gitbook/assets/스크린샷 2025-03-14 오전 11.15.48.png" alt=""><figcaption></figcaption></figure></div>

CPU와 메인 메모리는 몇 cm 떨어져 있지만 반드시 메모리 버스를 경유해야해서 CPU 저장소에 비하면 전송률이 낮아진다. 그 결과 메모리 버스를 **폰 노이만 병목**이라 부른다.&#x20;



**2차 저장 장치** - CPU 명령이 직접 참조할 수 없다. (HDD, SSD 등)\
SATA 컨트롤러에 연결된 HDD는 I/O 버스를 통해 I/O컨트롤러에 연결되고 이는 다시 메모리 버스로 연결 된다.\
장치를 경유할 때마다 전송이 지연된다.



### <mark style="background-color:yellow;">지역성</mark>

```kts
var sum = 0
val arr = intArrayOf(1,2,3,4)
for (i in 0 .. 3) {
    sum += arr[i]
}
```

**시간적 지역성** - 같은 데이터에 반복적으로 접근하는 경향\
for문이 반복되는동안 sum, i 에 반복 접근한다.\
메인 메모리에서 캐시로 한번만 로드하면 두 번째부터는 고속 캐시에서 접근한다.

**공간적 지역성** - 접근한 데이터 가까이에 있는 데이터를 사용하는 경향\
배열을 읽을때 근접된 데이터를 접근한다.\
첫 번째 인덱스에 접근하면 그 뒤에 정수 몇 개도 함께 캐시에 저장한다.



### <mark style="background-color:yellow;">CPU 캐시</mark>

1. 프로그램은 가장 먼저 데이터의 메모리 주소를 계산
2. 캐시와 메인 메모리에 동시에 주소를 전송
3. 캐시가 메모리 보다 빠르기 때문에 캐시의 응답이 먼저 도착한다.

**캐시 히트** - 캐시가 존재&#x20;

**캐시 미스** - 캐시가 존재하지 않음, 메인 메모리에 대한 요청이 완료되면 데이터를 캐시에 쓰게된다.

* **강제 실패 / 콜드 스타트 실패** - 새로운 메모리 주소에 처음 접근
* **용량 실패** - 프로그램이 캐시보다 더 많은 메모리를 사용
* **충돌 실패** - 같은 캐시 라인을 두고 경쟁&#x20;



### 다이렉트 맵드 캐시

<div align="left"><figure><img src="../../.gitbook/assets/스크린샷 2025-03-14 오후 12.07.34.png" alt=""><figcaption></figcaption></figure></div>

저장소 공간을 **캐시 라인**이라는 단위로 나눈다.

**캐시 데이터 블록** - 데이터를 캐시로 로딩할 때, 항상 블록 크기 만큼의 데이터를 받는다. \
**큰 블록** - 공간적 지역성을 활용하는 성능을 개선\
**많은 블록** - 다양한 메모리의 버스 셋을 저장&#x20;

**메타데이터** - 캐시 라인의 데이터 블록의 내용에 관한 정보를 저장한다.

메모리의 서브셋 식별을 돕는 정보를 저장

<div align="left"><figure><img src="../../.gitbook/assets/스크린샷 2025-03-14 오전 11.48.54.png" alt=""><figcaption></figcaption></figure></div>

같은 캐시 라인에 매핑된 메모리 영역은 같은 캐시 라인의 공간을 두고 경쟁\
**태그, 인덱스, 오프셋**을 통해 데이터 요청&#x20;

### 셋 어소시에이티브 캐시

<div align="left"><figure><img src="../../.gitbook/assets/스크린샷 2025-03-14 오후 12.06.10.png" alt=""><figcaption></figcaption></figure></div>

캐시 라인을을 하나의 셋에 두어 충돌실패를 줄인다.



**Write Through** - 캐시의 값을 수정하고, 동시에 메인 메모리를 업데이트

**Write Back** - 캐시 데이터를 업데이트 한 뒤, 메인 메모리는 나중에 업데이트\
**더티 비트**라는 추가 메타데이터 비트를 저장





