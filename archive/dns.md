# 내부망 DNS 이슈로 인한 응답지연

#### **🚨 문제 발생**

신규 서비스를 배포하기 위해 서버를 새로 할당받아 테스트하는 도중, 5초 단위의 비상적인 응답이 발생하였습니다.

동일한 요청을 반복적으로 보냈을 때, 이상하게도 5초 단위의 응답이 오는 현상이 이해되지 않았습니다.

저는 APM과 로그 등을 통해 원인을 찾아보았습니다.

#### **🔎 원인 분석 및 해결**

가장 먼저 확인한 것은 애플리케이션 로그였습니다.

요청이 들어온 시점과 응답을 반환하기 직전의 시간이 5초 이상 걸린 로그들을 필터링했을 때,

서비스 로직이나 DB의 문제는 아닌 것으로 판단할 수 있었습니다.

두 번째로 확인한 것은 APM 트레이싱 로그였습니다.

Nginx로 들어온 요청부터 애플리케이션 로그까지의 분산 로그를 확인한 결과, 5초 이상 소요된 로그들을 필터링할 수 있었습니다.

이 두 가지 결과를 토대로 의심이 가는 서블릿 코드를 디버깅해 본 결과, CustomLoggingFilter에서 사용된

`InetAddress.*getLocalHost*()` 내부에서 5초의 지연이 발생하는 원인을 발견하였습니다.

해당 `InetAddress.*getLocalHost*()` 코드는 아래와 같습니다.

```java
public static InetAddress getLocalHost() throws UnknownHostException {

    @SuppressWarnings("removal")
    SecurityManager security = System.getSecurityManager();
    try {
        CachedLocalHost clh = cachedLocalHost;
        if (clh != null && (clh.expiryTime - System.nanoTime()) >= 0L) {
            if (security != null) {
                security.checkConnect(clh.host, -1);
            }
            return clh.addr;
        }
        String local = impl.getLocalHostName();

        if (security != null) {
            security.checkConnect(local, -1);
        }

        InetAddress localAddr;
        if (local.equals("localhost")) {
            localAddr = impl.loopbackAddress();
        } else {
            try {
                localAddr = getAllByName0(local, false, false)[0];
            } catch (UnknownHostException uhe) {
                UnknownHostException uhe2 =
                    new UnknownHostException(local + ": " + uhe.getMessage());
                uhe2.initCause(uhe);
                throw uhe2;
            }
        }
        cachedLocalHost = new CachedLocalHost(local, localAddr);
        return localAddr;
    } catch (java.lang.SecurityException e) {
        return impl.loopbackAddress();
    }
}
```

해당 코드의 동작을 정리해보면 다음과 같습니다.

1. **캐시에 유효한 데이터가 있으면 즉시 반환합니다.**
2. `impl.getLocalHostName()` **을 호출하여 호스트 이름을 확인합니다.**

이때 \*\*[impl.getLocalHostName()](https://github.com/openjdk/jdk17/blob/4afbcaf55383ec2f5da53282a1547bac3d099e9d/src/java.base/unix/native/libnet/Inet4AddressImpl.c#L60-L71)\*\*은 네이티브 메서드로 작성된 코드로

이는 내부적으로 C 라이브러리(glibc) 함수인 [https://man7.org/linux/man-pages/man2/gethostname.2.html](https://man7.org/linux/man-pages/man2/gethostname.2.html) 를 호출하게 되고

리눅스 시스템콜인 [https://man7.org/linux/man-pages/man2/uname.2.html](https://man7.org/linux/man-pages/man2/uname.2.html) 을 통해 커널에서 호스트 정보를 복사해오게 됩니다.

1. **호스트 이름이** `localhost`**이면 바로 루프백 주소를 반환하고 아니라면** `getAllByName0()`**을 호출하여 IP 주소를 가져옵니다.**

```java
==== private static InetAddress[] getAllByName0(
		String host, boolean check, boolean useCache) ====

a.		if (addrs == null) {
		    Addresses oldAddrs = cache.putIfAbsent(
		        host, addrs = new NameServiceAddresses(host)
		    );
		    if (oldAddrs != null) {
		        addrs = oldAddrs;
		    }
		}
		return addrs.get().clone(); 

==== public InetAddress[] get() ====
		
b.  		lookupLock.lock();
		try {

c.		try {
		    inetAddresses = getAddressesFromNameService(host); 
		    ex = null;
		    cachePolicy = InetAddressCachePolicy.get();
		} catch (UnknownHostException uhe) {
		    inetAddresses = null;
		    ex = uhe;
		    cachePolicy = InetAddressCachePolicy.getNegative();
		}

d.		if (cache.replace(host, this, cachedLookup) &&
		    cachePolicy != InetAddressCachePolicy.FOREVER) {
		    expirySet.add(cachedLookup);
		}
		return inetAddresses;

e.		} finally {
		    lookupLock.unlock();
		}
		return addresses.get();	
```

1. `addrs`에 `NameServiceAddresses`를 저장합니다. 이미 값이 있다면 해당 값을 사용합니다.
2. `addrs.get()` 을 통해 내부적으로 DNS를 질의 하고 이 과정에서 락을 걸게 됩니다.
3. `getAddressesFromNameService(host)` 는 네이티브 메서드인 [**lookupAllHostAddr(String hostname)**](https://github.com/openjdk/jdk17/blob/4afbcaf55383ec2f5da53282a1547bac3d099e9d/src/java.base/unix/native/libnet/Inet4AddressImpl.c#L86-L197) 를 통해 DNS 질의를 진행합니다.
4. 조회 결과를 캐시에 저장합니다.
5. 락을 해제하고 주소를 리턴합니다.
6. **조회한 IP 주소를** `cachedLocalHost`**에 저장합니다.**

이를 확인한 후 "**혹시나 락을 획득한 뒤 DNS 질의 과정에 문제가 있는 것일까?**"라는 의문이 생겼고,

확인을 위해 DNS 질의가 진행되는 순서를 따라가 보았습니다.

DNS 질의는 다음과 같은 순서로 이루어집니다.

1. `/etc/hosts` 파일에서 해당 IP 주소가 등록되어 있는지 확인합니다.
2. 등록된 정보가 없다면 `/etc/resolv.conf`에 설정된 로컬 DNS 서버에 질의합니다.
3. 로컬 DNS 서버에서도 응답하지 않을 경우, 외부 DNS 서버로 질의합니다.

이 과정을 단계별로 확인한 결과, 로컬 DNS 서버에 질의하는 과정(2번)에서 문제의 원인을 찾을 수 있었습니다.

실제로 `/etc/resolv.conf` 내부에는 8.8.8.8, 즉 구글 DNS 주소가 등록되어 있었고,

이슈가 발생한 서버는 내부망에 위치해 있었기 때문에 DNS 질의 후 5초가 지나 타임아웃이 발생한 것이었습니다.

이 시간은 `/usr/include/resolv.h` 를 확인해보면 디폴트 타임아웃이 5초인 것을 확인 할 수 있습니다.

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

전체 흐름을 정리해보면 아래와 같습니다.

1. 서블릿 필터에 작성된 `InetAddress.getLocalHost()` 메서드를 통해 해당 서버의 IP를 요청합니다.
2. 해당 코드는 내부적으로 DNS 질의를 수행합니다.
   1. `/etc/hosts` 파일을 확인합니다.
   2. `/etc/resolv.conf`(로컬 DNS)를 확인한 후 질의합니다. (내부망에서 외부로 질의하여 5초 타임아웃 발생)
   3. 외부 DNS 서버로 질의한 후 정상 응답을 받습니다.
3. 받아온 IP를 캐싱한 후 반환합니다.
4. 로깅 후 최종적으로 응답을 반환합니다.

즉, 캐시가 만료될 때마다 로컬 DNS 질의에서 타임아웃 시간인 5초가 지연 되었다는 것을 파악할 수 있었고,

해당 부분을 수정한 후 정상적으로 응답이 오는 것으로 해당 이슈를 해결할 수 있었습니다.
