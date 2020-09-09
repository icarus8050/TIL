# Item 30. 한정적 와일드카드를 사용해 API 유연성을 높이라

 매개변수화 타입은 불공변(invariant)입니다. 즉, 서로 다른 타입 Type1과 Type2가 있을 때 List<Type1>은 List<Type2>의 하위 타입도 상위 타입도 아닙니다.

 매개변수화 타입의 불공변이라는 특성보다 유연한 설계가 필요할 때는 한정적 와일드카드를 이용하면 됩니다.

```
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
}

```

 위 스택 클래스에서 일련의 원소를 넣는 메서드를 추가해야 한다고 가정해보겠습니다.

```
public void pushAll(Iterable<E> src) {
    for (E e : src)
        push(e);
}
```

 이 메서드는 정상적으로 컴파일되지만, Stack<Number>로 선언한 후에 pushAll(intVal)을 호출하면 오류 메시지가 나타납니다(여기서 intVal은 Integer 타입입니다).

```
Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = ...;
numberStack.pushAll(integers);

```

 이는 매개변수화 타입이 불공변이기 때문에 발생하는 문제입니다.

 이를 위한 해결책이 한정적 와일드카드입니다. pushAll의 입력 매개변수 타입은 'E의 Iterable'이 아니라 'E의 하위 타입의 Iterable'이어야 하며, 와일드카드 타입 Iterable<? extends E>가 이러한 뜻을 나타냅니다. 이를 반영한 코드를 살펴보면 아래와 같습니다.

**E 생산자(producer) 매개변수에 한정적 와일트카드 타입 적용**

```
    public void pushAll(Iterable<? extends E> src) {
        for (E e : src) {
            push(e);
        }
    }
```

 이제 반대로 popAll() 메서드를 살펴 보겠습니다.

```
public void popAll(Collection<E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

 위 코드는 Collection<Object>가 Collection<Number>의 하위 타입이 아니기 때문에 문제가 발생합니다.

```
Stack<Number> numberStack = new Stack<>();
Collection<Object> objects = ...;
numberStack.popAll(objects);
```

 이를 해결하기 위해서 마찬가지로 한정적 와일드카드 타입을 활용할 수 있습니다. 이번에는 'E의 Collection'이 아니라 'E의 상위 타입의 Collection'이어야 합니다.

**E 소비자(consumer) 매개변수에 와일드카드 타입 적용**

```
    public void popAll(Collection<? super E> dst) {
        while (!isEmpty()) {
            dst.add(pop());
        }
    }
```

 위의 과정을 통해 알 수 있는 메시지는 다음과 같습니다.

** 유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라.**

 단, 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 써도 좋을 게 없습니다. 타입을 정확히 지정해야 하는 상황으로, 이때는 와일트카드 타입을 쓰지 말아야 합니다.

#### 펙스(PECS) 공식 : producer-extends, consumer-super

 매개변수화 타입 T가 생산자라면 <? extends T>를 사용하고, 소비자라면 <? super T>를 사용해야 한다는 공식입니다.

 제대로만 사용한다면 클래스 사용자는 와일드카드 타입이 쓰였다는 사실조차 의식하지 못하며 API를 사용할 것입니다. 반아들여야 할 매개변수를 받고 거절해야 할 매개변수는 거절하는 작업이 알아서 이뤄집니다. **클래스 사용자가 와일드카드 타입을 신경 써야 한다면 그 API에 무슨 문제가 있을 가능성이 큽니다.**

#### 조금 더 복잡한 예제

```
public static <E extends Comparable<E>> E max(List<E> list)
```

 위의 max() 메서드는 와일드카드 타입을 사용해 다음과 같이 고칠 수 있습니다.

```
public static <E extends Comparable<? super E>> E max(List<? extends E> list)
```

 이번 예제는 PECS 공식이 두 번 적용되어 있습니다.

 첫 번째로 입력 매개변수는 E 인스턴스를 생산하므로 원래의 List<E>를 List<? extends E>로 수정했습니다.

 두 번째는 타입 매개변수 부분입니다. 원래 선언에서는 E가 Comparable<E>를 확장한다고 정의했는데, 이때 Comparable<E>는 E 인스턴스를 소비합니다(그리고 선후 관계를 뜻하는 정수를 생산합니다). 그래서 매개변수화 타입 Comparable<E>를 한정적 와일드카드 타입인 Comparable<? super E>로 대체했습니다. Comparable은 언제나 소비자이므로, **일반적으로 Comparable<E>보다는 Comparable<? super E>를 사용하는 편이 더 낫습니다. Comparator도 마찬가지 입니다.**

#### 타입 매개변수와 와일드카드, 둘 중 어느 것을 사용해도 괜찮을 경우

 타입 매개변수와 와일드카드에는 공통되는 부분이 있어서, 메서드를 정의할 때 둘 중 어느 것을 사용해도 괜찮을 때가 많습니다. 예를 들어 주어진 리스트에서 명시한 두 인덱스의 아이템들을 교환(swap)하는 정적 메서드를 두 방식 모두로 정의해보겠습니다.

```
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

 만약 public API라면 간단한 두 번째가 더 낫습니다. 어떤 리스트든 이 메서드에 넘기면 명시한 인덱스의 원소들을 교환해 줄 것입니다.

 기본 규칙은 이렇습니다. **메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체합니다.** 이때 비한정적 타입매개변수라면 비한정적 와일드카드로 바꾸고, 한정적 타입 매개변수라면 한정적 와일드카드로 바꾸변 됩니다.

 하지만 두 번째 swap 선언에는 문제가 하나 있는데, 다음과 같이 직관적으로 구현한 코드가 컴파일되지 않는다는 것입니다.

```
public static void swap(List<?> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```

 원인은 리스트의 타입이 List<?>인데, List<?>에는 nul 이외에는 어떤 값도 넣을 수 없다는 데 있습니다. 이를 해결하기 위해서는 와일드카드 타입의 실제 타입을 알려주는 메서드를 private 도우미 메서드로 따로 작성하여 활용하는 방법입니다.

```
public static void swap(List<E> list, int i, int j) {
    swapHelper(list, i, j);
}

// 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```

 swapHelper 메서드는 리스트가 List<E> 임을 알고 있습니다. 즉, 이 리스트에서 꺼낸 값의 타입은 항상 E이고, E 타입의 값이라면 이 리스트에 넣어도 안전함을 알고 있습니다. 이상으로 swap 메서드 내부에서는 더 복잡한 제네릭 메서드를 이용했지만, 덕분에 외부에서는 와일드카드 기반의 깔끔한 메서드를 유지할 수 있습니다. 즉, swap 메서드를 호출하는 클라이언트는 복잡한 swapHelper의 존재를 모른 채 그 혜택을 누릴 수 있습니다.

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LET&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LET&Kc=)
