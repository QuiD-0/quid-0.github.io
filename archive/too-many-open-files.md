# Too Many Open Files 에러와 원인을 찾는 과정

### **🚨 문제 발생**

개선 사항을 적용한 서비스를 배포한 뒤 초기 모니터링에서는 별다른 문제가 없었지만,

약 4시간이 지나자 서비스가 다운되는 이슈가 발생했습니다.

로그를 확인해 보니 다음과 같은 오류가 출력 되었고

```
java.lang.OutOfMemoryError: unable to create new native thread  
...  
Caused by: java.net.SocketException: **Too many open files**  
```

문제가 없던 버전으로 롤백 후 재가동 시키고 원인 분석에 나섰습니다.

### **🔎 원인 분석 및 해결**

서비스가 정상적으로 동작하다가 4시간이 지나서 갑자기 다운된 이유는 무엇일까요?

로그를 분석하며 원인을 추적해 보니, **리눅스의 파일 디스크립터(File Descriptor) 제한**과 관련된 문제라는 것을 발견했습니다.

1. **리눅스에서는 모든 것이 파일이다.**

리눅스 시스템에서는 단순한 텍스트 파일뿐만 아니라, **소켓, 디바이스, 파이프** 같은 것도 파일처럼 관리됩니다.

즉, 애플리케이션이 네트워크 통신을 할 때도 **소켓을 하나의 파일처럼 열어서 데이터를 주고받고**,

작업이 끝나면 이를 닫아야 합니다.

1. **파일을 한 번에 열 수 있는 개수에는 한계가 있다.**

운영체제(OS)에는 **한 프로세스가 동시에 열 수 있는 파일 개수 제한**이 설정되어 있습니다.

이를 확인하려면 다음 명령어를 사용할 수 있습니다.

```bash
ulimit -a -S  # 소프트 제한 (프로세스가 생성될 때 적용되는 값)
ulimit -a -H  # 하드 제한 (프로세스에서 최대로 늘릴 수 있는 값)
```

1. **어플리케이션이 너무 많은 소켓을 열면 문제가 발생한다.**

이번 경우처럼 애플리케이션이 예상보다 많은 네트워크 통신을 처리하면,

OS에서 허용하는 파일 개수보다 많은 소켓이 열리게 되고, 결국 **Too Many Open Files** 오류가 발생하게 됩니다.

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption><p>총 1024개의 파일을 열수있는 상태</p></figcaption></figure>

해당 이슈만 보면 해결 방법은 아주 간단합니다.

OS단에 설정되어있는 값을 프로세스가 사용하는 소켓의 연결개수보다 큰 값으로 바꿔주면 **Too Many Open Files** 에러는 사라질것입니다.

하지만 조금 더 생각을 해보았을때 “**왜 평소에는 괜찮았는데 배포후에 문제가 발생한걸까?**” 라는 생각이 들었고 코드에 어떤 변화가 있었는지 확인해 보았습니다.

변경 내역은 아래와 같았습니다.

> 알림 메시지를 전송하는 로직에서 결과 응답을 기다렸던 코드를 비동기로 개선하는 과정에서 \
> RestTemplate -> WebClient로 변경하여 비동기 처리 적용

우선 비동기로 동시에 너무 많은 요청을 해서 소켓이 과도하게 많이 열렸나? 라는 생각이 들어

이것을 확인해보기 위해 간단한 테스트를 진행해보았습니다.

요청이 들어왔을때 RestTemplate, WebClient로 각각 요청을 보내는 테스트 코드를 만들었고

소켓이 얼마나 열리는지 모니터링 하기 위해 아래의 명령어를 통해 열리는 소켓 파일을 확인해보았습니다.

```
jps                   // 현재 실행중인 java process의 pid를 출력
ls /proc/{pid}/fd     // 해당 프로세스가 사용중인 fileDesciptor 목록
```

우선 1000개의 요청을 보냈을때 성공요청이 정상적으로 왔고 다음으로 fileDesciptor를 확인해보았습니다. 하지만 이상하게 WebClient는 응답을 모두 받아 정상적으로 종료되었지만 해당 소켓은 닫히지 않고 여전히 열려있다는것을 알게 되었고 어딘가에서 leak이 발생한다는 것을 의미하였습니다.

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

저는 원인을 찾던 도중 Reactor-netty 레포에서 해당 이슈를 발견하게 되었습니다.

&#x20;[https://github.com/reactor/reactor-netty/issues/1152](https://github.com/reactor/reactor-netty/issues/1152)

해당 이슈를 정리하면 다음과 같습니다.

1. Reactor-Netty 0.9.8 버전에서 **file descriptor leak** 발생
2. 채널 클래스 정보를 가져올 때 \*\*`newSocketStream()`\*\*이 호출 되면서 리소스가 해제되지 않음
3. 해당 문제가 0.9.9 버전에서 패치

WebClient는 Reactor-Netty 기반의 Spring WebFlux HTTP 클라이언트이므로, 동일한 문제가 발생할 가능성이 있다고 판단하였고, 즉시 서비스에서 사용 중인 Reactor-Netty의 버전을 확인하였습니다.

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

결과적으로, 문제가 발생한 0.9.8 버전을 사용하고 있었으며, 라이브러리 버전 업그레이드를 통해 예상보다 쉽게 이슈를 해결할 수 있었습니다.

버전 업그레이드 후 동일한 테스트를 해보았을때 정상적으로 소켓이 Close되는것을 확인 할 수 있었습니다.
