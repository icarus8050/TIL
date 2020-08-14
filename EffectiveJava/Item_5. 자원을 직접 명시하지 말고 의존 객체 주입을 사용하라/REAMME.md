# Item 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

 의존 객체 주입은 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식입니다. 의존 객체 주입을 이용하면 환경에 따라 다양한 객체를 주입이 가능해짐으로써 사용하는 자원에 따라 동작이 달라지도록 클래스를 유연하게 설계할 수 있습니다.

```
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon lexicon) {
        this.dictionary = Objects.requireNonNull(lexicon);
    }

    public boolean isValid(String word) {
        //something..
        return true;
    }

    public List<String> suggestions(String typo) {
        // something..
        return Collections.singletonList("blah");
    }
}
```

 위와 같은 패턴은 dictionary라는 딱 하나의 자원만 사용하지만, 자원이 몇 개든 의존 관계가 어떻든 상관없이 잘 동작합니다. 또한 불변성을 보장하여 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있기도 합니다.

 이 패턴의 쓸만한 변형으로, 생성자에 자원 팩토리를 넘겨주는 방식이 있습니다. 팩토리란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말합니다. 즉, 팩토리 메서드 패턴을 구현하는 것입니다. 대표적으로 자바 8에서 소개한 Supplier<T> 인터페이스가 팩토리를 표현한 예시입니다.

 의존 객체 주입이 유연성과 테스트 용이성을 개선해주긴 하지만, 의존성이 수 천 개나 되는 큰 프로젝트에서는 코드의 복잡성을 증가시키기도 합니다. 스프링 프레임워크 같은 의존 객체 주입 프레임워크를 사용하면 이런 복잡성을 줄일 수 있습니다.

---

## 참고자료

[http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
