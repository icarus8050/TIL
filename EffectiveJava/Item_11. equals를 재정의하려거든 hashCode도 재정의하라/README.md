# Item 11. equals를 재정의하려거든 hashCode도 재정의하라

 equals()를 재정의한 클래스는 모두 hashCode()도 재정의해야 합니다. 그렇지 않으면 hashCode() 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으키게 됩니다.

#### Object에 명세되어 있는 규약

-   equals() 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode() 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
-   equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode()는 똑같은 값을 반환해야 한다.
-   equals(Obejct)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode()가 서로 다른 값을 반환할 필요는 없다. 하지만 다른 객체에 대해서 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

 위 규약에서 지켜지지 않았을 경우, 문제가 되는 항목은 두 번째 입니다. 즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 합니다.

```
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");
m.get(new PhoneNumber(707, 867, 5309));	//null을 반환
```

 위 코드는 3번째 라인을 실행하면 null이 반환됩니다. PhoneNumber 클래스는 논리적으로는 동치이지만 hashCode()를 재정의하지 않았기 때문에 서로 다른 해시코드를 반환하여 null이 반환합니다.

 이러한 문제를 해결하기 위해서는 아래처럼 hashCode()를 재정의 할 수 있지만 절대 사용해서는 안됩니다.

```
//절대 사용하지 말 것!
@Override
public int hashCode() {
    return 42;
}
```

 이 코드는 모든 객체에 대해서 같은 해시코드를 반환하므로 같은 버킷에 담기면 마치 연결 리스트처럼 동작하게 됩니다. 그 결과 평균 수행 시간이 O(n)으로 느려지기 때문에 해시테이블의 장점이 모두 사라지게 됩니다.

 좋은 해시 함수라면 서로 다른 인스턴스에는 다른 해시코드를 반환해야 합니다. 이 규약은 세 번째 규약에서 요구하는 속성입니다. 이상적인 해시 함수는 서로 다른 인스턴스에 대해서 32비트 정수 범위에 균일하게 분배해야 합니다.

#### hashCode()를 작성하는 요령

1.  int 변수 result를 선언한 후 값 c로 초기화한다. 이때 c는 해당 객체의 첫 번째 핵심 필드를 단계 2.1 방식으로 계산한 해시코드다. (핵심 필드란 equals() 비교에 사용되는 필드를 말한다.)
2.  해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.
    1.  해당 필드의 해시코드 c를 계산한다.
        1.  기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본 타입의 박싱 클래스다.
        2.  참조 타입 필드면서 이 클래스의 equals() 메서드가 이 필드의 equals()를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode()를 재귀적으로 호출한다. 계산이 복잡해질 것 같으면, 이 필드의 표준형을 만들어 그 표준형의 hashCode()를 호출한다. 필드의 값이 null이면 0을 사용한다. (다른 상수도 괜찮지만 전통적으로 0을 사용한다.)
        3.  필드가 배열이라면, 핵심 원소 각가을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 단계 2.2 방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수(0을 추천)를 사용한다. 모든 원소가 핵심 원소라면 Arrays.hashCode()를 사용한다.
    2.  단계 2.1에서 계산한 해시코드 c로 result를 갱신한다. 코드로는 다음과 같다. result = 31 \* result + c;
3.  result를 반환한다.

 파생 필드는 해시코드 계산에서 제외해도 됩니다. 즉, 다른 필드로부터 계산해낼 수 있는 필드는 모두 무시해도 됩니다. 또한 equals() 비교에 사용되지 않는 필드는 **반드시** 제외해야 합니다.

 단계 2.2의 곱셈 31 \* result는 필드를 곱하는 순서에 따라 result 값이 달라지게 합니다. 이 곱셈 연산은 시프트 연산과 뺄셈으로 대체해 최적화할 수 있습니다(31 \* i = (i << 5) - i).

 위의 규칙을 적용한 PhoneNumber 클래스의 hashCode() 예시는 아래와 같습니다.

```
@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```

 Objects 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드인 hash()를 제공합니다. 이 메서드를 활용하면 앞서의 요령대로 구현한 코드와 비슷한 수준의 hashCode() 함수를 단 한 줄로 작성할 수 있습니다. 하지만 속도는 더 느립니다. 입력 인수를 담기 위한 배열이 만들어지고, 입력 중 기본 타입이 있다면 박싱과 언박싱도 거쳐야 하기 때문입니다. 다음 코드는 Objects.hash()를 이용한 방법입니다.

```
@Override
public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
```

 클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다는 캐싱하는 방식을 고려해야 합니다. 이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해둬야 합니다. 이를 위해 지연 초기화(lazy initialization) 전략을 사용할 수 있지만, 필드를 지연 초기화하기 위해서는 Thread-Safe 하도록 신경써야 합니다.

```
private int hashCode; //자동으로 0으로 초기화

@Override
public int hashCode() {
    int result = hashCode;
    if (result == 0) {
        result = Short.hashCode(areaCode);
        result = result * 31 + Short.hashCode(prefix);
        result = result * 31 + Short.hashCode(lineNum);
        hashCode = result;
    }
    return result;
}
```

---

## 참고자료

[http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
