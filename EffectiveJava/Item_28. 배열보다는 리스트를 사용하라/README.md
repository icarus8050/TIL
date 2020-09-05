# Item 28. 배열보다는 리스트를 사용하라

#### 배열과 제네릭 타입의 차이

** 배열은 공변(covariant)**입니다. Sub가 Super의 하위 타입이라면 Sub\[\]는 배열 Super\[\]의 하위 타입이 됩니다 (공변, 즉 함께 변한다는 뜻입니다).  **제네릭은 불공변(invariant)**입니다. 즉, 서로 다른 타입 Type1과 Type2가 있을 때, List<Type1>은 List<Type2>의 하위 타입도 아니고 상위 타입도 아닙니다.

**런타임에 실패**

```
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다."; // ArrayStoreException이 발생한다.

```

**컴파일에 실패**

```
List<Object> ol = new ArrayList<Long>(); // 호환되지 않는 타입
ol.add("타입이 달라 넣을 수 없다.");

```

 어느 쪽이든 Long용 저장소에 String을 넣을 수는 없습니다. 다만 배열에서는 그 실수를 런타임에야 알게 되지만, 리스트를 사용하면 컴파일할 때 바로 알 수 있습니다.

** 배열은 실체화(reify)**됩니다. 즉, 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인합니다. 그래서 위의 런타임에 실패했던 코드에서 보듯 Long 배열에 String 을 넣으려 하면 ArrayStoreException이 발생합니다. 반면, **제네릭은 타입 정보가 런타임에는 소거(erasure)**됩니다. 원소 타입을 컴파일 타임에만 검사하며 런타임에는 알수조차 없습니다. 소거는 제네릭이 지원되기 전의 레거시 코드와 제네릭 타입을 함께 사용할 수 있게 해주는 메커니즘입니다.

 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없습니다. 즉, 코드를 new List<E>\[\], new List<String>\[\], new E\[\] 식으로 작성하면 컴파일할 때 제네릭 배열 생성 오류를 일으킵니다.

#### 제네릭 배열을 만들지 못하게 막은 이유

 제네릭 배열은 타입에 안전하지 않기 때문입니다.

```
List<String>[] stringLists = new List<String>[1]; // (1)
List<Integer> intList = List.of(42);              // (2)
Object[] objects = stringLists;                   // (3)
objects[0] = intList;                             // (4)
String s = stringLists[0].get(0);                 // (5)

```

** 위의 코드에서 (1)이 허용된다고 가정했을 때의 상황**을 보겠습니다.

(2) 원소가 하나인 List<Integer>을 생성합니다.

(3) (1)에서 생성한 List<String>의 배열을 Object 배열에 할당합니다. 배열은 공변이니 아무런 문제가 없습니다.

(4) (2)에서 생성한 List<Integer>의 인스턴스를 Object 배열의 첫 원소로 저장합니다. 제네릭은 타입 정보가 소거되기 때문에 성공합니다. 즉, 런타임에는 ListInteger> 인스턴스의 타입은 List가 되고, List<Integer>\[\] 인스턴스의 타입은 List\[\]가 됩니다. 따라서 (4)에서도 ArrayStoreException을 일으키지 않습니다

(5) List<String> 인스턴스만 담겠다고 선언한 stringLists 배열에는 현재 List<Integer>가 저장돼 있습니다. 그리고 (5)는 이 배열의 처음 리스트에서 첫 원소를 꺼내려 하고 있습니다. 컴파일러는 꺼낸 원소를 자동으로 String으로 형변환하는데, 이 원소는 Integer이므로 런타임에 ClassCastException이 발생합니다.

** 위와 같은 문제가 발생하지 않도록 하려면 (1)의 과정에서 컴파일 오류를 내야 합니다.**

#### 실체화 불가 타입(non-reifiable type)

 E, List<E>, List<String> 같은 타입을 실체화 불가 타입이라고 합니다. 실체화되지 않아서 런타임에는 컴파일타임보다 타입 정보를 적게 가지는 타입니다.

 소거 메커니즘 때문에 매개변수화 타입 가운데 실체화 될 수 있는 타입은 List<?>와 Map<?, ?> 같은 비한정적 와일드카드 타입뿐입니다.

#### 배열보다는 리스트를 사용하라

 배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 대부분은 배열인 E\[\] 대신 컬렉션인 List<E>를 사용하면 해결됩니다. 다만 코드가 조금 복잡해지고 성능이 살짝 나빠질 수도 있지만, 그 대신 타입 안전성과 상호운용성은 좋아집니다.

**제네릭을 사용하지 않은 클래스**

```
import java.util.Collection;
import java.util.Random;
import java.util.concurrent.ThreadLocalRandom;

public class Chooser {
    private final Object[] choiceArray;
    
    public Chooser(Collection choices) {
        choiceArray = choices.toArray();
    }
    
    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}

```

 위 코드는 제네릭을 쓰지 않고 컬렉션 안의 원소 중 하나를 무작위로 선택하는 Chooser 클래스입니다. 이 클래스를 사용하려면 choose 메서드를 호출할 때마다 반환된 Object를 원하는 타입으로 형변환해야 합니다. 혹시나 다른 타입의 원소가 들어 있었다면 런타임에 형변환 오류가 날 것입니다.

**제네릭을 사용한 클래스 (컴파일되지 않음)**

```
import java.util.Collection;
import java.util.Random;
import java.util.concurrent.ThreadLocalRandom;

public class Chooser<T> {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        choiceArray = choices.toArray();
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}

```

 위 코드는 컴파일되지 않습니다. 컬렉션의 toArray() 메서드가 Object\[\] 를 반환하기 때문입니다. 이를 형변환하도록 코드를 다음과 같이 변경해야 합니다.

```
choiceArray = (T[]) choices.toArray();
```

 하지만 위와 같은 코드는 T가 무슨 타입인지 알 수 없으니 컴파일러는 이 형변환이 런타임에도 안전할지 보장할 수 없다는 메시지를 내보낼 것입니다. 코드를 작성하는 사람이 안전하다고 확신한다면 주석을 남기고 애너테이션을 달아 경고를 숨겨도 됩니다. 하지만 애초에 경고의 원인을 제거하는 편이 훨씬 낫습니다.

 비검사 형변환 경고를 제거하려면 배열 대신 리스트를 쓰면 됩니다.

```
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.Random;
import java.util.concurrent.ThreadLocalRandom;

public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}

```

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEA&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEA&Kc=)
