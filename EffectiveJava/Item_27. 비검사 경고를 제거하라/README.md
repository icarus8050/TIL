# Item 27. 비검사 경고를 제거하라

 제네릭을 사용하기 시작하면 수많은 컴파일러 경고를 보게 될 것입니다. 비검사 형변환 경고, 비검사 메서드 호출 경고, 비검사 매개변수화 가변인수 타입 경고, 비검사 변환 경고 등입니다.

** 비검사 경고는 할 수 있는 한 모두 제거해야 합니다.** 모두 제거한다면 그 코드는 타입 안전성이 보장됩니다. 즉, 런타임에 ClassCastException이 발생할 일이 없고, 의도한 대로 잘 동작하리라 확신할 수 있습니다.

 **경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있다면 @SuppressWarnings("unchecked") 애너테이션을 달아 경고를 숨기는 것이 좋습니다.**

 타입 안전함을 검증하지 않은 채 경고를 숨긴다면, 그 코드는 경고 없이 컴파일되겠지만, 런타임에는 여전히 ClassCastException을 던질 수 있습니다.

 안전하다고 검증된 비검사 경고를 숨기지 않고 그대로 두면, 진짜 문제를 알리는 새로운 경고가 나와도 눈치채지 못할 수 있습니다. 제거하지 않은 수많은 거짓 경고 속에 새로운 경고가 파묻힐 것이기 때문입니다.

#### @SuppressWarnings

\- 선언 범위

 개별 지역변수 선언부터 클래스 전체까지의 모든 선언부

** @SuppressWarnings 애너테이션은 항상 가능한 한 좁은 범위에 적용하는 것이 좋습니다.**

 한 줄이 넘는 메서드나 생성자에 달린 @SuppressWarnings 애너테이션을 발견하면 지역변수 선언 쪽으로 옮기는 것이 좋습니다. 이를 위해 지역변수를 새로 선언하는 수고를 해야 할 수도 있지만, 그만한 값어치가 있습니다.

```
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        return (T[]) Arrays.copyOf(elements, size, a.getClass());
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}

```

 위의 코드는 ArrayList의 toArray() 메서드 입니다. ArrayList를 컴파일하면 unchecked cast 경고가 발생합니다. 애너테이션은 선언에만 달 수 있기 때문에 return 문에는 @SuppressWarnings를 다는 것이 불가능합니다. 메서드 전체에 달 수도 있지만 범위가 필요 이상으로 넓어집니다. 반환값을 담을 지역변수를 선언하고 그 변수에 애너테이션을 달아주는 것이 좋습니다.

```
public <T> T[] toArray(T[] a) {
    if (a.length < size) {
        // 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로
        // 올바른 형변환이다.
        @SuppressWarnings("unchecked") T[] result =
            (T[]) Arrays.copyOf(elements, size, a.getClass());
        return result;
    }
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}

```

** @SuppressWarnings("unchecked") 애너테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 합니다.**

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
