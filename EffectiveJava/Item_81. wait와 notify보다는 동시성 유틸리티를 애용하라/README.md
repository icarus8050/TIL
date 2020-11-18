# Item 81. wait와 notify보다는 동시성 유틸리티를 애용하라

## 동시성 컬렉션 (concurrent collection)

 동시성 컬렉션은 List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 고려하여 구현한 고성능 컬렉션입니다. 높은 동시성에 도달하기 위해 동기화를 각자의 내부에서 수행합니다. 따라서 **동시성 컬렉션에서 동시성을 무력화하는 건 불가능하며, 외부에서 락을 추가로 사용하면 오히려 속도가 느려집니다.**

 동시성 컬렉션에서 동시성을 무력화하지 못하므로 여러 메서드를 원자적으로 묶어 호출하는 일은 불가능합니다. 그래서 여러 기본 동작을 하나의 원자적 동작으로 묶는 '상태 의존적 수정' 메서드들이 추가되었습니다.

 예를 들어 Map의 putIfAbsent(key, value) 메서드는 주어진 키에 매핑된 값이 아직 없을 때만 새 값을 넣고 null을 반환하고, 기존 값이 있었다면 그 값을 반환합니다.

**ConcurrentMap으로 구현한 동시성 정규화 맵**

```
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;

public class ConcurrentMapExam {
    private static final ConcurrentMap<String, String> map =
            new ConcurrentHashMap<>();

    public static String intern(String s) {
        String result = map.get(s);
        if (result == null) {
            result = map.putIfAbsent(s, s);
            if (result != null)
                result = s;
        }
        return result;
    }
}

```

 위 예제는 ConcurrentHashMap은 get 같은 검색 기능에 최적화되어 있으므로 get을 먼저 호출하여 필요할 때만 putIfAbsent를 호출하도록 구현된 코드입니다.

** Collections.synchronizedMap 보다는 ConcurrentHashMap을 사용하는게 성능적으로 훨씬 좋습니다.**

## 동기화 장치 (synchronizer)

 동기화 장치는 스레드가 다른 스레드를 기다릭 수 있게 하여, 서로 작업을 조율할 수 있게 해줍니다. 가장 자주 쓰이는 동기화 장치는 CountDownLatch와 Semaphore입니다. CyclicBarrier와 Exchanger는 그보다 덜 쓰입니다. 가장 강력한 동기화 장치는 Phaser입니다.

 CountDownLatch는 일회성 장벽으로, 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 합니다. CountDownLatch의 생성자에서는 int 값을 받으며, 이 값은 countDown() 메서드를 몇 번 호출해야 대기 중인 스레드를 깨우는지 결정합니다.

**CountDownLatch 예제 코드**

```
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.Executor;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class CountDownLatchExam {
    public static long time(Executor executor, int concurrency,
                            Runnable action) throws InterruptedException {
        CountDownLatch ready = new CountDownLatch(concurrency);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done = new CountDownLatch(concurrency);

        for (int i = 0; i < concurrency; i++) {
            executor.execute(() -> {
                // 타이머에게 준비를 마쳤음을 알린다.
                ready.countDown();
                try {
                    // 모든 작업자 스레드가 준비될 때까지 기다린다.
                    start.await();
                    System.out.println("Start at " + Thread.currentThread().getName());
                    action.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    done.countDown();
                }
            });
        }

        ready.await();  // 모든 작업자가 준비될 때까지 기다린다.
        System.out.println("Ready!");
        long startNanos = System.nanoTime();
        start.countDown();  // 작업자들을 깨운다.
        done.await();   // 모든 작업자가 일을 끝마치기를 기다린다.
        System.out.println("Done!");
        return System.nanoTime() - startNanos;
    }

    public static void main(String[] args) throws Exception {
        try (BufferedReader br = new BufferedReader(new InputStreamReader(System.in))) {
            ExecutorService executorService = Executors.newFixedThreadPool(10);

            while (true) {
                String str = br.readLine();
                if (str.equals("exit")) break;

                Runnable action = () -> System.out.println(Thread.currentThread().getName() + " : " + str);
                System.out.println(time(executorService, 10, action));
            }
        }
    }
}

```

 ready 래치는 작업자 스레드들이 준비가 완료됐음을 타이머 스레드에 통지할 때 사용합니다. 통지를 끝낸 작업자 스레드들은 start 래치가 열리기를 기다리며, 마지막 작업자 스레드가 ready.countDown() 메서드를 호출하면 잠들어 있던 모든 작업자 스레드 깨어나서 start.countDown() 메서드가 호출됩니다. 이후에는 action.run()이 호출되어 기능이 수행되고, 모든 작업자 스레드가 작업을 끝낼 때까지 done 래치에 의해 대기하게 됩니다. doen 래치가 열리면 마지막 남은 로직들이 수행됩니다.

 time 메서드에 넘겨진 **실행자(executor)는 concurrent 매개변수로 지정한 동시성 수준만큼의 스레드를 생성할 수 있어야 합니다.** 그렇지 못하면 이 메서드는 끝나지 못하게 되고, 이러한 상태를 스레드 기아 교착상태(thread starvation deadlock)라 합니다.

## wait과 notify의 주의사항

 새로운 코드를 작성하는 경우라면 언제나 동시성 유틸리티를 쓰는 것이 좋습니다. 하지만 기존의 레거시 코드를 다뤄야 하는 경우에는 wait()과 notify() 코드를 다뤄야 할 때도 있습니다. wait() 메서드는 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용합니다. 락 객체의 wait() 메서드는 반드시 그 객체를 잠근 동기화 영역 안에서 호출해야 합니다.

**wait() 메서드를 사용하는 표준 방식**

```
synchronized (obj) {
    while (<조건이 충족되지 않았다>)
        obj.wait();	// 락을 놓고 대기 상태로 들어간다.
        
    // 조건이 충족됐을 때의 동작을 수행한다.
}
```

** wait() 메서드를 사용할 때는 반드시 대기 반복문(wait loop) 관용구를 사용해야 하비다. 반복문 밖에서는 절대로 호출하지 말아야 합니다.** 이 반복문은 wait() 호출 전후로 주건이 만족하는지를 검사하는 역할을 합니다.

 notify와 notifyAll 중에서 선택해야 하는 경우에는 일반적으로 notifyAll을 사용하는게 안전합니다. 깨어나야 하는 모든 스레드가 깨어남을 보장할 수 있으니 항상 정확한 결과를 얻을 수 있기 때문입니다.

 모든 스레드가 같은 조건을 기다리고, 조건이 한 번 충족될 때마다 단 하나의 스레드만 혜택을 받을 수 있다면 notifyAll 대신 notify를 사용해 최적화할 수도 있습니다.

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
