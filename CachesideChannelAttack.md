# Analysis of Cache Side-Channel Attack and Countermeasures

## Abstract

클라우드환경에서 캐시 공유는 매우 흔하게 일어나기 때문에 이러한 공격은 매우 위협적이다.<br>

 왜냐하면 클라우드에서 제공하는 서비스 사용자라면 누구든지 피해자가 될수 있기 때문이다.<br>

이러한 공격에 더불어 방어기법들도 진화되어 왔다.<br>

하지만 이러한 대항책은 얼마나 효과적일까?<br>

`Side-Channel-Attack`은 희생자의 행동과 코드 실행에 직접적으로 영향을 받는 희생자의 하드웨어를 관찰하여 개인 정보를 복구하는 기법이다.<br>

이중에 `Cache-Side-Channel Attack`은 cache를 공격벡터로 사용하는 공격이다.<br>

이 공격은 매우 강력하며, 대부분 희생자의 개인키를 얻는것에 초점을 둔다.<br>

이 공격이 성공하면 개인정보, 인증,기밀성은 더이상 보장되지않는다.<br>

대응책은 software, hardware, software-hardware 상호작용하는 레벨에 적용되었다.<br>

software대응책은 이 공격 기법이 희생자의 실행에 따라 타이밍과 데이트 엑세스의 차이점을 분석하면서 개인키를 찾아낸다는 이점을 이용하였다.<br>

따라서 software 방어는 `constant-time algorithm`을 향상 시켰다.<br>

이것은 execution time과 data access bit를 개인키를 구성하는 비트와 독립시켰다. <br>

* 대칭키/비대칭키 조사

그림 2.3 2.4

----
## Side-Channel-Attack

일반적인 도청 공격은 네트워크를 통한 트래픽을 보지만, 이 공격은 그외에도 물리적인 요소나 현상을 관찰할 수 있다.<br>

* software 기반 공격 (Cache side channel attack)
* Hardware 기반 공격 (전력소비같은 물리적 요소 기반)

----
## cache

L1,L2,L3 Cache<br>
L3 Cache = LLC (Last Level Cache)<br>

* [1]Cache hit : 캐시에 필요한 정보가 이미 있을때
* [2]Cache miss :Cache에 정보가 저장되지 않았을때

* spatial locality
* temporal locality

L1 Cache는 L1-I(Instruction) L1-D(Data)로 나누어져 있다.<br>

하지만 L2,LLC는 instruction 과 Data가 나눠져있지 않고 같은장소에 저장된다.<br>

Intel에서는 캐시들이 `[3]Inclusiveness`의 특징이 있다.<br>
LLC에는 L2,L1에 저장되어있는 블록 set들이 포함되어있다.<br>

----
## Cache-Side-Channel Attack

* time-based Attack : 사용자의 명령에 따라 cache hit cache miss 가 일어나는 횟수와 연관된 시간을 기반으로 한다.<br>
* access-based attack : 사용자가 `private key`를 사용하는 시점의 `[1]cache hit, [2]cache miss` 체크한다.<br>

LLC는 `Cross-core` , `[3]Inclusiveness`, `High resolution`의 특성 때문에 사용한다.<br>

Flush+Reload : cache를 초기화 시켜놓고 희생자가 액세스를 하면 속도가 빨라지는 점을 사용하여 메모리 위치를 파악해서 재접속<br>

- 장점 : Low noise / 다른 CPU까지 적용 가능 / non-inclusive cache에서도 동작 가능
- 단점 : 요구사항이 존재 (deduplication/flush) / 정적 데이터만 복구가능

Evict+Reload : (flush가 불가할때 사용)Cache를 기존의 offset보다 큰 `hugh page`를 사용하면서 다른 부분을 차지한다.<br>

- 장점 : flush 없이도 공격가능
- 단점 : 정적메모리만 타겟팅 가능 / LLC slice 해결 문제 / inclusiveness cache 에서만 동작 / 같은 CPU에서만 가능

Prime+Probe : (`shared memory page`가 없을때 사용) cache를 미리 채워둔 상태로 공격<br>

- 장점 : `shared memory` 불필요 / 정적 동적 메모리 둘다 타겟팅 가능
- 단점 : `Flush+Reload`보다 noise 심함 / LLC slice 문제 / inclusive cache에서만 동작 / 같은 CPU에서만 동작 / target set를 알아야함


* 실제 예시 좀더

----

## Contermeasures

* Cache leakage Free code
<br>

* Page Coloring : User별로 `물리주소`를 비트를 00 01 10 11 같은 식으로 다르게 준다.<br>
<br>

* Cache Allocation Technology : Intel CAT는 cache를 lock하는 하드웨어 프레임워크를 제공한다.<br>
* `un-evictable`하기 위해 cache를 마크한다.<br>
* 따라서 공격자는 희생자의 `cache accesse`에 영향을 줄수 없다.<br>
<br>

* Behavior Detection
* Hardware Performance Conters 는 `hardware event`를 track할수있다.<br>
* `LLC Attack`은 `cache misses/hits`면에서 자취를 남긴다.<br>
* `Hypervisor/OS`는 이 비정상적인 행위를 추적한다.<br>
* 탐지는 `memory access`를 점검하면 향상시킬수있다.<br>
