# GC

### 마크 앤 스위프&#x20;

GC에서 사용되는 가장 기초 알고리즘입니다.

1. 할당 리스트를 순회 하면서 마크 비트를 지운다.
2. GC 루트부터 살아있는 객체를 찾는다. (DFS)
3. 찾은 객체에 마크 비트를 세팅한다.
4. 할당 리스트를 순회하면서 마크 비트가 세팅되지 않은 객체를 찾는다.
   1. 힙에서 메모리를 회수해 프리 리스트에 되돌린다.
   2. 할당 리스트에서 객체를 삭제한다.



### 자바의 객체

자바에서는&#x20;

* 기본형
* 객체 레퍼런스

두가지의 값만 사용합니다.

자바는 C++와 달리 주소를 역참조하는 일반적인 메커니즘이 없고 오직 `오프셋 연산자` 만으로 필드에 엑세스 하거나 객체 레퍼런스의 메서드를 호출할 수 있습니다.

또한 자바는 call by value 방식으로만 메서드를 호출합니다.



### 핫스팟의 가비지 수집

자바는 C/C++ 계열의 환경과 달리 OS를 이용해 동적으로 메모리를 관리하지 않습니다.

대신 프로세스가 시작되면 JVM은 메모리를 할당(예약)하고 유저 공간에서 연속된 단일 메모리 풀을 관리합니다.&#x20;

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

에덴 영역은 대부분의 객체가 탄생하는 영역이고 단명객체는 다른 곳에는 위치할 수 없으므로 특별히 잘 관리 해야하는 영역입니다.

JVM은 에덴을 여러 버퍼로 나누어 각 스레드가 새 객체를 할당하는 구역으로 활용하도록 배포합니다.\
이 구역을 `스레드 로컬 할당 버퍼(TLAB)` 라고 합니다.



### 병렬 수집

병렬 수집기는 세대 전체 콘텐츠를 대상으로 한번에, 가능한 효율적으로  가비지를 수집합니다.

* 영 세대

스레드가 에덴 영역에 객체를 할당 하려는데 자신이 할당받은 TLAB 공간은 부족하고 새 TLAB를 할당할 수 없을 때 영 세대 수집이 발생 합니다.

영 세대 수집이 실행되면 JVM은 전체 어플리케이션을 중단( STW ) 시킵니다.

핫스팟은 에덴과 서바이버 공간을 확인하여 가비지가 아닌 객체를 골라냅니다. 그 이후 세대 카운트를 늘리며 비어있는 서바이버 공간으로 모두 방출 합니다.

이후 어플리케이션 스레드를 재시작합니다.

* 올드 세대

올드 세대에 더 방출할 공간이 없으면 내부에서 재배치하여 공간을 회수합니다.

메모리 단편화를 줄이고 효율적으로 처리 가능합니다.



병렬 수집기는 세대 전체 콘텐츠를 대상으로 한번에, 가능한 효율적으로 가비지를 수집합니다.\
하지만 단점이 있는데 실행될때 마다 풀 STW를 유발합니다.

이는 올드 세대 수집시 STW 시간이 힙 크기에 비례한다는 점입니다.





Garbage Collection이란 무엇인가?

→ 자바는 언어 레벨에서 메모리 해제가 불가능한데 이것을 개발자가 아닌 JVM에서 미사용 메모리를 탐지하고 해제시켜준다.



Java의 메모리 할당과 해제 방식은 어떻게 다른가?

→ 메모리 할당은 new 연산으로 이루어지지만 직접 해제는 불가능함



Java에서 메모리를 해제하는 방법은 무엇인가?

→ GC가 미사용 객체를 추적하여 메모리를 해제 이때 미사용 객체를 탐색하여야하는데\
reference count, reachability등의 방법이 있다.\
현재는 Reachability를 사용하며 레퍼런스 카운트의 경우 순환참조시 메모리 해제를 하지 못하는 상황이 발생한다.



Garbage Collector는 무엇을 기준으로 메모리를 해제하나요?

→ 현재 사용중인 객체 트리(Root Set)를 탐색하여 사용하고있는지 안하는지를 기준으로 해제한다.



Root Set이란 무엇인가요?

→ GC가 일어날때 현재 도달 가능한 객체들을 파악하기 위한 최상위 객체 참조 집합



Garbage Collection의 장점은 무엇인가요?

→ 개발자가 직접 메모리 관리를 안해도 되기때문에 개발 편의성이 올라간다.



Garbage Collection의 단점은 무엇인가요?

→ 메모리를 원하는 시점에 해제할 수 없고 full GC 발생시 stop the world가 발생한다.



Mark-and-Sweep Algorithm이란 무엇인가요?

→ 마킹 후 정리 알고리즘 : 객체의 사용 카운트를 파악한 뒤 카운트가 0인것들을 해제한다.



Mark-and-Compaction Algorithm은 무엇인가요?

→ 객체 정리 과정에서 힙 내부의 단편화가 발생하는데 이것을 피하기 위해 메모리 주소를 이동시키는 알고리즘



Copying Algorithm이란 무엇인가요?

→ 메모리를 2개의 구조로 분리하여 사용하는 객체와 미사용 객체가 함께 있을때 사용하는 객체만 다른 메모리 주소로 copy 후 현재의 메모리 주소를 모두 정리하는 알고리즘



Generational Algorithm이란 무엇인가요?

→ 객체를 세대별로 나누어 관리하는 알고리즘, 새롭게 생성된 객체부터 young → old → permanent(Metaspace)로 승격



Garbage Collection이 발생하는 주된 이유는 무엇인가요?

→ 메모리영역이 가득차게 되면 GC발생



Heap 단편화(Heap Fragmentation)이란 무엇인가요?

→ 여러 객체가 생성될때 메모리 주소는 순차적으로 할당 되지만 해제의 순서는 순차적이지 않기 때문에 메모리 중간중간 빈 공간이 생기게 되고 이러한 현상을 단편화 라고 한다.



Hotspot JVM에서 사용하는 Garbage Collection 방식은 무엇인가요?

→ 여러 알고리즘을 혼합하여 사용한다. 객체의 사용여부를 확인하기 위해 Marking, survival1, 2로 나뉘어져있는 공간에 Copy 각 리전별로 세대를 승격 등등



Reference Counting Algorithm의 문제점은 무엇인가요?

→ 순환참조 발생시 참조 카운트가 0이 되지 않지만 실제로는 참조하지 않을 수 있다.



Garbage Collection이 메모리 누수를 방지하는 방법은 무엇인가요?

→ 참조 그래프를 탐색하여 실제 사용중인지 감지한다.



Mark-and-Sweep Algorithm의 단점은 무엇인가요?

→ STW 시간이 길다. 단편화가 많이 발생한다.



Generational Algorithm의 장점은 무엇인가요?

→ 새롭게 생성된 객체들은 오래 유지되지 못할 확률이 높다라는 기조를 바탕으로 Full GC보다는 Young 세대의 Minor GC가 주로 발생 STW 시간을 최대한 줄인다.



Mark-and-Compaction Algorithm의 단점은 무엇인가요?

→ 메모리 이동을 위한 추가 작업에대한 코스트가 존재한다.



TLAB이란 무엇인가요?

→ 쓰레드별로 다른 주소의 메모리리 버퍼를 할당하는 방법 메모리 획득에 대한 경쟁을 줄여 성능을 최적화 한다.



Promotion이란 무엇인가요?

→ 객체가 메모리의 하위 세대에서 상위 세대로 이동하는 것 객체는 일반적으로 young 영역에서 생성이 되며 객체가 오래 생존시에만 old 영역으로 이동 할 수 있다.



Parallel Compaction Collector는 어떤 유형의 CPU에서 유리한가요?

→ CPU 코어가 많을수록 병렬처리를 많이 할 수 있기때문에 유리하다. + 하이퍼 쓰레딩(논리 코어) 기능 지원 CPU



Parallel Compaction Algorithm의 세 가지 단계는 무엇인가요?

→ Mark-Summary-Compaction



Mark Phase에서는 어떤 작업을 수행하나요?

→ 살아있는 객체들을 탐지하고 마킹합니다.



Summary Phase에서는 어떤 정보를 평가하나요?

→ 메모리의 상태를 확인하여 정보를 수집합니다.



Compaction Phase에서는 어떤 작업을 수행하나요?

→ 살아있는 객체들을 이동시켜 메모리를 정리합니다.



Parallel Compaction Collector는 Young Generation에서 어떤 알고리즘을 사용하나요?

→ Parallel Collector 와 동일한 Parallel Copy Algorithm 을 사용



Compaction Phase에서 사용되는 두 가지 Region 유형은 무엇인가요?

→ Destination Region, Source Region



Dense Prefix는 무엇을 나타내나요?

→ reachable object의 대부분을 차지한다고 판단되는 Region을 구분



Compaction Phase에서 첫 번째 단계는 무엇인가요?

→ 쓰레드들이 Region을 할당받아 압축을 수행



Compaction Phase에서 두 번째 단계는 무엇인가요?

→ Sweep을 수행 Garbage Region을 정리한다.



Mark Phase에서 Region의 크기는 보통 얼마인가요?

→ 1kb



G1 Collector에서 Young Generation과 Old Generation은 어떻게 구분되나요?

→ 물리적으로 연속되어있던 이전 GC들과 달리 메모리에 비순차적이며 논리적으로 구분된다.



G1 Collector의 Remember Set은 무엇을 관리하나요?

→ 리전마다 보유하고 있는 외부에서 힙 영역 내부를 참조하는 레퍼런스를 관리하기 위한 자료구조 입니다.



G1 Collector의 Marking Algorithm은 무엇인가요?

→ concurrent mark (SATB - snapshot at the beginning)



G1 Collector에서 Old Region Reclaim Phase의 두 가지 단계는 무엇인가요?

→ Remark, Evacuation



G1 Collector의 Compaction Phase의 주요 목표는 무엇인가요?

→ Large Chunk로 FreeSpace를 병합하여 메모리 단편화를 방지



Parallel Compaction Algorithm에서 Source Region과 Destination Region은 무엇인가요?

→ live object만 새로운 메모리 Region으로 옮기기 위한 두 리전 (from, to)



G1 Collector의 Region 크기는 보통 얼마인가요?

→ 1Mb \~ 32Mb
