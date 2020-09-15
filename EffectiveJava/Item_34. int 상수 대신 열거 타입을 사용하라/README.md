# Item 34. int 상수 대신 열거 타입을 사용하라

#### 상수 열거 패턴

```
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

 위의 코드처럼 정수 상수를 한 묶음으로 모아서 선언해놓은 것이 정수 열거 패턴입니다.

#### 정수 열거 패턴의 단점

-   정수 열거 패턴에는 타입 안전을 보장할 수 없습니다. 예를 들어 오렌지를 건네야 할 메서드에 사과를 보내고 동등 연산자(==)로 비교하더라도 컴파일러는 아무런 경고 메시지를 출력하지 않습니다.
-   자바는 정수 열거 패턴을 위한 별도의 이름공간(namespace)을 지원하지 않기 때문에 어쩔 수 없이 접두어를 써서 이름 충돌을 방지해야 합니다.
-   정수 상수는 문자열로 출력하기가 까다롭습니다. 그 값을 출력하거나 디버거로 살펴보면 단지 숫자로만 보이기 때문에 디버깅에 도움이 되지 않습니다.
-   같은 정수 열거 그룹에 속한 모든 상수를 한 바퀴 순회하는 방법이 마땅치 않고, 그 안에 상수가 몇 개인지도 알 수가 없습니다.
-   평범한 상수를 나열한 것뿐이라 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨집니다. 따라서 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일해야 합니다. 다시 컴파일하지 않은 클라이언트는 실행이 되더라도 엉뚱하게 동작하게 됩니다.
-   문자열 열거 패턴 형식으로 변형할 수 있지만, 상수의 이름을 하드 코딩해야 하기 때문에 오타가 있어도 컴파일러를 통과하고 런타임 버그가 생길 가능성이 생깁니다.

#### 열거 타입

 열거 타입은 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입입니다.

 열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개합니다. 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final 입니다. 따라서 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없으니 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재함이 보장됩니다(싱글턴).

**enum 타입의 작성 예시**

```
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

 열거 타입은 컴파일타임 타입 안전성을 제공합니다. 만약 다른 타입의 값을 넘기려하면 컴파일 오류가 납니다.

 열거 타입은 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현할 수도 있습니다. Object 메서드들과 Comparable, Serializable을 구현했으며, 그 직렬화 형태도 웬만큼 변형을 가해도 문제없이 동작하게끔 구현되어 있습니다.

#### 데이터와 메서드를 갖는 열거 타입

```
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6),
    MARS(6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN(5.685e+26, 6.027e7),
    URANUS(8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass; // 질량 (단위: 킬로그램)
    private final double radius; // 반지름 (단위: 미터)
    private final double surfaceGravity; // 표면중력 (단위: m / s^2)

    // 중력 상수 (단위 : m^3 / kg s^2)
    private static final double G = 6.67300E-11;

    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double getMass() {
        return mass;
    }

    public double getRadius() {
        return radius;
    }

    public double surfaceGravity() {
        return surfaceGravity;
    }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity; // F = ma
    }
}

```

** 열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 됩니다.** 열거 타입은 근본적으로 불변이라 **모든 필드는 final**이어야 합니다. 필드를 public으로 선언해도 되지만, private으로 두고 별도의 public 접근자 메서드를 두는 것이 좋습니다.

 열거 타입은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드인 values()를 제공합니다. 값들은 선언된 순서로 저장됩니다.

 각 열거 타입 값의 toString() 메서드는 상수 이름을 문자열로 반환합니다.

 열거 타입은 일반 클래스와 마찬가지로, 그 기능을 클라이언트에 노출해야 할 합당한 이유가 없다면 private로, 혹은 (필요하다면) package-private으로 선언하는 것이 좋습니다.

 널리 쓰이는 열거 타입은 톱레벨 클래스로 만들고, 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스로 만듭니다.

 열거 타입에는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 valueOf() 메서드가 자동 생성됩니다.

#### 상수에 다양한 기능을 제공하는 방법

```
public enum Operations {
    PLUS, MINUS, TIMES, DIVIDE;

    public double apply(double x, double y) {
        switch (this) {
            case PLUS:
                return x + y;
            case MINUS:
                return x - y;
            case TIMES:
                return x * y;
            case DIVIDE:
                return x / y;
        }

        throw new AssertionError("알 수 없는 연산: " + this);
    }
}

```

 위의 코드는 사칙연산에 대한 연산의 종류를 열거 타입으로 선언하고, 각 연산마다 switch문을 통해 분기하고 있습니다. 만약 새로운 상수를 추가하면 해당 case문을 추가해야 합니다. 혹시라도 깜빡한다면, 컴파일은 되지만 새로 추가한 연산을 수행하려 할 때 "알 수 없는 연산"이라는 런타임 오류를 낼 것입니다.

 이를 더 나은 방법으로 동작하도록 만드는 방법은 추상 메서드를 선언하고 각 상수별 클래스 몸체(constant-specific class body)를 자신에 맞게 재정의 하는 방법입니다.

**상수별 메서드 구현을 활용한 열거 타입**

```
public enum Operations {
    PLUS {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    }, 
    DIVIDE {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    };
    
    public abstract double apply(double x, double y);
}

```

 열거 타입의 toString() 메서드를 재정의하려거든, toString이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드도 함께 제공하는 것을 고려해 볼 수 있습니다.

```
import java.util.Map;
import java.util.Optional;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public enum Operations {
    PLUS("+") {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    private static Map<String, Operations> stringToEnum =
            Stream.of(values()).collect(
                    Collectors.toMap(Object::toString, e -> e));

    Operations(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }

    // 지정한 문자열에 해당하는 Operations을 (존재한다면) 반환한다.
    public static Optional<Operations> fromString(String symbol) {
        return Optional.ofNullable(stringToEnum.get(symbol));
    }

    public abstract double apply(double x, double y);
}

```

** Operations 상수가 stringToEnum 맵에 추가되는 시점은 열거 타입 상수 생성 후 정적 필드가 초기화될 때입니다.** **열거 타입의 정적 필드 중 열거 타입의 생성자에서 접근할 수 있는 것은 상수 변수 뿐**입니다. 열거 타입 생성자가 실행되는 시점에는 정적 필드들이 아직 초기화되기 전이라, 자기  자신을 추가하지 못하게 하는 제약이 꼭 필요합니다. 이 제약의 특수한 예로, 열거 타입 생성자에서 같은 열거 타입의 다른 상수에도 접근할 수 없습니다.

#### 상수별 메서드 구현의 단점

 상수별 메서드 구현은 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있습니다. 상수별로 같은 처리를 하는 메서드가 있더라도 추상 메서드를 재정의 해야하니 중복되는 코드가 발생하게 됩니다.

 이러한 문제를 깔끔하게 해결하는 방법은 새로운 상수를 추가할 때 **'전략'**을 선택하도록 구현하는 것입니다.

```
public enum PayrollDay {
    MONDAY(PayType.WEEKDAY),
    TUESDAY(PayType.WEEKDAY),
    WEDNESDAY(PayType.WEEKDAY),
    THURSDAY(PayType.WEEKDAY),
    FRIDAY(PayType.WEEKDAY),
    
    SATURDAY(PayType.WEEKEND),
    SUNDAY(PayType.WEEKEND);
    

    private final PayType payType;
    
    PayrollDay(PayType payType) {
        this.payType = payType;
    }
    
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }
    
    // 전략 열거 타입
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            @Override
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };
        
        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;
        
        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}

```

 위 코드는 주중과 주말에 따른 잔업 수당을 포함하는 임금을 계산하는 열거 타입입니다. 잔업수당 계산을 private 중첩 열거 타입(PayType)으로 정의하고, PayrollDay 열거 타입의 생성자에서 이중 적당한 것을 선택합니다. 이 방법은 PayrollDay 열거 타입은 잔업수당 계산을 그 전략 열거 타입에 위임하여, switch 문이나 상수별 메서드 구현이 필요 없게 됩니다.

 switch 문은 열거 타입의 상수별 동작을 구현하는 데 적합하지 않습니다. 하지만 **기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch 문이 좋은 선택이 될 수 있습니다.** 예를 들어 서드파티에서 가져온 Operations 열거 타입이 있는데, 각 연산의 반대 연산을 반환하는 메서드가 필요하다면 아래와 같이 적용할 수 있습니다.

```
public static Operations inverse(Operations op) {
    switch(op) {
        case PLUS: return Operations.MINUS;
        case MINUS: return Operations.PLUS;
        case TIMES: return Operations.DIVIDE;
        case DIVEDE: return Operations.TIMES;
        
        default: throw new AssertionError("알 수 없는 연산: " + op);
    }
}
```

 추가하려는 메서드가 의미상 열거 타입에 속하지 않는다면 직접 만든 열거 타입이라도 이 방식을 적용하는 것이 좋습니다. 종종 쓰이지만 열거 타입 안에 포함할만큼 유용하지는 않은 경우도 마찬가지 입니다.

열거 타입을 메모리에 올리는 공간과 초기화하는 시간이 들긴 하지만 대부분의 경우 성능적인 부분에서 상수와 별반 다르지 않습니다.

#### 열거 타입을 사용해야 할 때

** 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용해야 합니다.**

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
