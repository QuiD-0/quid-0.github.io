# Class Loader

Java의 Dynamic Loading이란 무엇인가요?

→ 런타임 시점에 실행되는 클래스나 리소스를 동적으로 로딩하여 사용하는 것



Load Time Dynamic Loading과 Runtime Dynamic Loading의 차이점은 무엇인가요?

→ 데이터가 로딩되는 시점이 프로그램이 로드 될때 로딩 되는것과 실행시점에 로드되는것이 다르다.



ClassLoader의 역할은 무엇인가요?

→ `ClassLoader`는 클래스 파일(`.class`)을 특정 경로에서 찾아 읽어들이고, 이를 JVM의 메모리에 적재합니다.



JVM에서 Class를 구별하는 방법은 무엇인가요?

→`클래스의 이름`과 `클래스를 로드한 ClassLoader` 조합을 이용



Namespace란 무엇이며, ClassLoader와 어떤 관련이 있나요?

→**ClassLoader**는 Java에서 클래스를 로드하고, 각 클래스에 대해 고유한 네임스페이스를 제공하여, 동일한 이름을 가진 클래스가 충돌하지 않도록 관리



ClassLoader Delegation Model이란 무엇인가요?

→ 클래스를 로드할 때 우선적으로 부모 `ClassLoader`에 요청하고, 부모가 로드하지 못할 경우에만 자식 `ClassLoader`가 클래스를 로드



Bootstrap ClassLoader의 역할은 무엇인가요?

→ 최상위 `ClassLoader`로, JVM의 핵심 클래스들 (`rt.jar`에 있는 표준 라이브러리)을 로드합니다.



Extension ClassLoader는 어떤 역할을 하나요?

→ Bootstrap ClassLoader의 하위에 있으며, JRE의 확장 디렉토리에 있는 클래스를 로드합니다.(`lib/ext` 디렉토리)



Application ClassLoader는 어떤 역할을 하나요?

→ Extension ClassLoader의 하위에 있으며, 애플리케이션 클래스패스(`classpath`)에 있는 클래스를 로드합니다.



User Defined ClassLoader는 어떤 경우에 사용되나요?

→ 동적 클래스 로딩, 클래스 경로 커스터마이징, 클래스 버전 관리, 애플리케이션 격리, 클래스 변조, 보안 제어 등



Class Sharing이란 무엇이며, 어떤 장점을 가지고 있나요?

→ JVM이 동일한 클래스를 여러 인스턴스에서 공유하여 메모리 사용을 최적화, 애플리케이션 시작 시간을 단축시키며, 전체 시스템 성능을 개선할 수 있다.



Verification 과정에서 검증되는 항목에는 어떤 것들이 있나요?

→

* **파일 형식 검증 (File Format Verification)**:
  * 클래스 파일의 전체적인 형식이 올바른지 검사합니다.
* **구조적 검증 (Static Verification)**:
  * 클래스 파일 내부 구조의 일관성을 검증합니다.
* **타입 체크 및 제어 흐름 검증 (Control Flow and Type Verification)**:
  * 바이트코드의 타입 일관성과 제어 흐름의 유효성을 확인합니다.
* **동적 검증 (Runtime Verification)**:
  * 클래스가 실제로 실행되는 동안 발생할 수 있는 오류를 방지하기 위해 마지막으로 동적으로 검증을 수행합니다.



Preparation 과정에서 수행되는 작업은 무엇인가요?

→

* **정적 필드 메모리 할당**:
  * 클래스나 인터페이스의 정적 필드(static field)를 위한 메모리를 JVM이 할당합니다. 이 메모리는 JVM의 메모리 영역 중에서 Method Area에 위치합니다.
* **기본값으로 초기화**:
  * 모든 정적 필드는 해당 타입에 맞는 기본값으로 초기화됩니다.
    * 숫자 타입 (`int`, `long`, `float`, `double`): 0 또는 0.0
    * 참조 타입 (`Object`, 배열 등): `null`
    * `boolean`: `false`
    * `char`: `\\u0000`
* **final static 필드의 초기화**:
  * `final static` 필드의 경우, 이 Preparation 단계에서는 여전히 기본값으로 초기화됩니다. 하지만 이들 필드는 Initialization 단계에서 명시적으로 설정된 값으로 다시 초기화됩니다.
* ### **상수 풀 참조 준비**:



Resolution 과정이란 무엇인가요?

→ JVM이 클래스 파일 내의 심볼릭 레퍼런스를 런타임에 사용할 수 있는 직접 참조로 변환하는 작업입니다. 이 과정은 클래스, 필드, 메서드 등의 심볼릭 레퍼런스를 메모리 상의 실제 엔티티로 연결



Initialization 과정은 언제 수행되며, 어떤 작업을 포함하나요?

→**Initialization** 과정은 클래스 로딩의 마지막 단계로, 클래스 또는 인터페이스의 정적 필드들을 실제 값으로 초기화하고, 정적 초기화 블록(static initializer)을 실행하는 작업을 포함합니다. 이 과정은 클래스가 처음 사용될 때 수행되며, 클래스 로딩 과정의 다른 단계들(로딩, 링크, 검증, 준비, 해석)이 완료된 후에 발생합니다.



ClassLoader의 Eager 방식과 Lazy 방식의 차이점은 무엇인가요?

→

* **Eager 방식**: 모든 클래스를 프로그램 시작 시 로드하여, 클래스 접근 속도를 높이지만 메모리 사용량이 늘어납니다.
* **Lazy 방식**: 클래스가 실제로 필요할 때만 로드하여, 메모리를 절약하지만 첫 접근 시 지연이 있을 수 있습니다.
*



Loading 과정에서 수행되는 세 가지 기본 동작은 무엇인가요?

→

* **클래스 이름을 기반으로 바이너리 데이터를 찾아 로드**: 클래스 파일을 찾아 메모리로 로드합니다.
* **Class 객체 생성**: 로드된 데이터를 기반으로 `Class` 객체를 생성합니다.
* **클래스의 부모 클래스 확인 및 로딩**: 상속 관계에 따라 부모 클래스와 인터페이스를 확인하고 로드합니다.



JVM 스펙에 의해 엄격하게 정의된 Initialization의 수행 시점은 언제인가요?

→ 클래스가 처음 "사용"될 때 수행됩니다. 이 사용 시점에는 정적 필드 접근, 정적 메서드 호출, 인스턴스 생성, 리플렉션 사용 등이 포함



Class Loader Work의 세 가지 과정은 무엇인가요?

→

* **Loading (로딩)**:
  * 클래스 파일을 찾아서 메모리에 읽어들이는 과정입니다.
* **Linking (링크)**:
  * **Verification (검증)**: 바이트코드의 유효성을 검사합니다.
  * **Preparation (준비)**: 정적 필드를 기본값으로 초기화합니다.
  * **Resolution (해석)**: 심볼릭 레퍼런스를 실제 참조로 변환합니다.
* **Initialization (초기화)**:
  * 정적 필드를 실제 값으로 초기화하고, 정적 초기화 블록을 실행합니다.
