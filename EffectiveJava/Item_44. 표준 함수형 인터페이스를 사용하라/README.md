# Item 44. 표준 함수형 인터페이스를 사용하라

 자바 8부터는 람다 지원을 위해 java.util.function 패키지에 표준 함수형 인터페이스를 다양하게 제공하고 있습니다. **필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하는 것이 좋습니다.** 그러면 API가 다루는 개념의 수가 줄어들어 익히기 더 쉬워지게 됩니다. 또한 유용한 디폴트 메서드를 많이 제공하므로 다른 코드와의 상호운용성도 크게 좋아지게 됩니다.

 java.util.function 패키지에는 총 43개의 인터페이스가 담겨 있습니다. 이중에 기본 인터페이스 6개만 기억하면 나머지를 충분히 유추해 낼 수 있습니다.

-   Operator : 인수가 1개인 UnaryOperator와 2개인 BinaryOperator로 나뉘며, 반환값과 인수의 타입이 같은 함수를 뜻합니다.
-   Predicate : 인수 하나를 받아 boolean을 반환하는 함수를 뜻합니다.
-   Function : 인수와 반환 타입이 다른 함수를 뜻합니다.
-   Supplier : 인수를 받지 않고 값을 반환(혹은 제공)하는 함수를 뜻합니다.
-   Comsumer : 인수를 하나 받고 반환값은 없는(특히 인수를 소비하는) 함수를 뜻합니다.

**기본 함수형 인터페이스 표**

<table style="border-collapse: collapse; width: 100%; height: 114px;" border="1" data-ke-style="style4"><tbody><tr style="height: 19px;"><td style="width: 33.3333%; height: 19px;"><b>인터페이스</b></td><td style="width: 33.3333%; height: 19px;"><b>함수 시그니처</b></td><td style="width: 33.3333%; height: 19px;"><b>예</b></td></tr><tr style="height: 19px;"><td style="width: 33.3333%; height: 19px;">UnaryOperator&lt;T&gt;</td><td style="width: 33.3333%; height: 19px;">T apply(T t)</td><td style="width: 33.3333%; height: 19px;">String::toLowerCase</td></tr><tr style="height: 19px;"><td style="width: 33.3333%; height: 19px;">BinaryOperator&lt;T&gt;</td><td style="width: 33.3333%; height: 19px;">T apply(T t1, T t2)</td><td style="width: 33.3333%; height: 19px;">BigInteger::add</td></tr><tr style="height: 19px;"><td style="width: 33.3333%; height: 19px;">Predicate&lt;T&gt;</td><td style="width: 33.3333%; height: 19px;">boolean test(T t)</td><td style="width: 33.3333%; height: 19px;">Collection::isEmpty</td></tr><tr style="height: 19px;"><td style="width: 33.3333%; height: 19px;">Function&lt;T, R&gt;</td><td style="width: 33.3333%; height: 19px;">R apply(T t)</td><td style="width: 33.3333%; height: 19px;">Arrays::asList</td></tr><tr style="height: 19px;"><td style="width: 33.3333%; height: 19px;">Supplier&lt;T&gt;</td><td style="width: 33.3333%; height: 19px;">T get()</td><td style="width: 33.3333%; height: 19px;">Instant::now</td></tr><tr><td style="width: 33.3333%;">Consumer&lt;T&gt;</td><td style="width: 33.3333%;">void accept(T t)</td><td style="width: 33.3333%;">System.out::println</td></tr></tbody></table>

#### 기본 타입을 위한 표준 함수형 인터페이스

 표준 함수형 인터페이스는 기본 타입인 int, long, double용으로 각 3개씩 변형이 생겨납니다. 그 이름도 표준 인터페이스의 이름 앞에 해당 기본 타입 이름을 붙여 지어졌습니다.

 int를 받는 Predicate는 IntPredicate가 되고 long을 받아 long을 반환하는 BinaryOperator는 LongBinaryOperator가 되는 식입니다.

#### Function 인터페이스의 변형

 Function의 변형은 위의 기본 타입 변형들과 다르게 반환 타입만 매개변수화가 됐습니다. 예를 들어 LongFunction<int\[\]>은 long 인수를 받아 int\[\]을 반환합니다.

 이외에도 Function 인터페이스에는 기본 타입을 반환하는 변형이 총 9개가 더 있습니다.

 입력과 결과 타입이 모두 기본 타입이면 접두어로 SrcToResult를 사용합니다. 예컨대 long을 받아 int을 반환하면 LongToIntFunction이 되는 식입니다(총 6개).

 입력이 객체 참조이고 결과가 int, long, double인 변형 인터페이스들은 입력을 매개변수화하고 접두어로 ToResult를 사용합니다. 예컨대 ToLongFunction<int\[\]>은 int\[\] 인수를 받아 long을 반환합니다(총 3개).

#### 그 외의 표준 함수형 인터페이스 변형

 표준 함수형 인터페이스 중 3개에는 인수를 2개씩 받는 변형이 있습니다. (BiPredicate<T, Y., BiFunction<T, U, R>, BiConsumer<T, U>)

 BiFunction에는 다시 기본 타입을 반환하는 세 변형 ToIntBiFunction<T, U>, ToLongBiFunction<T, U>, ToDoubleBiFunction<T, U>가 존재합니다.

 Consumer에는 객체 참조와 기본 타입 하나, 즉 인수를 2개 받는 변형인 ObjDoubleConsumer<T>, ObjIntConsumer<T>, ObjLongConsumer<T>가 존재합니다.

 BooleanSupplier 인터페이스는 boolean을 반환하도록 한 Supplier의 변형입니다.

#### 표준 함수형 인터페이스 주의사항

 표준 함수형 인터페이스 대부분은 기본 타입만 지원합니다. 그렇다고 **기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용해서는 안됩니다.** 동작은 하지만 "박싱된 기본 타입 대신 기본 타입을 사용하라" 라는 조언을 위배합니다. 특히 계산량이 많을 때는 성능이 느려질 수 있습니다.

#### 전용 함수형 인터페이스를 구현해야 하는 경우

 대부분 상화에서는 직접 작성하는 것보다 표준 함수형 인터페이스를 사용하는 편이 낫습니다. 하지만 표준 인터페이스 중 필요한 용도에 맞는게 없다면 직접 작성해야 합니다.

 아래의 세 가지 항목 중, 하나 이상을 만족한다면 전용 함수형 인터페이스를 구현해야 하는 건 아닌지 고민해야 합니다.

-   자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다.
-   반드시 따라야 하는 규약이 있다.
-   유용한 디폴트 메서드를 제공할 수 있다.

#### @FunctionalInterface

 직접 전용 함수형 인터페이스를 작성할 경우, 항상 @FunctionalInterface 애너테이션을 사용해야 합니다. 이 애너테이션을 사용하는 이유는 @Override를 사용하는 이유와 비슷합니다.

-   해당 클래스의 코드나 설명 문서를 읽을 이에게 그 인터페이스가 람다용으로 설계된 것임을 알려준다.
-   해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다.
-   그 결과 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.

#### 함수형 인터페이스를 API에서 사용할 때의 주의점

 함수형 인터페이스를 API에서 사용할 때, 서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중정의해서는 안됩니다. 클라이언트에게 불필요한 모호함만 안겨줄 뿐이며, 이 모호함으로 인해 실제로 문제가 일어나기도 합니다.

 ExecutorService의 submit 메서드는 Callable<T>를 받는 것과 Runnable을 받는 것을 다중정의 했습니다. 그래서 올바른 메서드를 알려주기 위해 형변환해야 할 때가 빈번하게 생깁니다.

 이런 문제를 피하는 가장 쉬운 방법은 서로 다른 함수형 인터페이스를 같은 위치의 인수로 사용하는 다중정의를 피하는 것입니다.

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
