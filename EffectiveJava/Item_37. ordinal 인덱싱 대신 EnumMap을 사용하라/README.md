# Item 37. ordinal 인덱싱 대신 EnumMap을 사용하라

```
public class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;

    public Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
}

```

```
import java.util.HashSet;
import java.util.List;
import java.util.Set;

public class OrdinalIndexingMain {
    public static void main(String[] args) {
        List<Plant> garden = List.of(
                new Plant("A", Plant.LifeCycle.ANNUAL),
                new Plant("B", Plant.LifeCycle.BIENNIAL),
                new Plant("C", Plant.LifeCycle.PERENNIAL),
                new Plant("D", Plant.LifeCycle.ANNUAL));

        Set<Plant>[] plantsByLifeCycle =
                (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
                
        for (int i = 0; i < plantsByLifeCycle.length; i++) {
            plantsByLifeCycle[i] = new HashSet<>();
        }

        for (Plant plant : garden) {
            plantsByLifeCycle[plant.lifeCycle.ordinal()].add(plant);
        }

        // 결과 출력
        for (int i = 0; i < plantsByLifeCycle.length; i++) {
            System.out.printf("%s: %s%n",
                    Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
        }
    }
}

```

 위 코드는 Plant 클래스의 LifeCycle 별로 총 3개의 집합을 배열로 만들고, ordinal 값을 그 배열의 인덱스로 사용한 코드입니다.

#### ordinal 인덱싱의 문제점

 방금 보았던 코드에서 사용한 배열은 제네릭과 호환이 되지 않으므로 비검사 형변환을 수행해야 합니다. 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 합니다.

 가장 심각한 문제는 정확한 정숫값을 사용한다는 것을 작성자가 직접 보증해야한다는 점입니다. 정수는 열거 타입과 달리 타입 안전하지 않기 때문입니다. 잘못된 값을 사용하면 잘못된 동작을 수행하거나 (운이 좋다면) ArrayIndexOutOfBoundsException을 던질 것입니다.

#### EnumMap을 사용하는 방법

 ordinal 인덱싱의 문제점을 해결하기 위해서 EnumMap을 사용할 수 있습니다. EnumMap은 열거 타입을 키로 사용하도록 설계한 구현체입니다.

**EnumMap을 사용해 데이터와 열거 타입을 매핑한 예제**

```
import java.util.*;

import static java.util.stream.Collectors.groupingBy;
import static java.util.stream.Collectors.toSet;

public class EnumMapMain {
    public static void main(String[] args) {
        List<Plant> garden = List.of(
                new Plant("A", Plant.LifeCycle.ANNUAL),
                new Plant("B", Plant.LifeCycle.BIENNIAL),
                new Plant("C", Plant.LifeCycle.PERENNIAL),
                new Plant("D", Plant.LifeCycle.ANNUAL));

        Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
                new EnumMap<>(Plant.LifeCycle.class);
                
        for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
            plantsByLifeCycle.put(lc, new HashSet<>());
        }

        for (Plant plant : garden) {
            plantsByLifeCycle.get(plant.lifeCycle).add(plant);
        }

        System.out.println(plantsByLifeCycle);
    }
}

```

 안전하지 않은 형변환은 쓰지 않고, 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하니 출력 결과에 직접 레이블을 달 일도 없습니다.  나아가 배열의 인덱스를 계산하는 과정에서 오류가 날 가능성도 생기지 않습니다.

 EnumMap은 내부에서 배열을 사용하기 때문에 ordinal을 쓴 배열과 성능이 비슷합니다. 내부 구현 방식을 안으로 숨겨서 Map의 타입 안전성과 배열의 성능을 모두 얻어낸 것입니다.

 EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로, 런타임 제네릭 타입 정보를 제공합니다.

#### 스트림을 사용한 코드

**스트림을 사용한 코드 - EnumMap을 사용하지 않는다.**

```
System.out.println(Arrays.stream(garden)
    .collect(groupingBy(p -> p.lifeCycle)));
    
```

  위 코드는 EnumMap이 아닌 고유한 맵 구현체를 사용했기 때문에 EnumMap을 써서 얻은 공간과 성능 이점이 사라진다는 문제가 있습니다.

 매개변수 3개짜리 Collectors.groupingBy 메서드는 mapFactory 매개변수에 원하는 맵 구현체를 명시해 호출할 수 있습니다.

**스트림을 사용한 코드 - EnumMap을 이용해 데이터와 열거 타입을 매핑한다.**

```
System.out.println(Arrays.stream(garden)
    .collect(groupingBy(p -> p.lifeCycle,
        () -> new EnumMap<>(LIfeCycle.class), toSet())));
```

 스트림을 사용하면 EnumMap만 사용했을 때와는 살짝 다르게 동작합니다. **EnumMap 버전은 언제나 열거 타입당 하나씩 중첩 맵을 만들지만, 스트림 버전에서는 해당 열거 타입에 속하는 객체가 있을 때만 만듭니다.**

#### 두 열거 타입 값을 매핑하는 방법

 다음은 두 가지 상태(Phase)를 전이(Transition)와 매핑하도록 구현한 프로그램입니다. 예를 들어 액체(LIQUID)에서 고체(SOLID)로의 전이는 응고(FREEZE)가 되고, 액체에서 기체(GAS)로의 전이는 기화(BOIL)가 됩니다.

**배열들의 배열의 인덱스에 ordinal()을 사용 - 나쁜 예**

```
public enum Phase {
    SOLID, LIQUID, GAS;
    
    public enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
        
        // 행은 from의 ordinal을, 열은 to의 ordinal을 인덱스로 쓴다.
        private static final Transition[][] TRANSITIONS = {
                { null, MELT, SUBLIME },
                { FREEZE, null, BOIL },
                { DEPOSIT, CONDENSE, null }
        };
    }
}

```

 위 예제는 앞서 살펴보았던 예제와 마찬가지로 컴파일러는 ordinal과 배열 인덱스 관계를 알 수가 없습니다. 즉, PHase나 Phase.Transition 열거 타입을 수정하면서 상전이 표 TRANSITIONS를 함께 수정하지 않거나 실수로 잘못 수정하면 런타임 오류가 발생할 수 있습니다. 거기다가 상전이 표의 크기는 상태의 가짓수가 늘어나면 제곱해서 커지며 null로 채워지는 칸도 늘어날 것입니다.

**중첩 EnumMap으로 데이터와 열거 타입 쌍을 연결한 예시**

```
import java.util.EnumMap;
import java.util.Map;
import java.util.stream.Stream;

import static java.util.stream.Collectors.groupingBy;
import static java.util.stream.Collectors.toMap;

public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        // 상전이 맵을 초기화한다.
        private static final Map<Phase, Map<Phase, Transition>>
                m = Stream.of(values()).collect(groupingBy(t -> t.from,
                () -> new EnumMap<>(Phase.class),
                toMap(t -> t.to, t -> t,
                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));


        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}

```

 이 맵의 타입인 Map<Phase, Map<Phase, Transition>은 "이전 상태에서 '이후 상태에서 전이로의 맵'에 대응시키는 맵"이라는 뜻입니다.

 이러한 맵의 맵을 초기화하기 위해 수집기(java.util.stream.Collector)2개를 차례로 사용했습니다. 첫 번째 수집기인 groupingBy에서는 전이를 이전 상태를 기준으로 묶고, 두 번째 수집기인 toMap에서는 이후 상태를 전이에 대응시키는 EnumMap을 생성했습니다.

 이 상테에서 새로운 상태인 플라즈마(PLASMA)를 추가하면 간단하게 아래와 같이 추가할 수 있게 됩니다.

**EnumMap 버전에 새로운 상태 추가하기**

```
import java.util.EnumMap;
import java.util.Map;
import java.util.stream.Stream;

import static java.util.stream.Collectors.groupingBy;
import static java.util.stream.Collectors.toMap;

public enum Phase {
    SOLID, LIQUID, GAS, PLASMA;

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
        IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        // 나머지 코드는 동일
    }
}

```

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LET&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LET&Kc=)
