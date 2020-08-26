# Item 14. Comparable을 구현할지 고려하라

#### Comparable

 Comparable 인터페이스는 compareTo() 메서드를 가지고 있고, 이를 구현한 클래스는 그 인스턴스들에 자연적인 순서가 있음을 뜻합니다. Comparable을 구현한 객체들의 배열은 다음과 같이 간단하게 정렬할 수 있습니다.

```
Arrays.sort(a);
```

 알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 Comparable 인터페이스를 구현하는 것이 좋습니다.

#### compareTo() 메서드의 일반 규약

 이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환합니다. 이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다.

 다음 설명에서 sgn(표현식) 표기는 수학에서 말하는 부호 함수(signum function)를 뜻하며, 표현식의 값이 음수, 0, 양수일 때 -1, 0, 1을 반환하도록 정의했다.

-   Comparable을 구현한 클래스는 모든 x, y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다(따라서 x.compareTo(y)는 y.compareTo(x)가 예외를 던질 때에 한해 예외를 던져야 한다).
-   Comparable을 구현한 클래스는 추이성을 보장해야 한다. 즉, (x.compareTo(y) > 0 && y.compareTo(z) > 0)이면 x.compareTo(z) > 0이다.
-   Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z)) > 0이다.
-   이번 권고가 필수는 아니지만 꼭 지키는 것이 좋다. (x.compareTo(y) == 0) == (x.equals(y))여야 한다. Comparable을 구현하고 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다. 다음과 같이 명시하면 적당할 것이다. "주의 : 이 클래스의 순서는 equals 메서드와 일관되지 않다."

 위 일반 규약은 모든 객체에 대해 전역 동치 관계를 부여하는 equals() 메서드와 달리, compareTo()는 타입이 다른 객체는 신경쓰지 않아도 됩니다. 즉, 타입이 다른 객체가 주어지면 간단히 ClassCastException을 던져도 되며, 대부분 그렇게 합니다.

 위의 첫 번째부터 세 번째 규약은 compareTo() 메서드로 수행하는 동치성 검사도 equals 규약과 똑같이 반사성, 대칭성, 추이성을 충족해야 함을 뜻합니다.

 마지막 규약은 필수는 아니지만 꼭 지키는 것이 좋습니다. 마지막 규약은 compareTo() 메서드로 수행한 동치성 테스트의 결과가 equals()와 같아야 한다는 말입니다.

#### 객체 참조 필드 비교 방법

 compareTo() 메서드는 각 필드의 순서를 비교합니다. 객체 참조 필드를 비교하려면 compareTo() 메서드를 재귀적으로 호출합니다. Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자(Comparator)를 대신 사용합니다.

**객체 참조 필드가 하나뿐인 비교자**

```
import java.util.Objects;

public class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    @Override
    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    }
}

```

 예전에는 compareTo() 메서드에서 정수 기본 타입 필드를 비교할 때는 관계 연산자인 <와 >를, 실수 기본 타입 필드를 비교할 때는 정적 메서드인 Double.compare, Float,compare을 사용하는 것이 권장되었습니다. 하지만 자바 7부터는 **박싱된 기본 타입 클래스들에 새로 추가된 정적 메서드인 compare를 이용할 수 있습니다.**

**클래스에 핵심 필드가 여러 개인 경우**

```
public class Foo implements Comparable<Foo> {
    private int first;
    private int second;
    private int third;

    public Foo(int first, int second, int third) {
        this.first = first;
        this.second = second;
        this.third = third;
    }

    @Override
    public int compareTo(Foo o) {
        int result = Integer.compare(first, o.first);
        if (result == 0) {
            result = Integer.compare(second, o.second);

            if (result == 0) {
                result = Integer.compare(third, o.third);
            }
        }

        return result;
    }
}

```

 핵심 필드가 여러 개라면 가장 핵심적인 필드부터 비교해나가는 것이 좋습니다. 비교 결과가 0이 아니라면 즉시 순서가 결정되면서 그 결과를 반환하면 됩니다.

#### Comparator

 자바 8에서는 Comparator 인터페이스에서 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었습니다. 이 비교자들을 Comparable 인터페이스가 요구하는 compareTo() 메서드를 구현하는데 활용할 수 있습니다. 이 방식은 간결함이라는 장점이 있지만, 약간의 성능 저하가 뒤따릅니다.

```
import java.util.Comparator;

import static java.util.Comparator.*;

public class Foo implements Comparable<Foo> {
    private int first;
    private int second;
    private int third;

    private static final Comparator<Foo> COMPARATOR = 
            comparingInt((Foo foo) -> foo.first)
                    .thenComparingInt(foo -> foo.second)
                    .thenComparingInt(foo -> foo.third);

    public Foo(int first, int second, int third) {
        this.first = first;
        this.second = second;
        this.third = third;
    }

    @Override
    public int compareTo(Foo o) {
        return COMPARATOR.compare(this, o);
    }
}

```

 위 코드에서 비교자의 comparingInt()의 람다에서 (Foo foo)를 명시한 이유는 자바의 타입 추론 능력이 이 상황에서 타입을 알아낼 만큼 강력하지 않기 때문입니다. 그 이후에 thenComparingInt()를 호출할 때는 타입을 명시하지 않았는데, 여기서는 자바의 타입 추론이 가능하기 때문입니다.

 Comparator는 수많은 보조 생성 메서드들이 갖춰져 있습니다. long과 double용으로는 comparingLong()과 comparingDouble()이 준비되어 있습니다. short와 같이 더 작은 정수 타입에는 int용 버전을 사용하면 됩니다.

#### 비교자의 주의 사항

```
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();
    }
};
```

 위의 방식은 사용해서는 안됩니다. 이 방식은 정수의 오버플로우를 일으키거나 부동소수점 계산 방식에 따른 오류를 낼 수 있기 때문입니다. 그 대신 다음의 두 방식 중 하나를 사용하는 것이 좋습니다.

**정적 compare 메서드를 활용한 비교자**

```
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
};
```

**비교자 생성 메서드를 활용한 비교자**

```
static Comparator<Object> hashCodeOrder =
    Comparator.comparingInt(o -> o.hashCode());
```

---

## 참고자료

[http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
