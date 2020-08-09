# Item 1. 생성자 대신 정적 팩터리 메서드를 고려하라

#### 클라이언트가 클래스의 인스턴스를 얻는 수단

-   public 생성자를 이용하는 방법
-   **정적 팩토리 메서드(static factory method)**를 이용하는 방법

 boolean 값을 받아 박싱 클래스(boxed class)인 Boolean 객체 참조로 변환해주는 코드

```
public static final Boolean TRUE = new Boolean(true);
public static final Boolean FALSE = new Boolean(false);

public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
}
```

#### 정적 팩토리 메서드의 장점

**1\. 이름을 가질 수 있다.**

 생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못합니다. 반면 정적 팩토리는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있습니다.

 하나의 시그니처로는 생성자를 하나만 만들 수 있습니다. 하지만 정적 팩토리 메서드는 이런 제약 없이 각각의 차이를 잘 드러내는 이름을 지어주면 됩니다.

**2\. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.**

 불변 클래스(Immutable class)는 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있습니다. 따라서 (특히 생성 비용이 큰) 같은 객체가 자주 요청되는 상황이라면 성능을 상당히 끌어올릴 수 있습니다.

**3\. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.**

이 능력은 반환할 객체의 클래스를 자유롭게 선택할 수 있게 합니다. 이 유연성을 응용하면 구체적인 구현 클래스를 공개하지 않고 그 객체를 반환할 수 있어 API를 작게 유지할 수 있습니다.

**4\. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.**

반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환할 수 있습니다.

**5\. 정적 팩토리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.**

 구체적인 클래스 객체를 반환하지 않고, 인터페이스를 반환할 수 있습니다. 이런 유연함은 서비스 제공자 프레임 워크를 만드는 근간이 됩니다. 대표적인 서비스 제공자 프레임워크로는 JDBC(Java Database Connectivity)가 있습니다.

#### 서비스 제공자 프레임워크 구성요소

-   서비스 인터페이스 (Service Interface) : 구현체의 동작을 정의
-   제공자 등록 API (Provider Registration API) : 제공자가 구현체를 등록
-   서비스 접근 API (Service Access API) : 클라이언트가 서비스의 인스턴스를 얻을 때 사용

 위 세 가지 컴포넌트와 더불어 종종 **서비스 제공자 인터페이스(Service Provider Interface)**라는 네 번째 컴포넌트가 쓰이기도 합니다. 이 컴포넌트는 서비스 인터페이스의 인스턴스를 생성하는 팩토리 객체를 설명해줍니다.

#### 정적 팩토리 메서드의 단점

**1\. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩토리 메서드만 제공하면 하위 클래스를 만들 수 없다.**

이 제약은 상속보다 컴포지션을 사용하도록 유도하고 불변 타입으로 만들려면 이 제약을 지켜야 한다는 점에서 오히려 장점으로 받아들일 수도 있습니다.

**2\. 정적 팩토리 메서드는 프로그래머가 찾기 어렵다.**

생성자처럼 API 설명에 명확히 드러나지 않으니 사용자는 정적 팩토리 메서드 방식 클래스를 인스턴스화할 방법을 알아내야 합니다. 아래는 이를 위해 잘 알려진 명명 방식으로 사용되는 메서드 이름입니다.

-   from : 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
-   of : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 메서드
-   valueOf : from과 of의 더 자세한 버전
-   instance 혹은 getInstance : (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반화하지만, 같은 인스턴스임을 보장하지 않음
-   create 혹은 newInstance : instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장
-   getType : getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩토리 메서드를 정의할 때 쓴다.
-   newType : newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩토리 메서드를 정의할 때 쓴다.
-   type : getType과 newType의 간결한 버전

---

## 참고자료

[http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)

[https://devyongsik.tistory.com/294](https://devyongsik.tistory.com/294)

[https://itnext.io/java-service-provider-interface-understanding-it-via-code-30e1dd45a091](https://itnext.io/java-service-provider-interface-understanding-it-via-code-30e1dd45a091)
