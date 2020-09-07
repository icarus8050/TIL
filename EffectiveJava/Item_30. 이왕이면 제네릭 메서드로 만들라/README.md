# Item 30. 이왕이면 제네릭 메서드로 만들라

#### 제네릭 메서드

 클래스와 마찬가지로, 메서드도 제네릭으로 만들 수 있습니다. 매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제네릭입니다. 예를 들어, Collections의 알고리즘 메서드(binarySearch, sort 등)는 모두 제네릭입니다.

**타입 안전성이 보장되지 않은 메서드**

```
    public static Set union(Set s1, Set s2) {
        Set result = new HashSet(s1);
        result.addAll(s2);

        return result;
    }
```

 위 메서드는 컴파일은 되지만 타입 안전성이 보장되지 않으므로new HashSet(s1) 부분과 result.addAll(s2) 부분에서 경고가 발생합니다. 

** 타입 매개변수들을 선언하는 타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 옵니다.**

**제네릭 메서드를 활용한 메서드**

```
    public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
        Set<E> result = new HashSet<>(s1);
        result.addAll(s2);

        return result;
    }
```

#### 항등함수 예시

 항등함수(identity function)를 담은 클래스를 만드는 예시를 살펴보겠습니다(실제로는 자바 라이브러리의 Function.identity를 사용하면 됩니다).

```
    private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;
    
    @SuppressWarnings("unchecked")
    public static <T> UnaryOperator<T> identityFunction() {
        return (UnaryOperator<T>) IDENTITY_FN;
    }
```

 IDENTITY\_FN을 UnaryOperator<T>로 형변환하면 비검사 형변환 경고가 발생합니다. T가 어떤 타입이든 UnaryOperator<Object>는 UnaryOperator<T>가 아니기 때문입니다. 하지만 항등함수란 입력 값을 수정 없이 그대로 반환하는 특별한 함수이므로, T가 어떤 타입이든 UnaryOperator<T>를 사용해도 타입 안전합니다. 이를 근거로 @SuppressWarnings 애너테이션을 추가하여 컴파일 경고를 없애줍니다.

#### 재귀적 타입 한정(Recursive type bound) 예시

 재튀적 타입 한정이라는 개념을 이용하면 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수도 있습니다. 재귀적 타입 한정은 주로 타입의 자연적 순서를 정하는 Comparable 인터페이스와 함께 쓰입니다.

```
public interface Comparable<T> {
    int compareTo(T o);
}
```

 Comparable 인터페이스의 타입 매개변수 T는 해당 인터페이스를 구현한 타입이 비교할 수 있는 원소의 타입을 정의합니다.

```
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("컬렉션이 비어 있습니다.");
    
    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result Object.requireNonNull(e);
    
    return result;
}
```

 타입 한정인 <E extends Comparable<E>>는 "모든 타입 E는 자신과 비교할 수 있다"라고 해석할 수 있습니다. 위의 코드는 컬렉션에 담긴 원소의 자연적 순서를 기준으로 최댓값을 계산하며, 컴파일 오류나 경고는 발생하지 않습니다.

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEA&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEA&Kc=)
