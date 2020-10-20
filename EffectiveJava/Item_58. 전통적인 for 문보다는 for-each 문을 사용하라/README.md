# Item 58. 전통적인 for 문보다는 for-each 문을 사용하라

**for 문으로 컬렉션 순회하기**

```
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    //Something job..
}
```

**for 문으로 배열 순회하기**

```
for (int i = 0; i < a.length; i++) {
    //Something job...
}
```

 위에 작성된 for 문들은 반복자와 인덱스 변수들로 인해 코드를 지저분하게 만듭니다. 잘못된 변수를 사용 했을 때 컴파일러가 잡아주리라는 보장도 없습니다. 이 문제들은 for-each 문을 사용하면 간단하게 해결됩니다.

**컬렉션과 배열을 순회하는 for-each 문**

```
for (Element e : elements) {
    //Something job...
}
```

#### 컬렉션을 중첩해 순회하는 경우 for-each 문의 이점

**버그가 숨어있는 코드**

```
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING }

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
        deck.add(new Card(i.next(), j.next()));
```

 위 코드는 for 문의 반복자들에 의해 코드가 지저분한 상태입니다. 여기서 가장 마지막 줄의 i.next()를 호출하는 부분과 같은 실수를 범할 수 있습니다.

**컬렉션이나 배열의 중첩 반복을 위한 for-each 문**

```
for (Suit suit : suits)
    for (Rank rank : ranks)
        deck.add(new Card(suit, rank));
```

 for-each 문을 통해서 코드가 상당히 간결해진 것을 확인할 수 있습니다.

#### for-each 문을 사용할 수 없는 상황

**파괴적인 필터링 (destructive filtering)**

 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야 합니다. 자바 8부터는 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있습니다.

**변형 (transforming)**

 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 합니다.

**병렬 반복 (parallel iteration)**

 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 합니다.

#### Iterable 인터페이스

 for-each 문은 컬렉션과 배열은 물론 Iterable 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있습니다.

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
