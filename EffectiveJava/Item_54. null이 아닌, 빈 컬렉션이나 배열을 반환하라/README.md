# Item 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

```
private final List<Cheese> cheesesInStock = ...;

public List<Cheese> getCheese() {
    return cheesesInStock.isEmpty() ? null
        : new ArrayList<>(cheesesInStock);
}
```

 위 코드는 리스트가 비어있다면 null을 반환하는 코드입니다. 이러한 코드를 사용하는 클라이언트는 이 null 상황을 처리하는 코드를 추가로 작성해야만 합니다.

```
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON))
    System.out.println("Great");
```

 컬렉션이나 배열 같은 컨테이너가 비었을 때 null을 반환하는 메서드를 사용한다면 항상 이와 같은 방어 코드를 넣어줘야 합니다. 클라이언트에서 실수로 방어 코드를 빼먹으면 오류가 발생하게 됩니다. 실제로 객체가 0개일 가능성이 거의 없는 상황에서는 오랜 시간이 지난 뒤에야 오류가 발생하기도 합니다.

#### null이 아닌, 빈 컬렉션이나 배열을 반환하는 것이 나은 이유

 빈 컨테이너를 할당하는 데도 비용이 드니 null을 반환하는 쪽이 낫다는 주장도 있습니다. 하지만 이는 두 가지 면에서 반박할 수 있습니다.

1.  성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 한, 이 정도의 성능 차이는 신경 쓸 수준이 되지 못합니다.
2.  빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있습니다.

 대부분의 상황에서는 아래와 같은 코드로 빈 컬렉션을 반환할 수 있습니다.

```
public List<Cheese> getCheese() {
    return new ArrayList<>(cheesesInStock);
}
```

 가능성은 적지만, 사용 패턴에 따라 빈 컬렉션 할당이 성능을 눈에 띄게 떨어뜨릴 수도 있습니다. 이러한 경우 매번 똑같은 빈 **'불변'** 컬렉션을 반환하면 됩니다. 불변 객체는 자유롭게 공유해도 안전합니다.

 빈 불변 컬렉션을 반환하는 메서드

-   Collections.emptyList()
-   Collections.emptySet()
-   Collections.emptyMap()

**빈 컬렉션을 매번 새로 할당하지 않는 코드**

```
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? Collections.emptyList()
        : new ArrayList<>(cheesesInStock);
}
```

**길이가 0일 수도 있는 배열을 반환하는 코드**

```
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
}
```

 위와 같은 방식이 성능을 떨어뜨릴 것 같다면 길이 0짜리 배열을 미리 선언해두고 매 번 그 배열을 반환하면 됩니다.

```
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheese() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

 단순히 성능을 개선할 목적이라면 toArray에 넘기는 배열을 미리 할당하는 건 추천하지 않습니다. 오히려 성능이 떨어진다는 연구 결과도 있습니다.

**배열을 미리 할당하면 성능이 나빠집니다.**

```
return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
```

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
