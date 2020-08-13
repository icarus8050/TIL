# Item 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

#### private 생성자의 사용

-   java.lang.Math와 java.util.Arrays 처럼 기본 타입 값이나 배열 관련 메서드를 모아놓을 수 있습니다.
-   java.util.Collections 처럼 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드(혹은 팩토리)를 모아놓을 수도 있습니다. (자바 8부터는 이런 메서드를 인터페이스에 넣을 수 있습니다.)
-   final 클래스와 관련한 메서드들을 모아놓을 때 사용합니다.

 이러한 정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계하는게 아닙니다. 하지만 생성자를 명시하지 않으면 컴파일러가 자동으로 디폴트 생성자를 만들게 됩니다. 이러한 사실은 사용자에게는 생성자 자동 생성된 것인지 구별할 수 없게 합니다.

 인스턴스화를 막기 위해서 추상 클래스를 생각할 수 있지만 **추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없습니다.** 추상 클래스는 하위 클래스를 만들어 인스턴스화 할 수 있기 때문입니다. 이를 본 사용자는 상속하여 쓰라는 뜻으로 오해할 수 있습니다.

#### 인스턴스 생성을 막는 방법

 인스턴스 생성을 막는 방법은 **private 생성자를 추가**하면 클래스의 인스턴스화를 쉽게 막을 수 있습니다. 명시적 생성자가 private 이니 클래스 바깥에서는 접근할 수 없습니다. 클래스 안에서 실수로라도 생성자를 호출하지 않도록 생성자 안에서 Assertion Error 를 던지면 실수를 줄 일 수 있습니다.

```
public class UtilityClass {
	// 기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지용).
    private UtilityClass() {
    	throw new AssertionError();
    }
    
    // 나머지 코드..
}
```

 위 코드는 어떤 환경에서도 클래스가 인스턴스화되는 것을 막아줍니다. 또한 적절한 주석을 통해 생성자가 존재하지만 호출할 수 없다는 점에 직관성을 부여하는 것이 좋습니다. 이 방식은 생성자의 접근 지정자가 private 이므로 상속을 불가능하게 하는 효과도 있습니다.

---

## 참고자료

[http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)

[

이펙티브 자바 3/E - 교보문고

자바 6 출시 직후 출간된 『이펙티브 자바 2판』 이후로 자바는 커다란 변화를 겪었다. 그래서 졸트상에 빛나는 이 책도 자바 언어와 라이브러리의 최신 기능을 십분 활용하도록 내용 전반을 철�

www.kyobobook.co.kr



](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)