# Item 37. 비트 필드 대신 EnumSet을 사용하라

 열거한 값들이 주로 (단독이 아닌) 집합으로 사용될 경우, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해 왔습니다.

**비트 필드 열거 상수**

```
public class Text {
    public static final int STYLE_BOLD = 1 << 0; // 1
    public static final int STYLE_ITALIC = 1 << 1; //2
    public static final int STYLE_UNDERLINE = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8
    
    // 매개변수 style는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
    public void applyStyles(int styles) { ... }
}

```

 다음과 같은 식으로 비트별 OR를 사용해 여러 상수를 하나의 집합으로 모을 수 있으며, 이렇게 만들어진 집합을 비드 필드(bit field)라 합니다.

```
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

 비트 필드를 사용하면 비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행할 수 있습니다.

#### 비트 필드 열거 상수의 단점

 비트 필드는 정수 열거 상수의 단점을 그대로 지니고 있습니다.

 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기가 훨씬 어렵습니다. 비트 필드 하나에 녹아 있는 모든 원소를 순회하기도 까다롭습니다.

 최대 몇 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입(보통은 int나 long)을 선택해야 합니다. API를 수정하지 않고는 비트 수(32비트 or 64비트)를 더 늘릴 수 없기 때문입니다.

#### EnumSet 클래스

 EnumSet 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해줍니다. Set 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 어떤 Set 구현체와도 함께 사용할 수 있습니다.

 EnumSet의 내부는 비트 벡터로 구현되어 있습니다. 원소가 총 64개 이하라면, 대부분의 경우에 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여줍니다.

 removeAll과 retainAll 같은 대량 작업은 (비트 필드를 사용할 때 쓰는 것과 같은) 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현되었습니다.

```
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

    // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
    public void applyStyles(Set<Style>styles) {
        // Something to do..
    }
}

```

```
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
