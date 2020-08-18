# Item 8. finalizer와 cleaner 사용을 피하라

 자바에서는 두 가지 객체 소멸자를 제공합니다. 바로 finalizer와 cleaner 입니다.

 **finalizer 는 예측할 수 없고, 상황에 따라 위험할 수 있기 때문에 일반적으로 불필요합니다.** 오동작, 낮은 성능, 이식성 문제의 원인이 되기도 합니다. 그래서 자바 9에서는 finalizer를 deprecated API로 지정하고 cleaner 를 그 대안으로 소개합니다. **cleaner는 finalizer 보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요합니다.**

#### finalizer와 cleaner의 부작용

finalizer와 cleaner 는 즉시 수행된다는 보장이 없습니다. 객체에 접근할 수 없게 된 후 finalizer나 cleaner가 실행되기까지 얼마나 걸릴지 알 수 없습니다. 즉, **finalizer와 cleaner로는 제때 실행되어야 하는 작업은 절대 할 수 없습니다.**

 finalizer나 cleaner를 얼마나 신속히 수행할지는 전적으로 가비지 컬렉터 알고리즘에 달렸으며, 이는 가비지 컬렉터 구현마다 천차만별입니다. 자바 언어 명세에서는 두 소멸자의 수행 시점뿐만 아니라 수행 여부조차 보장하지 않습니다. 따라서 **상태를 영구적으로 수정하는 작업에서는 finalizer나 cleaner에 의존해서는 안됩니다.**

 finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료됩니다. 잡지 못한 예외 때문에 해당 객체는 마무리가 덜 된 상태로 남을 수 있습니다.

 finalizer와 cleaner는 가비지 컬렉터의 효율을 떨어뜨리기 때문에 심각한 성능 문제도 동반합니다.

 finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있습니다. 생성자나 직렬화 과정에서 예외가 발생하면, 이 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있게 됩니다. 이 finalizer는 정적 필드에 자신의 참조를 할당하여 가비지 컬렉터가 수집하지 못하게 막을 수 있습니다. 이 객체는 메서드 호출을 통해 애초에는 허용되지 않았을 작업을 수행하게 할 수 있게 됩니다. final 클래스들은 하위 클래스를 만들 수 없으니 이 공격으로부터 안전합니다. final이 아닌 클래스를 finalizer 공격으로부터 방어하려면 아무 일도 하지 않는 finalize 메서드를 만들고 final로 선언해야 합니다.

#### 객체의 클래스에서 finalizer나 cleaner를 대신하는 방법

 AutoCloseable을 구현하고, 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출하도록 하면 됩니다. 예외가 발생해도 제대로 종료될 수 있도록 try-catch-resources를 사용해야 합니다.

#### finalizer와 cleaner의 쓰임새

-   자원의 소유자가 close() 메서드를 호출하지 않는 것에 대비한 안전망 역할을 합니다. (finalizer나 cleaner가 언제 호출될지 보장할 수 없지만, 클라이언트가 하지 않은 자원 회수를 늦게라도 하는 것이 낫기때문입니다.)
-   네이티브 피어(native peer)와 연결된 객체에서 사용합니다. 네이티브 피어란 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말합니다. 네이티브 객체는 자바 객체가 아니므로 가비지 컬렉터가 그 존재를 알지 못합니다. 때문에 가비지 컬렉터가 네이티브 객체 자원을 회수하지 못합니다. 이를 위해 finalizer나 cleaner가 사용됩니다. 단, 성능 저하를 감당할 수 있거나 네이티브 피어가 심각한 자원을 가지고 있지 않을 때에만 해당됩니다. 만약 그렇지 않다면 **close() 메서드**를 사용해야 합니다.

#### cleaner를 안전망으로 활용하는 AutoCloseable 클래스 예시

```
import java.lang.ref.Cleaner;

public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    private static class State implements Runnable {
        int numJunkPiles;

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        @Override
        public void run() {
            System.out.println("방 청소");
            numJunkPiles = 0;
        }
    }

    private final State state;

    private final Cleaner.Cleanable cleanable;
    
    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override
    public void close() throws Exception {
        cleanable.clean();
    }
}

```

 위 코드 예시에서는 Room 자원을 수거하기 전에 쓰레기들을 청소(clean)해야 한다고 가정한 예시입니다. static으로 선언된 중첩 클래스인 State는 cleaner가 방을 청소할 때 수거할 자원(numJunkPiles)들을 담고 있습니다. State는 Runnable을 구현하고 있고, run() 메서드가 cleanable에 의해 한 번만 호출됩니다.

 run()은 Room의 close() 메서드를 호출하거나 가비지 컬렉터가 Room을 회수할 때까지 클라이언트가 close()를 호출하지 않는다면, cleaner가 State의 run()을 호출(보장되지 않음)해줄 것입니다.

 여기서 State 인스턴스는 절대로 Room 인스턴스를 참조해서는 안됩니다. Room 인스턴스를 참조할 경우 순환참조가 생기기 때문에 가비지 컬렉터가 Room 인스턴스를 회수할 수 없습니다. 이를 방지하기 위해서 State 클래스가 **정적 중첩 클래스**이어야 합니다. 정적이 아닌 중첩 클래스는 자동으로 바깥 객체의 참조를 갖게 되기 때문입니다.

---

## 참고자료

[http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
