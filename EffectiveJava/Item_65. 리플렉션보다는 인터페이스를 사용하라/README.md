# Item 65. 리플렉션보다는 인터페이스를 사용하라

 리플렉션 기능(java.lang.reflect)을 이용하면 프로그램에서 임의의 클래스에 접근할 수 있습니다. Class 객체가 주어지면 그 클래스의 생성자, 메서드, 필드에 해당하는 Constructor, Method, Field 인스턴스를 가져올 수 있고, 이 인스턴스들로는 그 클래스의 멤버 이름, 필드 타입, 메서드 시그니처 등을 가져올 수 있습니다.

 나아가 Constructor, Method, Field 인스턴스를 이용해 각각에 연결된 실제 생성자, 메서드, 필드를 조작할 수도 있습니다. 이 인스턴스들을 통해 해당 클래스의 인스턴스를 생성하거나, 메서드를 호출하거나, 필드에 접근할 수 있다는 뜻입니다.

#### 리플렉션의 단점

-   **컴파일타임 타입 검사가 주는 이점을 하나도 누릴 수 없습니다.** 예외 검사도 마찬가지입니다. 프로그램이 리플렉션 기능을 사용하여 존재하지 않는 혹은 접근할 수 없는 메서드를 호출하려 시도하면 (주의해서 대비 코드를 작성해두지 않았다면) 런타임 오류가 발생합니다.
-   **리플렉션을 이용하면 코드가 지저분해지고 장황해집니다.**
-   **성능이 떨어집니다.** 리플렉션을 통한 메서드 호출은 일반 메서드 호출보다 훨씬 느립니다.

#### 리플렉션을 사용하는 예시

** 리플렉션은 아주 제한된 형태로만 사용해야 그 단점을 피하고 이점만 취할 수 있습니다.** 컴파일타임에 이용할 수 없는 클래스를 사용해야만 하는 프로그램은 비록 컴파일타임이라도 적절한 인터페이스나 상위 클래스를 이용할 수는 있을 것입니다(아이템 64).

 **리플렉션은 인스턴스를 생성에만 쓰고, 이렇게 만든 인스턴스는 인터페이스나 상위 클래스로만 참조해 사용하는 것이 좋습니다.**

**리플렉션으로 생성하고 인터페이스로 참조해 활용하는 코드**

```
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;
import java.util.Arrays;
import java.util.Set;

public class ReflectionCreation {
    private static final String hashSet = "java.util.HashSet";
    private static final String treeSet = "java.util.TreeSet";
    private static String[] strings = {
            "apple",
            "orange",
            "peach",
            "apple",
            "banana",
            "pear",
            "grape"
    };

    public static void main(String[] args) {
        // 클래스 이름을 Class 객체로 변환
        Class<? extends Set<String>> cl = null;
        try {
            cl = (Class<? extends Set<String>>) // 비검사 형변환
                    Class.forName(treeSet);
        } catch (ClassNotFoundException e) {
            fatalError("클래스를 찾을 수 없습니다.");
        }

        // 생성자를 얻는다.
        Constructor<? extends Set<String>> cons = null;
        try {
            cons = cl.getDeclaredConstructor();
        } catch (NoSuchMethodException e) {
            fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
        }

        // 집합의 인스턴스를 만든다.
        Set<String> s = null;
        try {
            s = cons.newInstance();
        } catch (IllegalAccessException e) {
            fatalError("생성자에 접근할 수 없습니다.");
        } catch (InstantiationException e) {
            fatalError("클래스를 인스턴스화 할 수 없습니다.");
        } catch (InvocationTargetException e) {
            fatalError("생성자가 예외를 던졌습니다." + e.getCause());
        } catch (ClassCastException e) {
            fatalError("Set을 구현하지 않은 클래스입니다.");
        }

        // 생성한 집합을 사용한다.
        s.addAll(Arrays.asList(strings));
        System.out.println(s);
    }

    private static void fatalError(String msg) {
        System.err.println(msg);
        System.exit(1);
    }
}

```

 위의 예시는 리플렉션의 단점 두 가지를 보여줍니다.

1.  런타임에 총 여섯 가지나 되는 예외를 던질 수 있습니다.
2.  클래스 이름만으로 인스턴스를 생성해내기 위해 많은 코드를 작성해야 합니다. (리플렉션 예외 각각을 잡는 대신 모든 리플렉션 예외의 상위 클래스인 ReflectiveOperationException을 잡도록 하여 코드 길이를 줄일 수도 있습니다. ReflectiveOperationException은 자바 7부터 지원됩니다.)

 이 프로그램은 컴파일하면 비검사 형변환 경고가 뜹니다. Class<? extends Set<String>>으로의 형변환은 명시한 클래스가 Set을 구현하지 않았더라도 성공하고, 실제 문제로 이어지지 않습니다. 단, 그 클래스의 인스턴스를 생성하려 할 때 ClassCastException을 던지게 됩니다. 이 경고를 숨기는 방법은 아이템 27을 참고하면 됩니다.

#### 리플렉션이 유용한 경우

 리플렉션은 런타임에 존재하지 않을 수도 있는 다른 클래스, 메서드, 필드와의 의존성을 관리할 때 적합합니다. 이 기법은 버전이 여러 개 존재하는 외부 패키지를 다룰 때 유용합니다. 가동할 수 있는 최소한의 환경, 즉 주로 가장 오래된 버전만을 지원하도록 컴파일 후, 이후 버전의 클래스와 메서드 등은 리플렉션으로 접근하는 방식입니다.

 리플렉션을 사용하는 경우 접근하려는 새로운 클래스나 메서드가 런타임에 존재하지 않을 수 있다는 사실을 반드시 기억해야 합니다. 즉, **같은 목적을 이룰 수 있는 대체 수단을 이용하거나 기능을 줄여 동작하는 등의 적절한 조치를 취해야 합니다.**

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
