# Item 22. 인터페이스는 타입을 정의하는 용도로만 사용하라

#### 인터페이스의 용도

 인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 **타입 역할**을 합니다. 즉, 클**래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지**를 클라이언트에 이야기 해주는 것입니다. 인터페이스는 오직 이 용도로만 사용해야 합니다.

#### 안티패턴

상수 인터페이스 - 안티패턴

```
public interface AntiPhysicalConstants {
    // 아보가드로 수 (1/몰)
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    
    // 볼츠만 상수 (J/K)
    static final double boltzmann_constant = 1.380_648_52e-23;
    
    // 전자 질량 (kg)
    static final double ELECTRON_MASS = 9.109_383_56e-31;
}

```

 위와 같이 메서드 없이, 상수를 뜻하는 static final 필드로만 가득 찬 인터페이스의 형태는 상수 인터페이스로 불리며, 인터페이스의 안티패턴입니다.

#### 상수 인터페이스의 문제점

 **클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라 내부 구현에 해당합니다. 따라서 상수 인터페이스를 구현하는 것은 이 내부 구현을 클래스의 API로 노출하는 행위**입니다. 이는 클라이언트 코드가 내부 구현에 해당하는 이 상수들에 종속되게 합니다. 만약 final이 아닌 클래스가 상수 인터페이스를 구현한다면 모든 하위 클래스의 아름 공간이 그 인터페이스가 정의한 상수들로 오염되어 버립니다.

#### 해결 방법

 특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가해야 합니다. 모든 숫자 기본 타입의 박싱 클래스가 대표적으로, Integer와 Double에 선언된 MIN\_VALUE와 MAX\_VALUE 상수가 이런 예입니다. 열거 타입으로 나타내기 적합한 상수라면 열거 타입으로 만들어 공개하면 됩니다. 그것도 아니라면, **인스턴스화할 수 없는 유틸리티 클래스에 담아 공개**하면 됩니다.

```
public class PhysicalConstants {
    
    private PhysicalConstants() { } // 인스턴스화 방지
    
    // 아보가드로 수 (1/몰)
    public static final double AVOGADROS_NUMBER = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    public static final double boltzmann_constant = 1.380_648_52e-23;

    // 전자 질량 (kg)
    public static final double ELECTRON_MASS = 9.109_383_56e-31;
}

```

 유틸리티 클래스에 정의된 상수를 클라이언트에서 사용하려면 클래스 이름까지 함께 명시해야 합니다. 만약 유틸리티 클래스의 상수를 빈번하게 사용한다면 정적 임포트(static import)하여 클래스 이름은 생략할 수 있습니다.

```
import static effective_java.item_22.PhysicalConstants.*;

public class Test {
    
    double atoms(double mols) {
        return AVOGADROS_NUMBER * mols;
    }
}

```

---

## 참고자료

[http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
