# Item 42. 익명 클래스보다는 람다를 사용하라

#### 자바 8 이전, 익명 클래스

 자바 8 이전에는 함수 객체를 만드는 주요 수단은 익명 클래스였습니다. 하지만 익명 클래스 방식은 코드가 너무 길기 때문에 자바는 함수형 프로그래밍에 적합하지 않았습니다. 아래의 코드는 문자열을 길이 순으로 정렬하는데, 정렬을 위한 비교 함수로 익명 클래스를 사용하고 있습니다.

```
import java.util.Arrays;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("banana", "cat", "orange", "apple");

        Collections.sort(words, new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                return Integer.compare(o1.length(), o2.length());
            }
        });

        System.out.println(words);
    }
}

```

#### 자바 8 이후, 람다식

 자바 8부터는 추상 메서드 하나짜리 인터페이스로 정의되는 함수형 인터페이스를 이용하여 람다식을 사용해 코드를 훨씬 간결하게 작성할 수 있습니다.

```
public class Main {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("banana", "cat", "orange", "apple");

        Collections.sort(words,
                (o1, o2) -> Integer.compare(o1.length(), o2.length()));

        System.out.println(words);
    }
}

```

 위 코드는 익명 클래스에서 작성했던 코드를 람다식으로 작성하여 훨씬 간결하게 만든 코드입니다. 불필요한 코드들이 사라지고 어떤 동작을 하는지 명확하게 드러납니다.

 람다, 매개변수(o1, o2), 반환 값의 타입은 각각 (Comparator<String>), String, int지만 코드에서는 생략되었습니다. 이는 컴파일러가 문맥을 살펴서 타입을 추론해주기 때문에 가능합니다.

** 타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략하는 것이 좋습니다.**

 람다 자리에 비교자 생성 메서드를 사용하면 아래 코드처럼 더 간결하게 만들 수 있습니다.

```
import java.util.Arrays;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("banana", "cat", "orange", "apple");

        Collections.sort(words,
                Comparator.comparingInt(String::length));

        System.out.println(words);
    }
}

```

#### 람다 기반의 Operation 열거 타입

```
public enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x/ y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}

```

 위의 예시는 열거 타입의 인스턴스 필드를 이용하여 상수 별로 다르게 동작하는 코드를 간단하게 구현한 코드입니다. 각 열거 타입 상수의 동작을 람다로 구현해 생성자에 넘기고, 생성자는 이 람다를 인스턴스 필드로 저장해둔 것입니다. 그런 다음 apply 메서드에서 필드에 저장된 람다를 호출하기만 하면 정의한 동작을 수행하게 됩니다.

#### 람다를 사용하지 말아야 하는 경우

** 람다는 이름이 없고 문서화도 못합니다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 합니다.** 람다는 한 줄 일 때 가장 좋고 길어야 세 줄 안에 끝내는 것이 좋습니다.

 열거 타입 생성자에 넘겨지는 인수들의 타입은 컴파일타임에 추론됩니다. 따라서 열거 타입 생성자 안의 람다는 열거 타입의 인스턴스 멤버에 접근할 수 없습니다(인스턴스는 런타임에 만들어지기 때문입니다). 따라서 상수별 동작을 단 몇 줄로 구현하기 어렵거나, 인스턴스 필드나 메서드를 사용해야만 하는 상황이라면 상수별 클래스 몸체를 사용해야 합니다.

 람다는 자신을 참조할 수 없습니다. 람다에서의 this 키워드는 바깥 인스턴스를 가르킵니다. 그래서 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스를 써야합니다.

 람다도 익명 클래스처럼 직렬화 형태가 구현별로(가령 가상머신별로) 다를 수 있습니다. 따라서 **람다를 직렬화하는 일은 극히 삼가야 합니다(익명 클래스의 인스턴스도 마찬가지입니다).** 직렬화해야만 하는 함수 객체가 있다면(가령 Comparator처럼) private 정적 중첩 클래스의 인스턴스를 사용하는 것이 좋습니다.

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LET&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LET&Kc=)
