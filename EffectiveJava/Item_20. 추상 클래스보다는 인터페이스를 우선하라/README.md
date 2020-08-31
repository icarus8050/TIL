# Item 20. 추상 클래스보다는 인터페이스를 우선하라

#### 자바가 제공하는 다중 구현 메커니즘

 자바에서는 다중 구현 메커니즘은 **인터페이스와 추상 클래스**, 이렇게 두 가지입니다.

 자바 8부터는 인터페이스도 디폴트 메서드(default method)를 제공할 수 있게 되었습니다.

```
public interface Hello {
    default void say() {
        System.out.println("Hello World");
    }
}

```

 이로써 인터페이스와 추상 클래스는 모드 인스턴스 메서드를 구현 형태로 제공이 가능합니다.

#### 인터페이스와 추상 클래스의 차이

 두 메커니즘의 차이는 추상 클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다는 점입니다. 자바는 단일 상속만 지원하니, 추상 클래스 방식은 새로운 타입을 정의하는 데 커다란 제약이 생깁니다. 반면 인터페이스가 선언한 메서드를 모두 정의하고 그 일반 규약을 잘 지킨 클래스라면 다른 어떤 클래스를 상속했든 같은 타입으로 취급됩니다.

#### 믹스인 (mixin)

 인터페이스는 믹스인(mixin) 정의에 안성맞춤입니다. 믹스인이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 줄 수 있습니다. 예를 들어, Comparable은 자신을 구현한  클래스의 인스턴스들끼리는 순서를 정할 수 있다고 선언하는 믹스인 인터페이스입니다. 이처럼 **대상 타입의 주된 기능에 선택적 기능을 '혼합(mixed-in)'한다고 해서 믹스인이라고 부릅니다. **

#### 인터페이스의 유연성

 인터페이스는 계층구조가 없는 타입 프레임워크를 만들 수 있습니다. 타입을 계층적으로 정의하면 수많은 개념을 구조적으로 잘 표현할 수 있지만, 현실에는 계층을 엄격히 구분하기 어려운 개념도 있습니다.

```
public interface Singer {
    AudioClip sing(Song s);
}
```

```
public interface Songwriter {
    Song compose(int chartPosition);
}

```

 위와 같이 가수(Singer)와 작곡가(Songwriter)의 역할을 하는 인터페이스가 있습니다. 하지만 현실에서는 작곡을 함께하는 가수도 있습니다. 이러한 개념은 아래와 같이 쉽게 구현이 가능합니다.

```
public interface SingerSongwriter extends Singer, Songwriter {
    AudioClip strum();
    void actSensitive();
}

```

 작곡을 겸하는 가수를 확장한 타입은 위처럼 Singer와 Songwriter를 모두 확장하고, 새로운 메서드까지 추가한 제3의 인터페이스를 정의할 수도 있습니다. 만약 같은 구조를 클래스로 만들려면 가능한 조합 전부를 각각의 클래스로 정의해야 하므로 엄청나게 거대한 계층구조가 만들어질 것입니다.

#### 인터페이스의 디폴드 메서드

 인터페이스의 메서드 중 구현 방법이 명백한 것이 있다면, 그 구현을 디폴트 메서드로 제공하여 프로그래머들의 구현 부담을 덜어줄 수 있습니다. 디폴트 메서드를 제공할 때는 상속하려는 사람을 위한 설명을 @implSpec 자바독 태그를 붙여 문서화해야 합니다.

 디폴트 메서드는 equals()와 hashCode() 같은 Object의 메서드를 제공해서는 안 됩니다. 또한 **인터페이스는 인스턴스 필드를 가질 수 없고 public이 아닌 정적 멤버로 가질 수 없습니다(단, private 정적 메서드는 예외).**

#### 추상 골격 구현(skeletal implementation) 클래스

 인터페이스로는 타입을 정의하고, 필요하다면 디폴트 메서드 몇 개도 함께 제공합니다. 그리고 골격 구현 클래스는 나머지 메서드들까지 구현하는 방법을 사용하면 단순히 골격 구현을 확장하는 것만으로 이 인터페이스를 구현하는 데 필요한 일이 대부분 완료됩니다. 바로 **템플릿 메서드 패턴**입니다.

 관례상 인터페이스 이름이 Interface라면 그 골격 구현 클래스의 이름은 AbstractInterface로 짓는 것이 좋습니다. 대표적인 예로, 컬렉션 프레임워크의 AbstractCollection, AbstractSet, AbstractList, AbstractMap 각각이 바로 핵심 컬렉션 인터페이스의 골격 구현입니다.

**골격 구현을 사용해 완성한 구체 클래스**

```
public class Foo {

    static List<Integer> intArrayAsList(int[] a) {
        Objects.requireNonNull(a);

        return new AbstractList<>() {
            @Override
            public Integer get(int i) {
                return a[i];
            }

            @Override
            public Integer set(int i, Integer val) {
                int oldVal = a[i];
                a[i] = val;
                return oldVal;
            }

            @Override
            public int size() {
                return a.length;
            }
        };
    }
}

```

---

## 참고자료

[http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
