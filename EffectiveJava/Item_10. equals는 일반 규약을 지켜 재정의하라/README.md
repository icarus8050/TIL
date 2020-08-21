# Item 10. equals는 일반 규약을 지켜 재정의하라

## equals()를 재정의 하지 않는 것이 최선인 경우

** 각 인스턴스가 본질적으로 고유한 경우.** 값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스가 여기 해당합니다. Thread가 좋은 예로, Object의 equals() 메서드는 이러한 클래스에 딱 맞게 구현되어 있습니다.

** 인스턴스의 논리적 동치성(logical equality)을 검사할 일이 없는 경우.**

** 상위 클래스에서 재정의한 equals()가 하위 클래스에도 딱 들어 맞는 경우.**

** 클래스가 private이거나 package-private이고 equals() 메서드를 호출할 일이 없는 경우.** equals() 메서드가 실수로라도 호출되는 것을 막고자 한다면 아래와 같이 구현해두면 됩니다.

```
@Override
public boolean equals(Object o) {
    throw new AssertionError();	//호출 금지!
}
```

## equals()를 재정의해야 하는 경우

 객체의 식별성(Object identity; 두 객체가 물리적으로 같은가)이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals()가 논리적 동치성을 비교하도록 재정의되지 않았을 때입니다. 주로 값 클래스들이 여기에 해당됩니다. 값 클래스란 Integer와 String처럼 값을 표현하는 클래스를 말합니다. 두 객체가 같은지가 아니라 값이 같은지를 비교하여 논리적 동치성을 만족시키면 Map의 키와 Set의 원소로 사용할 수 있게 됩니다.

## equals()를 재정의할 때 지켜야할 일반 규약

#### 반사성(reflexivity) : null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.

 반사성은 객체는 자기 자신과 같아야 한다는 단순한 규칙입니다.

#### 대칭성(symmetry) : null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true다.

 두 객체는 서로에 대한 동치 여부에 똑같이 반환되어야 한다는 규칙입니다.

```
import java.util.Objects;

public class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
                    
        if (o instanceof String)
            return s.equalsIgnoreCase((String) o);
            
        return false;
    }
}

```

 CaseInsensitiveString 클래스의 equals()는 String 클래스의 문자열과도 비교를 할 수 있습니다. CaseInsensitiveString 클래스의 equals() 에서는 대소문자를 구별하지 않고 문자열 비교 연산을 합니다.

```
CaseInsensitiveString cis = new CaseIncensitiveString("Polish");
String s = "polish";
```

 위와 같이 두 객체가 생성되어 있고, equals() 메서드로 비교 연산을 하면 cis.equals(s)는 true를 반환할 것입니다. 하지만 String의 equals()는 CaseInsensitiveString 클래스의 존재에 대해 알지 못하므로 s.equals(cis)는 false를 반환하여 대칭성을 만족하지 못합니다. 이 문제를 해결하기 위해서는 CaseInsensitiveString 클래스의 equals()를 String과 연동하는 것은 포기하고 아래와 같이 모습이 바뀌어야 합니다.

```
    @Override
    public boolean equals(Object o) {
        return o instanceof CaseInsensitiveString &&
                ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
    }
```

#### 추이성(transitivity) : null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true다.

```
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}

```

```
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
}

```

 두 객체는 서로 **대칭성**을 만족하지 못합니다. Point 객체를 ColorPoint 객체에 비교한 결과와 그 둘을 바꿔 비교한 결과가 다를 수 있습니다. Point의 equals()는 색상을 무시하고, ColorPoint의 equals()는 입력 매개변수의 클래스 종류가 다르다며 매번 false를 반환할 것이기 때문입니다.

```
Point p = new Point(1, 2);
ColorPoint cp = new ColorPoint(1, 2, Color.RED);
```

 p.equals(cp)는 true를 반환하고, cp.equals(p)는 false를 반환합니다. 만약 아래와 같이 ColorPoint.equals()가 Point와 비교할 때는 색상을 무시하도록 하면 **추이성**이 깨져버리게 됩니다.

```
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        
        //o가 일반 Point면 색상을 무시하고 비교
        if (!(o instanceof ColorPoint))
            return o.equals(this);
        
        //o가 ColorPoint면 색상까지 비교
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
```

```
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
```

p1.equals(p2)는 true, p2.equals(p3)는 true, p1.equals(p3)는 false를 반환합니다. p1과 p2, p2와 p3는 색상을 무시했지만, p1과 p3 비교에서는 색상까지 고려했기 때문입니다. **구체 클래스를 확장해 새로운 값을 추가하면서 equals() 규약을 만족시킬 방법은 존재하지 않습니다. **하지만 상속 대신 컴포지션을 활용하면 우회하여 문제를 해결할 수 있습니다.

```
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    public Point asPoint() {
        return point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
}

```

 자바 라이브러리에도 구체 클래스를 확장해 값을 추가한 클래스가 종종 있습니다. 한 가지 예로 java.sql.Timestamp가 java.util.Date를 확장한 후 nanoseconds 필드를 추가하였습니다. 그 결과로 Timestamp의 equals()는 대칭성을 위배하여, Date 객체와 한 컬렉션에 넣거나 서로 섞어 사용하면 엉뚱하게 동작할 수 있습니다.

#### 일관성(consistency) : null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.

 두 객체가 같다면 (어느 하나 혹은 두 객체가 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 한다는 뜻입니다. 가변 객체는 비교 시점에 따라 서로 다르거나 같을 수도 있는 반면, 불변 클래스는 한 번 다르면 끝까지 달라야 합니다.

** 클래스가 불변이든 가변이든 eqausl()의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안됩니다. **equals()는 항시 메모리에 존재하는 객체만을 사용한 결정적(deterministic) 계산만 수행해야 합니다.

#### non-null : null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.

 모든 객체가 null가 같지 않아야 한다는 뜻입니다. 하지만 equals()에서 비교 객체가 null인지 검사할 필요는 없습니다. 동치성을 검사하기 위해서는 건네받은 객체를 적절한 타입으로 형변환 해야하는데, 형변환에 앞서 instanceof 연산자를 통해 올바른 타입인지 검사해야 합니다. **instanceof 연산자는 두 번째 피연산자와 무관하게 첫 번째 피연산자가 null이면 false를 반환합니다.** 따라서 null 검사를 명시적으로 할 필요가 없는 것입니다.

## Equals() 메서드를 구현하는 방법 정리

-   **\== 연산자를 사용해 입력이 자기 자신의 참조인지 확인합니다.** 이는 자기 자신이면 true를 반환하여 단순하게 성능을 최적화하기 위한 용도입니다. 비교 작업이 복잡한 상황에서 유용합니다.
-   **instanceof 연산자로 입력이 올바른 타입인지 확인합니다. **그렇지 않다면 false를 반환합니다. 올바른 타입은 equals()가 정의된 클래스인 것이 보통이지만, 가끔은 그 클래스가 구현한 특정 인터페이스가 될 수도 있습니다. 어떤 인터페이스는 자신을 구현한 (서로 다른) 클래스끼리도 비교할 수 있도록 equals() 규약을 수정하기도 합니다. 이런 인터페이스를 구현한 클래스라면 equals()에서 해당 클래스가 아닌 해당 인터페이스를 사용해야 합니다. 대표적인 예로 Set, List, Map, Map.Entry 등의 컬렉션 인터페이스들이 있습니다.
-   **입력을 올바른 타입으로 형변환합니다.**
-   **입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사합니다.** 2단계에서 인터페이스를 사용했다면 입력 필드 값을 가져올 때도 그 인터페이스의 메서드를 사용해야 합니다.

 float과 double을 제외한 기본 타입은 == 연산자로 비교하고, 참조 타입 필드는 각각의 equals()로, **float와 double 필드는 각각 정적인 메서드인 Float.compare(float, float)와 Double.compare(double, double)로 비교**합니다. float과 double을 특별 취급하는 이유는 Float.NoN, -0.0f, 특수한 부동소수 값 등을 다뤄야 하기 때문입니다. Float.equals()와 Double.equals() 메서드를 대신 사용할 수도 있지만, 이 메서드들은 오토박싱을 수반할 수 있으니 성능상 좋지 않습니다.

---

## 참고자료

[http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
