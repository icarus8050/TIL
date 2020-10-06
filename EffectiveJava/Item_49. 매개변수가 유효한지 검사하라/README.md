# Item 49. 매개변수가 유효한지 검사하라

  메서드와 생성자 대부분은 입력 매개변수의 값이 특정 조건을 만족하기를 기대합니다. 예를 들어 인덱스 값은 음수이면 안 되며, 객체 참조는 null이 아니어야 한다는 식입니다. 이런 제약은 반드시 문서화해야 하며 메서드 몸체가 시작되기 전에 검사해야 합니다.

 "오류는 가능한 한 빨리 (발생한 곳에서) 잡아야 한다"는 일반 원칙의 한 사례이기도 합니다. 오류를 발생한 즉시 잡지 못하면 해당 오류를 감지하기 어려워지고, 감지하더라도 오류의 발생 지점을 찾기 어려워 집니다.

#### 매개변수 검사를 제대로 하지 못하면 생기는 문제점

 메서드가 수행되는 중간에 모호한 예외를 던지며 실패할 수 있습니다. 더 나쁜 상황은 메서드는 문제없이 수행됐지만, 어떤 객체를 이상한 상태로 만들어놓아서 미래의 알 수 없는 시점에 이 메서드와 관련 없는 오류를 낼 때 입니다.

#### 공개 API를 반드시 문서화하라

 public과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야 합니다(@throws 자바독 태그를 사용하면 됩니다). 보통은 **IllegalArgumentException, IndexOutOfBoundsException, NullPointerException** 중 하나가 될 것입니다. 매개변수의 제약을 문서화한다면 그 제약을 어겼을 때 발생하는 예외도 함께 기술해야 합니다.

**BigInteger 클래스의 mod() 메서드 예시**

```
    /**
     * Returns a BigInteger whose value is {@code (this mod m}).  This method
     * differs from {@code remainder} in that it always returns a
     * <i>non-negative</i> BigInteger.
     *
     * @param  m the modulus.
     * @return {@code this mod m}
     * @throws ArithmeticException {@code m} &le; 0
     * @see    #remainder
     */
    public BigInteger mod(BigInteger m) {
        if (m.signum <= 0)
            throw new ArithmeticException("BigInteger: modulus not positive");

        BigInteger result = this.remainder(m);
        return (result.signum >= 0 ? result : result.add(m));
    }
```

 위 메서드는 m이 null이면 m.signum() 호출 때 NullPointerException을 던집니다. 그런데 "m이 null일 때 NullPointerException을 던진다"라는 말은 메서드 설명 어디에도 없습니다. 그 이유는 **이 설명을 (개별 메서드가 아닌) BigInteger 클래스 수준에서 기술했기 때문입니다.**

 클래스 수준 주석은 그 클래스의 모둔 public 메서드에 적용되므로 각 메서드에 일일이 기술하는 것보다 훨씬 깔끔한 방법 입니다.

 @Nullable이나 이와 비슷한 애너테이션을 사용해 특정 매개변수는 null이 될 수 있다고 알려줄 수도 있지만, 표준적인 방법은 아닙니다.

** 자바 7에 추가된 java.util.Objects.requireNonNull 메서드는 유연하고 사용하기도 편하니, 더 이상 null 검사를 수동으로 하지 않아도 됩니다.**

```
this.strategy = Objects.requireNonNull(strategy, "ErrorMessage");
```

 자바 9에서는 Objects에 범위 검사 기능도 더해졌습니다. 바로 **checkFromIndexSize, checkFromToIndex, checkIndex**라는 메서드들 입니다. 이들 메서드는 null 검사 메서드만큼 유연하지는 않습니다. 예외 메시지를 지정할 수 없고, 리스트와 배열 전용으로 설계됐습니다. 또한, 닫힌 범위(closed range; 양 끝단 값을 포함하는)는 다루지 못합니다.

#### 단언문(assert)

 공개되지 않은 메서드라면 패키지 제작자인 작성자가 메서드가 호출되는 상황을 통제할 수 있습니다. 따라서 오직 유효한 값만이 메서드에 넘겨지리라는 것을 작성자가 보증할 수 있습니다. 다시 말해 public이 아닌 메서드라면 단언문(assert)을 사용해 매개변수 유효성을 검증할 수 있습니다.

```
    private static void sort(long a[], int offset, int length) {
        assert a != null;
        assert offset >= 0 && offset <= a.length;
        assert length >= 0 && length <= a.length - offset;
        //계산 수행 ...
    }
```

 여기서의 핵심은 이 단언문들은 자신이 단언한 조건이 무족너 참이라고 선언한다는 것입니다.

#### 일반적인 유효성 검사와 단언문이 다른 점

1.  실패하면 AssertionError를 던집니다.
2.  런타임에 아무런 효과도, 아무런 성능 저하도 없습니다(단, java를 실행할 때 명령줄에서 -ea 혹은 --enableassertions 플래그를 설정하면 런타임에 영향을 줍니다).

#### 규칙의 예외 상황

 메서드 몸체 실행 전에 매개변수 유효성을 검사해야 한다는 규칙에도 예외는 있습니다. 유효성 검사 비용이 지나치게 높거나 실용적이지 않을 때, 혹은 계산 과정에서 암묵적으로 검사가 수행될 때 입니다.

 예를 들어 Collections.sort(List)처럼 객체 리스트를 정렬하는 메서드를 생각해 보겠습니다. 리스트 안의 객체들은 모두 상호 비교될 수 있어야 하며, 정렬 과정에서 이 비교가 이뤄집니다. 만약 상호 비교될 수 없는 타입의 객체가 들어있다면 그 객체와 비교할 때 ClassCastException을 던질 것입니다. 따라서 비교하기 앞서 리스트 안의 모든 객체가 상호 비교될 수 있는지 검사해봐야 별다른 실익이 없습니다.

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
