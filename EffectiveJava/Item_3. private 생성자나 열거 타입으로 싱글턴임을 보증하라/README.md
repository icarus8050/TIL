#### Singleton이란?

 싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말합니다. 싱글턴의 전형적인 예로는 Stateless한 객체나 설계상 유일해야 하는 시스템 컴포넌트를 예로 들 수 있습니다.

#### Singleton을 만드는 방법

 싱글턴을 만드는 방식에는 두 가지가 있습니다. 두 방식 모두 생성자는 private으로 감춰두고, 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 하나 마련해두어 구현합니다.

**1\. public static final 필드 방식의 싱글턴**

```
public class ElvisOne {
    public static final ElvisOne INSTANCE = new Elvis_1();
    
    private ElvisOne() {}
    
    public void leaveTheBuilding() {
        //Something..
    }
}
```

 private 생성자는 public static final 필드인 ElvisOne.INSTANCE를 초기화할 때 최초 한 번만 호출됩니다. public이나 protected 생성자가 없으므로 ElvisOne 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장됩니다.

 리플렉션 API인 AccessibleObject.setAccessible을 사용하여 private 생성자를 호출할 수 있기 때문에 이를 방지하려면 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지게 해야 합니다.

 이 방식의 장점은 해당 클래스가 싱글턴임이 API에 명백하게 들어난다는 점입니다. public static 필드가 final이니 다른 객체를 참조할 수 없습니다. 또 다른 장점으로는 간결함입니다.

**2\. 정적 팩터리 메서드 방식의 싱글턴**

```
public class ElvisTwo {
    private static final ElvisTwo INSTANCE = new ElvisTwo();
    
    private ElvisTwo() {}
    
    public static ElvisTwo getInstance() {
        return INSTANCE;
    }

    public void leaveTheBuilding() {
        //Something..
    }
}
```

 ElvisTwo.getInstance는 항상 같은 객체의 참조를 반환하므로 제 2의 Elvis 인스턴스가 만들어지지 않도록 할 수 있습니다. (리플렉션을 통한 예외는 똑같이 적용됩니다.)

이 방식의 장점 3가지

-   API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있습니다. 유일한 인스턴스를 반환하던 팩토리 메서드가 호출하는 스레드별로 다른 인스턴스를 넘겨주게 할 수 있습니다.
-   원한다면 정적 팩토리를 제네릭 싱글턴 팩토리로 만들 수 있습니다.
-   정적 팩토리의 메서드 참조를 Supplier로 사용할 수 있습니다. 가령 ElvisTwo::getInstance를 Supplier<ElvisTwo>로 사용하는 식입니다.

#### 싱글턴의 직렬화

 위에서 소개한 두 방법으로 싱글턴을 구현할 때 직렬화를 한다면 주의해야할 점이 있습니다. 이는 직렬화를 구현할 때 입니다. 직렬화를 위해 Serializable을 구현하는 것만으로는 싱글턴이 완벽하게 보장되지 않습니다. 직렬화된 싱글턴 객체가 다시 역직렬화가 될 때 새로운 인스턴스가 생성됩니다. 이를 방지하기 위해서는 readResolve() 메서드를 이용하면 됩니다.

```
import java.io.Serializable;

public class ElvisTwo implements Serializable {
    private static final ElvisTwo INSTANCE = new ElvisTwo();

    private ElvisTwo() {}

    public static ElvisTwo getInstance() {
        return INSTANCE;
    }

    public void leaveTheBuilding() {
        //Something..
    }

    private Object readResolve() {
        // 원래의 ElvisTwo를 반환하고, 가짜 ElvisTwo는 GC에 의해 반환
        return INSTANCE;
    }
}
```

#### 열거 타입 방식의 싱글턴

열거 타입 방식의 싱글턴은 더 간결하고, 추가적인 노력 없이 직렬화 할 수 있습니다. 리플렉션 공격에서도 제 2의 인스턴가 생기는 일을 완벽하게 막아줍니다. 단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없습니다.

```
public enum ElvisThree {
    INSTANCE;

    public void leaveTheBuilding() {
        //Something..
    }
}
```

---

## 참고자료

[http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)