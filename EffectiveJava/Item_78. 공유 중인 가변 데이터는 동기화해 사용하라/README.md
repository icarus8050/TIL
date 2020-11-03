# Item 78. 공유 중인 가변 데이터는 동기화해 사용하라

  **synchronized 키워드는 해당 메서드나 블록을 한 번에 한 스레드씩 수행하도록 동기화를 보장합니다.**

 동기화는 일관된 상태를 가진 객체에 접근하는 메서드가 그 객체에 락(lock)을 걸도록 하고, 락을 건 메서드는 객체의 상태를 확인하고 필요하면 수정하여 객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시는 것입니다. 즉, 동기화를 이용하면 다른 스레드가 객체의 일관성이 깨진 상태를 볼 수 없게합니다.

**적절히 동기화한 예제 코드**

```
import java.util.concurrent.TimeUnit;

public class StopThread {
    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested())
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}

```

 위 코드는 쓰게 메서드(requestStop)와 읽기 메서드(stopRequested) 모두를 동기화 했는데, 쓰기와 읽기 모두 동기화해야 동작을 보장할 수 있습니다.

 동기화에 대한 비용을 줄이려면 stopRequested 필드를 volatile으로 선언하여 동기화를 생략해도 됩니다.

**volatile 필드를 사용한 예제 코드**

```
import java.util.concurrent.TimeUnit;

public class StopThread {
    private static volatile boolean stopRequested;

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}

```

#### volatile 주의사항

```
    private static volatile int nextSerialNumber = 0;
    
    public static int generateSerialNumber() {
        return nextSerialNumber++;
    }
```

 위 메서드는 매번 고유한 값을 반환할 의도로 만들어졌습니다. 겉으로 보기엔 굳이 동기화하지 않더라도 불변식을 보호할 수 있어보이지만 동기화 없이는 올바르게 동작하지 않습니다.

  문제의 원인은 증가 연산자(++)입니다. 코드상으로는 하나지만 실제로는 nextSerialNumber 필드에 두 번 접근합니다. 먼저 값을 읽고, 그런 다음 (1 증가한) 새로운 값을 저장하는 것입니다. 만약 두 번째 스레드가 이 두 접근 사이를 비집고 들어와 값을 읽어가면 첫 번째 스레드와 똑같은 값을 돌려받게 됩니다.

 generateSerialNumber 메서드에 synchronized 한정자를 붙이면 이 문제가 해결됩니다. 동시에 호출해도 서로 간섭받지 않기 때문입니다. synchronized를 붙였다면 nextSerialNumber 필드에서는 volatile을 제거해야 합니다.

#### java.util.concurrent.atomic 패키지

 java.util.concurrent.atomic 패키지의 AtomicLong을 사용해도 좋습니다. 이 패키지에는 락 없이도(lock-free) 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨 있습니다. volatile은 동기화의 두 효과 중 통신 쪽만 지원하지만 이 패키지는 원자성(배타적 실행)까지 지원합니다.

**java.util.concurrent.atomic을 이용한 lock-free 동기화**

```
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
```

 이번 아이템에서 언급된 문제들을 피하는 가장 좋은 방법은 애초에 가변 데이터를 공유하지 않는 것입니다. 불변 데이터만 공유하거나 아무것도 공유하지 않는 것이 좋습니다.

** 가변 데이터는 단일 스레드에서만 쓰는 것이 좋습니다.**

한 스레드가 데이터를 다 수정한 후 다른 스레드에 공유할 때는 해당 객체에서 공유하는 부분만 동기화해도 됩니다. 그러면 그 객체를 다시 수정할 일이 생기기 전까지 다른 스레드들은 동기화 없이 자유롭게 읽어갈 수 있습니다.

#### 객체를 안전하게 발행하는 방법

 클래스 초기화 과정에서 객체를 정적 필드, volatile 필드, final 필드, 혹은 보통의 락을 통해 접근하는 필드에 저장해도 됩니다. 동시성 컬렉션에 저장하는 방법도 있습니다.

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
