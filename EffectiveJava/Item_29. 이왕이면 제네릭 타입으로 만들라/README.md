# Item 29. 이왕이면 제네릭 타입으로 만들라

 아래의 클래스는 Effective Java Item 7에서 다루었던 Stack 클래스입니다.

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
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size);
    }
}

```

 이 클래스는 클라이언트가 스택으로부터 객체를 꺼낸 후에 형변환을 해야합니다. 이는 런타임 오류가 날 위험이 있습니다. 따라서 제네릭 타입으로 구현하는 것이 좋습니다.

 일반 클래스를 제네릭 클래스로 만드는 첫 단계는 클래스 선언에 타입 매개 변수를 추가하는 것입니다.

**제네릭 스택으로 가는 첫 단계 (컴파일되지 않음)**

```
import java.util.Arrays;
import java.util.EmptyStackException;

public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY]; // 컴파일 에러
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null;
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size);
    }
}

```

 위 코드에서 컴파일 에러가 나는 이유는 E와 같은 실체화 불가 타입으로는 배열을 만들 수 없기 때문입니다. 이에 대한 해결책으로는 두 가지 가 있습니다.

 첫 번째 제네릭 배열 생성을 금지하는 제약을 우회하는 방법입니다. Object 배열을 생성한 다음 제네릭 배열로 형변환하면 됩니다. 하지만 컴파일러는 해당 형변환이 타입 안전한지 알 수 없기 때문에 비검사 형변환 경고를 띄울 것입니다. elements 배열은 private 필드이고, 클라이언트로 반환되거나 다른 메서드에 전달되는 일이 없습니다. 그리고 push() 메서드를 통해 배열에 저장되는 원소의 타입이 E임을 알고 있습니다. 따라서 비검사 형변환은 확실하게 안전하므로 @SuppressWarnnings 애너테이션으로 해당 경고를 숨기는 것이 좋습니다(아이템 27).

```
import java.util.Arrays;
import java.util.EmptyStackException;

public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    /**
     * 배열 elements는 push(E)로 넘어온 E 인스턴스만 받는다.
     * 따라서 타입 안전성을 보장하지만,
     * 이 배열의 런타임 타입은 E[]가 아닌, Object[]다.
     */
    @SuppressWarnings("unchecked")
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null;
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size);
    }
}

```

두 번째 방법은 elements 필드의 타입을 E\[\]에서 Object\[\]로 바꾸는 것입니다. 그러면 pop() 메서드에서 타입 에러가 발생하는데, 해당 원소를 E로 형변환 합니다. 형변환하면 아까와 마찬가지로 비검사 형변환에 대한 경고가 나타나는데, push()에서 E 타입만 허용하므로 타입 안전성을 보장할 수 있으므로 @SuppressWarnning 애너테이션으로 경고를 숨겨줍니다.

```
import java.util.Arrays;
import java.util.EmptyStackException;

public class Stack<E> {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        
        // push에서 E 타입만 허용하므로 이 형변환은 안전하다.
        @SuppressWarnings("unchecked")
        E result = (E) elements[--size];
        elements[size] = null;
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size);
    }
}

```

#### 아이템 28 "배열보다는 리스트를 우선하라"와의 모순?

 이번 주제는 "배열보다는 리스트를 우선하라"라는 이야기와는 모순이 되어 보입니다. 하지만 제네릭 타입 안에서 리스트를 사용하는 게 항상 가능하지도, 꼭 더 좋은 것도 아닙니다.

-   자바가 리스트를 기본 타입으로 제공하지 않으므로 ArrayList 같은 제네릭 타입도 결국 기본 타입인 배열을 사용해 구현해야 합니다.
-   HashMap 같은 제네릭 타입은 성능을 높일 목적으로 배열을 사용하기도 합니다.

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
