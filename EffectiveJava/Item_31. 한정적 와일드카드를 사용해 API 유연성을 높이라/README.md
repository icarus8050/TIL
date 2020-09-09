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