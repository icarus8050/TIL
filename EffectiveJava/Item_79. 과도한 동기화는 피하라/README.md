# Item 79. 과도한 동기화는 피하라

 과도한 동기화는 성능을 떨어뜨리고, 교착상태에 빠뜨리고, 심지어 예측할 수 없는 동작을 낳기도 합니다.

** 응답 불가와 한전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에게 양도하면 안 됩니다.** 예를 들어 동기화된 영역 안에서는 재정의할 수 있는 메서드를 호출하면 안 되며, 클라이언트가 넘겨준 함수 객체를 호출해서도 안됩니다.

**동기화 블록 안에서 외부 메서드를 호출하는 잘못된 코드**

```
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.Set;

public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers
            = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized (observers) {
            for (SetObserver<E> observer : observers) {
                observer.added(this, element);
            }
        }
    }

    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element);
        return result;
    }
}

```

```
@FunctionalInterface
public interface SetObserver<E> {
    // ObservableSet에 원소가 추가되면 호출된다.
    void added(ObservableSet<E> set, E element);
}

```

 위의 ObservableSet 클래스에서 상속한 ForwardingSet는 Item 18에서 사용했던 클래스입니다.

**0 ~ 99까지 출력하는 예제**

```
import java.util.HashSet;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Main {
    public static void main(String[] args) {
        ObservableSet<Integer> set =
                new ObservableSet<>(new HashSet<>());

        set.addObserver((s, e) -> System.out.println(e));

        for (int i = 0; i < 100; i++) {
            set.add(i);
        }
    }
}

```

 위 코드는 아무 이상없이 0부터 99까지 출력합니다.

**ConcurrentModificationException 이 발생하는 예제**

```
import java.util.HashSet;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Main {
    public static void main(String[] args) {
        ObservableSet<Integer> set =
                new ObservableSet<>(new HashSet<>());

        set.addObserver(new SetObserver<Integer>() {
            @Override
            public void added(ObservableSet<Integer> set, Integer element) {
                System.out.println(element);
                if (element == 23)
                    set.removeObserver(this);
            }
        });

        for (int i = 0; i < 100; i++) {
            set.add(i);
        }
    }
}

```

 위 코드는 아까 보았던 예제와 달리 ConcurrentModificationException을 던집니다. 관찰자의 added 메서드 호출이 일어난 시점에 notifyElementAdded가 observers에 대해 Lock을 걸고 리스트를 순회하는 도중이었기 때문입니다.

 added 메서드는 ObservableSet의 removeObserver 메서드를 호출하고, 이 메서드는 다시 observers.remove 메서드를 호출합니다. 리스트에서 원소를 제거하려는데, 이미 이전에 notifyElementAdded 메서드가 Lock을 걸고 순회하고 있었기 때문에 발생하는 문제입니다.

**쓸데없이 백그라운드 스레드를 사용하는 코드**

```
import java.util.HashSet;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Main {
    public static void main(String[] args) {
        ObservableSet<Integer> set =
                new ObservableSet<>(new HashSet<>());

        set.addObserver(new SetObserver<Integer>() {
            @Override
            public void added(ObservableSet<Integer> set, Integer element) {
                System.out.println(element);
                if (element == 23) {
                    ExecutorService exec =
                            Executors.newSingleThreadExecutor();

                    try {
                        exec.submit(() -> set.removeObserver(this)).get();
                    } catch (ExecutionException | InterruptedException ex) {
                        throw new AssertionError(ex);
                    } finally {
                        exec.shutdown();
                    }
                }
            }
        });

        for (int i = 0; i < 100; i++) {
            set.add(i);
        }
    }
}

```

 위 코드는 removeObserver를 직접 호출하지 않고 실행자 서비스(ExecutorService)를 사용하여 다른 스레드에게 위임합니다. 이 프로그램은 예외는 발생하지 않지만 교착상태에 빠집니다. 백그라운드 스레드가 set.removeObserver 메서드를 호출하면 관찰자를 잠그려고 시도하지만 메인 스레드가 이미 락을 쥐고 있기 때문입니다.

 이 예시는 굳이 백그라운드 스레드를 이용할 필요가 없는데도 작성한 코드여서 억지스럽지만, 실제 시스템에서도 동기화된 영역 안에서 외부 메서드를 호출하여 교착상태에 빠지는 사례가 있다는 사실을 알려주기 위한 예시입니다.

#### 락의 재진입(reentrant)

 아까 위에서 보았던 예시에서 동기화 영역이 보호하는 자원(관찰자)은 외부 메서드(added)가 호출될 때 일관된 상태였습니다. 하지만 똑같은 상황에서 불변식이 임시로 깨진 경우라면 자바 언어의 락은 재진입(reentrant)을 허용하므로 교착상태에 빠지지 않습니다.

**예외를 발생시켰던 예시의 경우**

 외부 메서드를 호출하는 스레드는 이미 락을 쥐고 있으므로 다음번 락 획득도 성공합니다. 그 락이 보호하는 데이터에 대해 개념적으로 관련이 없는 다른 작업이 진행 중이어도 락 획득을 성공하고, 이는 원하지 않는 결과를 유발할 수도 있습니다.

 재진입 가능 락은 객체 지향 멀티스레드 프로그램을 쉽게 구현할 수 있도록 해주지만, 응답 불가(교착상태)가 될 상황을 안전 실패(데이터 훼손)로 변모시킬 수도 있습니다.

#### 락의 재진입 문제 해결 방법

 외부 메서드 호출을 동기화 블록 바깥으로 옮기는 간당한 방법으로 해결할 수 있습니다. 위의 notifyElementAdded 메서드에서라면 관찰자 리스트를 복사해 쓰면 락 없이도 안전하게 순회할 수 있습니다.

```
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers) {
        snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer : snapshot)
        observer.added(this, element);
}
```

#### CopyOnWriteArrayList

** 자바의 동시성 컬렉션 라이브러리의 CopyOnWriteArrayList**를 이용하면 더 나은 방법으로 해결할 수 있습니다. 이름에서 알 수 있듯이 ArrayList를 구현한 클래스로, 내부를 변경하는 작업은 항상 복사본을 만들어 수행하도록 구현되어 있습니다. 내부의 배열은 절대 수정되지 않으니 순회할 때 락이 필요 없어 매우 빠릅니다.

#### 동기화의 기본 규칙

 동기화 영역 바깥에서 호출되는 외부 메서드를 열린 호출(open call)이라 하는데, 이 메서드는 얼마나 오래 실행될 지 알 수가 없습니다. 동기화 영역 안에서 호출한다면 그동안 다른 스레드는 보호된 사원을 사용하지 못하고 대기해야만 하는 상황이 발생할 수 있습니다.

** 동기화 영역에서는 가능한 한 일을 적게 해야합니다.**

#### 동기화의 비용

과도한 동기화가 초래하는 진짜 비용은 락을 얻는 데 드는 CPU 시간이 아닙니다. 바로 경쟁하느라 낭비하는 시간, 즉 병렬로 실행할 기회를 잃고, 모든 코어가 메모리를 일관되게 보기 위한 지연시간이 진짜 비용입니다.

 가상머신의 코드 최적화를 제한한다는 점도 과도한 동기화의 숨은 비용입니다.

#### 가변 클래스를 동기화하는 방법

1.  동기화를 전혀 하지 말고, 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화하게 합니다.
2.  동기화를 내부에서 수행해 스레드 안전한 클래스로 만듭니다.

 단, 클라이언트가 외부에서 객체 전체에 락을 거는 것보다 동시성을 월등히 개선할 수 있을 때만 두 번째 방법을 선택해야 합니다.

 클래스를 내부에서 동기화하기로 했다면, 락 분할(lock splitting), 락 스트라이핑(lock striping), 비차단 동시성 제어(nonblocking concurrency control) 등 다양한 기법을 동원해 동시성을 높여줄 수 있습니다.

 여러 스레드가 호출할 가능성이 있는 메서드가 정적 필드를 수정한다면 그 필드를 사용하기 전에 반드시 동기화해야 합니다(비결정적 행동도 용인하는 클래스라면 상관없습니다).

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
