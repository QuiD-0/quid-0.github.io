# JVM

### JIT 컴파일

> Just In Time 컴파일

핫스팟은 인터프리티드 모드로 실행하는 동안 어플리케이션을 모니터링하면서 가장 자주 실행되는 코드를 발견해 JIT 컴파일을 수행합니다.

이때 임계점을 넘어가면 프로파일러가 특정 코드를 컴파일/최적화 하게됩니다.

컴파일러가 해석 단계에서 수집한 추적 정보를 근거로 최적화를 결정한다는 장점이 있습니다.



### JVM 메모리 관리

C, C++, C#과 같은 언어들은 개발자가 직접 메모리를 할당/해제 작업을 수행합니다.\
직접 메모리를 관리하게되면 더욱 성능을 끌어올릴수 있지만 메모리를 정확하게 계산해야하는 책임이 따라오게 됩니다.

자바는 GC(Garbage Collection)이라는 프로세스를 이용해 힙 메모리를 자동으로 관리하는 방식으로 해결합니다.

하지만 GC가 실행되면 그동안 어플리케이션이 모두 중단되고 하던 일은 멈춰야 합니다. 이 중단 시간은 아주 짧지만, 어플리케이션 부하가 늘수록 이 시간도 무시할 수 없습니다.



### JVM 모니터링

* JMX

JVM과 그 위에서 동작하는 어플리케이션을 제어하고 모니터링하는 강력한 범용툴

메서드를 호출하고 매개변수를 바꿀 수 있습니다.

* 자바 에이전트

java.lang.instrument 인터페이스로 메서드 바이트코드를 조작합니다.

* JVM 툴 인터페이스

자바 에이전트로 가져올수 없는 정보를 가져오는데 사용된다. 저수준 코드이기 때문에 악영향을 주기 쉽습니다.

* 서비서빌리티 에이전트

자바 객체, 핫스팟 자료 구조 모두 표출 가능한 API와 툴을 모아둔것 입니다.



### GC

Garbage Collection이란 무엇인가?

→ 자바는 메모리 해제가 불가능한데 JVM단에서 사용하지 않는 객체를 정리하기 위한 기술



Java의 메모리 할당과 해제 방식은 어떻게 다른가?

→ 메모리 할당은 new 연산으로 이루어지지만 직접 해제는 불가능함



Java에서 메모리를 해제하는 방법은 무엇인가?

→ GC가 미사용 객체를 추적하여 메모리를 해제



Garbage Collector는 무엇을 기준으로 메모리를 해제하나요?

→ 현재 사용중인 객체 트리를 탐색하여 사용하고있는지 안하는지를 기준으로 해제한다.



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

