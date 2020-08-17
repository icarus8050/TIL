# Item 7. 다 쓴 객체 참조를 해제하라

 아래의 코드는 메모리 누수가 일어나고 있는 Stack 클래스 입니다.

```
import java.util.Arrays;
import java.util.EmptyStackException;

public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size);
    }
}

```

 이 코드는 겉으로 보기에는 딱히 문제가 없지만 메모리 누수라는 문제가 숨어 있습니다. 이 스택을 사용하는 프로그램을 오래 실행하다 보면 점차 가비지 컬렉션 활동과 메모리 사용량이 늘어나게 되고, 결국 성능이 저하될 것입니다. 메모리 누수가 심한 경우에는 디스크 페이징이나 OutOfMemoryError를 일으켜 프로그램이 예기치 않게 종료되기도 합니다.

 코드상에서 스택 크기가 커졌다가 줄어들었을 때 스택에서 꺼내진 객체들을 GC가 회수하지 않습니다. 프로그램에서 그 객체들을 더 이상 사용하지 않더라도 말입니다. 이는 스택이 배열에서 여전히 그 객체를 참조하고 있기 때문입니다. 이러한 객체 참조를 하나 살려두면 GC는 해당 객체 뿐만 아니라 그 객체가 참조하고 있는 모든 객체(그리고 그 객체들이 참조하고 있는 모든 객체)를 회수하지 못합니다.

 이러한 문제점들의 해법은 간단합니다. 다쓴 객체는 null 처리하여 참조를 해제시켜주면 됩니다. pop() 메서드의 코드를 예로 바꾸어 보면 아래와 같습니다.

```
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null;
    return result;
}
```

** 객체 참조를 null 처리하는 일은 예외적인 경우여야 합니다. **다 쓴 참조를 해체하는 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위(Scope) 밖으로 밀어내는 것입니다.

#### null 처리를 해야할 때

 **일반적으로 자기 메모리를 직접 관리하는 클래스라면 원소를 다 사용한 즉시 null 처리하여 메모리 누수에 주의를 해야합니다.**

 캐시 역시 메모리 누수의 누수를 일으키는 주범입니다. 이를 해결하는 방법은 아래와 같습니다.

-   캐시 외부에서 키(Key)를 참조하는 동안만 엔트리가 살아 있는 캐시가 필요한 상황이라면 WeakHashMap을 사용을 사용한다.
-   시간이 지날수록 엔트리의 가치를 떨어뜨리는 방식으로 캐시의 유효 기간을 정의한다. (에이징 기법)

 두 번째 항목의 방법은 쓰지 않는 엔트리를 청소해주어야 합니다. (Scheduled ThreadPoolExecutor 같은) 백그라운드 스레드를 활용하거나 캐시에 새 엔트리를 추가할 때 부수 작업으로 수행하는 방법이 있습니다. LinkedHashMap은 removeEldestEntry() 메서드를 사용하여 후자의 방식으로 처리합니다. removeEldestEntry() 메서드는 간단하게 말씀드리자면 Map 에서 가장 오래된 항목의 제거 여부를 판단하는 메서드입니다. 이 메서드는 put() 이나 putAll() 메서드에 의해 호출됩니다. 자세한 사항은 아래의 링크를 참조하시면 됩니다.

[https://www.tutorialspoint.com/java/util/linkedhashmap\_removeeldestentry.htm](https://www.tutorialspoint.com/java/util/linkedhashmap_removeeldestentry.htm)

---

## 참고자료

[http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
