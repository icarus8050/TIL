# Garbage Collector 이해하기

## Garbage Collector

 가비지 컬렉터는 애플리케이션의 할당된 동적 메모리를 자동으로 관리합니다. gc는 다음의 동작들을 통해 동적 메모리 관리를 자동으로 수행합니다.

-   메모리를 운영체제에 할당하고 반환
-   요청 시에 메모리를 애플리케이션에 전달
-   메모리가 애플리케이션에 의해 여전히 사용되고 있는지 결정
-   재사용을 위해 사용되지 않는 메모리를 애플리케이션으로부터 회수

 가비지 컬렉터는 다음의 두 가지 가설을 관점으로 설계되었습니다.

-   대부분의 객체는 금방 접근 불가능 상태(unreachable)가 된다.
-   오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다.

 이러한 가정을 **약한 세대 가설(weak generational hypothesis)**이라 합니다. 이를 바탕으로 매번 메모리 전체를 검사하지 않고 일부만 검사할 수 있도록 generational한 구조로 나뉘어졌습니다.

#### Young generation

-   새롭게 생성된 객체 대부분이 이 영역으로 위치합니다.
-   대부분의 객체가 금방 접근 불가능 상태가 되기 때문에 매우 많은 객체가 Young 영역에 생성되었다가 사라집니다.
-   이 곳이 가득차면 minor gc가 발생합니다.
-   minor gc가 발생하면 살아있는 객체들만 체크하고 나머지는 다 없애버립니다.
-   살아남은 객체들 중 더 오래 쓸 것 같은 것들은 Tenured 영역으로 옮깁니다.

#### Tenured generation

-   Young 영역에서 살아남은 객체들은 이 곳으로 복사됩니다.
-   이 곳이 가득차면 major gc(혹은 full gc)가 발생합니다.
-   대부분 Young 영역보다 크게 할당하며, 크기가 큰 만큼 Young 영역보다 gc는 적게 발생합니다.
-   major gc는 minor gc 보다 오래걸립니다.

## JVM의 힙 메모리 구조

![JVM Heap](./images/JVM_Heap.png)

 Java 8 이전에는 Metaspace 영역이 아닌 Permanent 영역이 존재하였습니다. Permanent 영역은 Class의 Meta 정보나 Method의 Meta 정보, Static 변수와 상수 정보들이 저장되는 공간으로 활용되었습니다. **Java 8 버전부터는 기존의 Permanent 영역이 Native 영역으로 이동하여 Metaspace 영역으로 변경되었습니다.**

#### Eden Space \[Young Generation\]

 처음 생성된 모든 객체는 에덴 영역에 존재합니다. JVM에 의해 정해진 임계치에 도달하면 minor gc가 수행됩니다. minor gc가 수행되면 참조되지 않는 객체들은 에덴 영역에서 제거되고, 살아남은 객체들은 'From' 영역에서 'To' 영역의 Survivor 영역으로 이동합니다. gc가 끝나고나면 'From'과 'To' Survivor 영역의 역할이 서로 바뀝니다. ('From'은 'To'가 되고, 'To'는 'From'이 됩니다.)

#### Survivor 1 (From)

 이 영역은 에덴 영역으로부터 살아남은 객체가 담깁니다. 이전의 gc 과정에서는 'To'의 역할을 하던 영역입니다.

#### Survivor 2 (To)

 이 영역은 gc가 수행될 때, 에덴 영역과 'From'의 역할을 하던 Survivor 영역에서 살아남은 객체들이 담깁니다.

#### Tenured \[Old Generation\]

 Survivor 영역의 객체가 minor gc에서 살아남아 다른 Survivor 영역으로 이동할 때마다 객체의 Age가 증가합니다. 이 Age가 일정 이상이 되면 Tenured 영역으로 이동하게 됩니다(Promotion).

 Promotion의 기준이 되는 Age는 **\-XX:MaxTenuringThreshold** 옵션으로 설정할 수 있습니다. Java SE 8 에서의 default 값은 15이며, 설정 가능한 범위는 0 ~ 15 입니다.

## STW (Stop The World)

 STW란 gc를 실행시키기 위해 JVM이 모든 애플리케이션 쓰레드를 중단시키는 것입니다. STW가 발생하면 gc를 실행하는 쓰레드를 제외한 나머지 쓰레드는 모두 작업을 멈춥니다. gc 작업이 완료된 이후에야 중단했던 작업을 다시 시작합니다. 대게의 경우 gc 튜닝이란 STW 시간을 줄이는 것입니다.

## Mark and Sweep

### Mark

 가비지 컬렉터가 사용중인 객체와 사용되지 않는 객체를 식별합니다.

### Sweep

 Mark 단계에서 식별된 사용되지 않는 객체를 수거합니다.

 전체적인 gc 알고리즘은 다음과 같습니다.

1.  할당 리스트를 순회하면서 마크 비트(mark bit)를 지웁니다.
2.  gc 루트부터 살아 있는 객체를 찾습니다.
3.  이렇게 찾은 객체마다 마크 비트를 세팅합니다.
4.  할당 리스트를 순회하면서 마크 비트가 세팅되지 않은 객체를 찾습니다.
    1.  힙에서 메모리를 회수해 **프리 리스트(free list)**에 되돌립니다. (프리 리스트는 동적 메모리 할당을 위해서 계획적으로 사용된 자료 구조로, 메모리의 할당되지 않은 영역들을 연결 리스트로 연결해서 운용합니다.)
    2.  할당 리스트에서 객체를 삭제합니다.

### Compaction

 살아남은 객체들을 연속된 영역으로 배열합니다. 이 과정은 메모리 단편화(memory fragmentation)를 방지합니다.

## HotSpot의 가비지 수집

 자바 프로세스가 시작되면 JVM은 메모리를 할당(또는 예약)하고 유저 공간에서 연속된 단일 메모리 풀을 관리합니다.

 이 메모리 풀은 각자의 목적에 따라 서로 다른 영역으로 구성되며, 객체는 보통 에덴 영역에 생성됩니다. 수집기가 줄곧 객체를 이동시키기 때문에 객체가 차지한 주소는 대부분 시간이 흐르면서 아주 빈번하게 바뀝니다. 이처럼 객체를 이동시키는 것을 '방출'이라고 하는데, 핫스팟 수집기는 대부분 방출 수집기입니다.

#### bump-the-pointer

 새로 생성되는 대부분의 객체는 에덴 영역에 할당됩니다. bump-the-pointer는 에덴에 할당된 마지막 객체를 추적합니다. 새로 생성되는 객체가 있으면, 해당 객체가 에덴 영역에 할당하기 적당한지 확인합니다. 만약 해당 객체 크기가 에덴 영역에 할당되기 적당하면 에덴 영역에 할당하고 포인터가 마지막 위치를 가르키도록 업데이트 합니다. 따라서, 새로운 객체를 생성할 때마다 마지막에 추가된 객체만 점검하면 되므로 매우 빠르게 메모리 할당이 이루어집니다.

 하지만 멀티 쓰레드 환경에서는 Thread-safe 를 고려해주어야 합니다. 여러 쓰레드에서 사용하는 객체를 에덴에 저장하려면 락(Lock)이 발생할 수 밖에 없고, lock-connection 때문에 성능이 매우 떨어지게 될 것입니다.

HotSpot VM에서 이를 해결하기 위한 것이**TLAB(****Thread-Local Allocation Buffer)**라고 합니다.

#### TLAB (Thread-Local Allocation Buffer)

 JVM은 에덴을 여러 버퍼로 나누어 각 애플리케이션 쓰레드가 새 객체를 할당하는 구역으로 활용하도록 배포합니다. 이렇게 하면 각 쓰레드는 혹여 다른 쓰레드가 자신의 버퍼에 객체를 할당하지는 않을까 염려할 필요가 없습니다.


---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9791162241776&orderClick=LAG&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9791162241776&orderClick=LAG&Kc=)

[https://docs.oracle.com/en/java/javase/12/gctuning/introduction-garbage-collection-tuning.html#GUID-326EB4CF-8C8C-4267-8355-21AB04F0D304](https://docs.oracle.com/en/java/javase/12/gctuning/introduction-garbage-collection-tuning.html#GUID-326EB4CF-8C8C-4267-8355-21AB04F0D304)

[d2.naver.com/helloworld/1329](https://d2.naver.com/helloworld/1329)

[johngrib.github.io/wiki/jvm-memory/#fnref:book2-Heap](https://johngrib.github.io/wiki/jvm-memory/#fnref:book2-Heap)

[johngrib.github.io/wiki/java-gc-eden-to-survivor/#survivor-%EC%97%90%EC%84%9C-old-%EC%98%81%EC%97%AD%EC%9C%BC%EB%A1%9C-promotion](https://johngrib.github.io/wiki/java-gc-eden-to-survivor/#survivor-%EC%97%90%EC%84%9C-old-%EC%98%81%EC%97%AD%EC%9C%BC%EB%A1%9C-promotion)

[dzone.com/articles/java-memory-architecture-model-garbage-collection](https://dzone.com/articles/java-memory-architecture-model-garbage-collection)

[www.infoworld.com/article/2078645/jvm-performance-optimization-part-3-garbage-collection.html](https://www.infoworld.com/article/2078645/jvm-performance-optimization-part-3-garbage-collection.html)
