# Item 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라

#### 가변인수

 가변인수는 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 해줍니다. 형태는 다음과 같습니다.

```
public static void example(String... args) {
    //....
}
```

 가변인수 메서드는 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 만들어집니다. 하지만 내부로 감춰야 했을 이 배열이 클라이언트에 노출된다는 문제점이 있습니다. 그 결과로 varargs 매개변수에 제네릭이나 매개변수화 타입이 포함되면 컴파일 경고가 발생하게 됩니다.

 제네릭과 같은 실체화 불가 타입은 런타임에는 컴파일타임보다 타입 관련 정보를 적게 담고 있습니다. 즉, 메서드를 선언할 때 실체화 불가 타입으로 varargs 매개변수를 선언하면 컴파일러가 경고를 보내게 됩니다. 가변인수 메서드를 호출할 때도 varargs 매개변수가 실체화 불가 타입으로 추론되면, 그 호출에 대해서 다음과 같이 경고를 보냅니다.

```
warning: [unchecked] Possible heap pollution from
    parameterized varargs type List<String>
```

 매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생합니다.

**제네릭과 가변인수를 혼영하여 타입 안전성이 깨진 예제**

```
    static void dangerous(List<String>... stringLists) {
        List<Integer> integerList = List.of(42); 
        Object[] objects = stringLists;
        objects[0] = integerList;   // 힙 오염 발생
        String s = stringLists[0].get(0);   // ClassCastException
    }
```

 위 메서드는 형변환하는 곳이 보이지 않는데도 인수를 건네 호출하면 ClassCastException을 던집니다. 마지막 줄에 컴파일러가 생성한 (보이지 않는) 형변환이 숨어 있기 때문입니다. 이처럼 타입 안전성이 깨지니 **제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않습니다.**

#### @SafeVarargs

 자바 7 전에는 제네릭 가변인수 메서드의 작성자가 호출자 쪽에서 발생하는 경고에 대해서 해줄 수 있는 일이 없었습니다. 따라서 사용자는 이 경고들을 그냥 두거나 (더 흔하게는) 호출하는 곳마다 @SuppressWarnings("unchecked") 애너테이션을 달아 경고를 숨겨야 했습니다.

 자바 7에서는 @SafeVarargs 애너테이션이 추가되어 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있게 되었습니다. **@SafeVarargs 애너테이션은 메서드 작성가 그 메서드가 타입 안전함을 보장하는 장치입니다.**

#### 안전한 제네릭 가변인수 메서드를 사용하는 방법

 메서드가 안전한 게 확실하지 않다면 @SafeVarargs 애너테이션을 달아서는 안 됩니다. **해당 메서드는 varargs 배열에 아무것도 저장하지 않고(그 매개변수들을 덮어쓰지 않고) 그 배열의 참조가 밖으로 노출되지 않는다면(신뢰할 수 없는 코드가 배열에 접근할 수 없다면) 타입 안전성이 보장됩니다.** 

**자신의 제네릭 매개변수 배열의 참조를 노출하는 코드(안전하지 않은 메서드)**

```
    static <T> T[] toArray(T... args) {
        return args;
    }
```

 이 메서드가 반환하는 배열의 타입은 이 메서드에 인수를 넘기는 컴파일타임에 결정되는데, 그 시점에는 컴파일러에게 충분한 정보가 주어지지 않아 타입을 잘못 판단할 수 있습니다. 따라서 자신의 varargs 매개변수 배열을 그대로 반환하면 힙 오염을 이 메서드를 호출한 쪽의 콜스택으로까지 전이하는 결과를 낳을 수 있습니다.

**T 타입의 인수 3개를 받아 그중 2개를 무작위로 골라 담은 배열을 반환하는 메서드**

```
    static <T> T[] pickTwo(T a, T b, T c) {
        switch (ThreadLocalRandom.current().nextInt(3)) {
            case 0: return toArray(a, b);
            case 1: return toArray(a, c);
            case 2: return toArray(b, c);
        }
        throw new AssertionError(); // 도달할 수 없다.
    }
```

 위의 메서드는 제네릭 가변인수를 받는 toArray() 메서드를 호출하고 있습니다. 컴파일러는 toArray()에 넘길 T 인스턴스 2개를 담을 varargs 매개변수 배열이 만드는 코드를 생성합니다. 이 코드가 만드는 배열의 타입은 Object\[\]인데, pickTwo에 어떤 타입의 객체를 넘기더라도 담을 수 있는 가장 구체적인 타입이기 때문입니다. 그리고 toArray() 메서드가 돌려준 이 배열이 그대로 pickTwo()를 호출한 클라이언트까지 전달됩니다. 즉, pickTwo()는 항상 Object\[\] 타입 배열을 반환합니다.

```
    public static void main(String[] args) {
        String[] attributes = pickTwo("좋은", "빠른", "저렴한");
    }
```

 위에서 작성했던 코드들을 기반으로 main 함수를 작성하면 ClassCastException이 발생합니다. 이는 pickTwo()의 반환값을 attributes에 저장하기 위해 String\[\]로 형변환하는 코드가 컴파일러가 자동 생성하기 때문입니다. 이 코드들은 힙 오염을 발생시킨 진짜 원인인 toArray()로부터 두 단계나 떨어져 있습니다.

 방금 본 예는 **제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다**는 점을 잘 보여주고 있습니다.

단, 예외가 두 가지 있습니다.

-   @SafeVarargs로 제대로 애너테이트된 또 다른 varargs 메서드에 넘기는 것은 안전합니다.
-   varargs 매개변수 배열 내용의 일부 함수를 호출만 하는(varargs를 받지 않는) 일반 메서드에 넘기는 것도 안전합니다.

**제네릭 varargs 매개변수를 안전하게 사용하는 메서드**

```
    @SafeVarargs
    static <T> List<T> flatten(List<? extends T>... lists) {
        List<T> result = new ArrayList<>();
        for (List<? extends T> list : lists) {
            result.addAll(list);
        }
        return result;
    }
```

#### @SafeVarargs 애너테이션을 사용해야 할 때를 정하는 규칙

 규칙은 간단합니다. **제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 @SafeVarargs를 달아야 합니다.** 이 말은 안전하지 않은 varargs 메서드는 절대 작성해서는 안 된다는 뜻입니다.

-   varargs 매개변수 배열에 아무것도 저장하지 않습니다.
-   그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않습니다.

 @SafeVarargs 애너테이션은 재정의할 수 없는 메서드에만 달아야 합니다. 재정의한 메서드도 안전할지는 보장할 수 없기 때문입니다. 자바 8에서 이 애너테이션은 오직 정적 메서드와 final 인스턴스 메서드에만 붙일 수 있고, 자바 9부터는 private 인스턴스 메서드에도 허용됩니다.

#### @SafeVarargs 애너테이션을 이용하지 않는 다른 방법

 (실체는 배열인) varargs 매개변수를 List 매개변수로 바꿀 수도 있습니다. 이 방식을 앞에서 살펴 보았던 flatten() 메서드에 적용하면 다음과 같이 작성할 수 있습니다. 단순히 매개변수 선언만 수정한 코드입니다.

```
    static <T> List<T> flatten(List<List<? extends T>> lists) {
        List<T> result = new ArrayList<>();
        for (List<? extends T> list : lists) {
            result.addAll(list);
        }
        return result;
    }
```

 정적 팩토리 메서드인 List.of()를 활용하면 다음 코드와 같이 이 메서드에 임의 개수의 인수를 넘길 수 있습니다. 이렇게 사용하는 게 가능한 이유는 List.of()에도 @SafeVarargs 애너테이션이 달려 있기 때문입니다.

```
audience = flattern(List.of(frends, romans, countrymen));
```

 이 방식의 장점은 컴파일러가 이 메서드의 타입 안전성을 검증할 수 있다는 데 있습니다. @SafeVarargs 애너테이션을 직접 달지 않아도 되며, 실수로 안전하다고 판단할 염려도 없습니다. 단점은 클라이언트 코드가 살짝 지저분해지고 속도가 조금 느려질 수 있다는 정도입니다.

 이 방식을 pickTwo에 적용하면 다음과 같습니다.

```
    static <T> List<T> pickTwo(T a, T b, T c) {
        switch (ThreadLocalRandom.current().nextInt(3)) {
            case 0: return List.of(a, b);
            case 1: return List.of(a, c);
            case 2: return List.of(b, c);
        }
        throw new AssertionError(); // 도달할 수 없다.
    }
```

```
    public static void main(String[] args) {
        List<String> attributes = pickTwo("좋은", "빠른", "저렴한");
    }
```

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LET&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LET&Kc=)
