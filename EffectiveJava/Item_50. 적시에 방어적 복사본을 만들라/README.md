# Item 50. 적시에 방어적 복사본을 만들라

 자바는 JVM이 메모리를 관리해주는 안전한 언어입니다. 덕분에 특별한 노력없이 메모리를 관리하고, 시스템의 다른 부분에서 불변식을 지킬 수 있습니다. 하지만 아무런 노력없이 다른 클래스로부터의 침범을 막을 수 있는건 아닙니다. 이를 예방하기 위해서 클라이언트가 불변식을 깨뜨리는 것을 막을 수 있도록 방어적으로 프로그래밍해야 합니다.

```
import java.util.Date;

public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(
                    start + " after " + end);
        this.start = start;
        this.end = end;
    }

    public Date getStart() {
        return start;
    }

    public Date getEnd() {
        return end;
    }
}

```

 위 클래스는 객체의 허락 없이는 외부에서 내부의 내용을 수정할 수 없는 것처럼 보입니다. 하지만 아래의 코드를 보면 클래스의 불변성을 간단하게 깨뜨려 버립니다.

```
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78);
```

 위와 같은 예시는 자바 8 이후부터는 간단하게 해결할 수 있습니다. Date 대신 불변인 Instant를 사용하거나 LocalDateTime 또는 ZonedDateTime을 사용하면 됩니다. **Date는 불변성을 해치기 때문에 새로운 코드를 작성할 때는 더 이상 사용해서는 안됩니다.**

####  방어적 복사

 불변성을 해치는 코드로부터 Period 인스턴스의 내부를 보호하려면 생성자에서 받는 가변 매개변수 각각을 방어적으로 복사(defensive copy)해야 합니다. 그리고 Period 인스턴스는 원본이 아닌 복사본을 사용하면 됩니다.

```
import java.util.Date;

public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());

        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(
                    start + " after " + end);
    }

    public Date getStart() {
        return new Date(start.getTime());
    }

    public Date getEnd() {
        return new Date(end.getTime());
    }
}

```

 새로 작성한 생성자를 사용하면 앞서의 공격은 더 이상 Period에 위협이 되지 않습니다.

** 매개변수의 유효성을 검사하기 전에 방어적 복사본을 만들고, 이 복사본으로 유효성을 검사해야 합니다.** 이는 멀티스레드 환경에서는 원본 객체의 유효성을 검사한 후 복사본을 만드는 찰나의 취약한 순간에 다른 스레드가 원본 객체를 수정할 위험이 있기 때문입니다. 방어적 복사를 매개변수 유효성 검사전에 수행하면 이런 위험을 예방할 수 있습니다.

 방어적 복사에 Date의 clone 메서드를 사용하지 않은 점도 주목해야 합니다. Date는 final이 아니므로 악의를 가진 하위 클래스의 인스턴스를 반환할 수도 있습니다. 예를 들어, 이 하위 클래스가 start와 end 필드의 참조를 private 정적 리스트에 담아뒀다가 공격자에게 이 리스트에 접근하는 길을 열어줄 수도 있습니다. 이런 공격을 막기 위해서는 **매개변수가 제3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용해서는 안 됩니다.**

#### 메서드의 방어적 복사

 위에 방어적 복사가 적용된 코드를 보시면 get메서드를 통한 접근자에서 새로운 인스턴스를 만들어 반환하는 것을 확인할 수 있습니다. 이는 **내부의 가변 필드가 외부에 의해 접근하는 것을 막기 위함**입니다. 이를 위해 가변 필드의 방어적 복사본을 반환하는 것입니다.

 생성자와 달리 접근자 메서드에서는 방어적 복사에 clone을 사용해도 됩니다. Period가 가지고 있는 Date 객체는 java.util.Date임이 확실하기 때문입니다. 즉 Date는 신뢰할 수 없는 하위 클래스가 아니기 때문입니다. 그렇더라도 인스턴스를 복사하는 데는 일반적으로 생성자나 정적 팩토리를 쓰는 것이 좋습니다.

#### 되도록 불변 객체를 조합해 객체를 구성

 메서드든 생성자든 클라이언트가 제공한 객체의 참조를 내부의 자료구조에 보관해야 할 때면 항시 그 객체가 잠재적으로 변경될 수 있는지를 생각해야 합니다. 변경 될 수 있다면 그 객체가 클래스에 넘겨진 뒤 임의로 변경되어도 문제없이 동작할지를 따져보아야 합니다.

 이상의 모든 작업에서 우리는 **"되도록 불변 객체들을 조합해 객체를 구성해야 방어적 복사를 할 일이 줄어든다"**는 교훈을 얻을 수 있습니다. Period 예제의 경우, 자바 8이상으로 개발해도 된다면 Instant, LocalDateTime 또는 ZonedDateTime을 사용하는 것이 좋습니다. 이전 버전의 자바를 사용한다면 Date 참조대신 Date.getTime()이 반환하는 long 정수를 사용하는 방법을 써도 됩니다.

 방어적 복사에는 성능 저하가 따르고, 또 항상 쓸 수 있는 것은 아닙니다. (같은 패키지에 속하는 등의 이유로) 호출자가 컴포넌트 내부를 수정하지 않으리라 확신하면 방어적 복사를 생략할 수 있습니다. 이러한 상황이라도 호출자에서 해당 매개변수가 반환값을 수정하지 말아야 함을 명확히 문서화하는게 좋습니다.

 다른 패키지를 사용한다고 해서 넘겨받은 가변 매개변수를 항상 방어적으로 복사해 저장해야 하는 것은 아닙니다. 이를 위해 해당 메서드를 호출하는 클라이언트는 해당 객체를 더 이상 직접 수정하는 일이 없다고 약속해야 합니다. 그리고 클라이언트가 건네주는 가변 객체의 통제권을 넘겨받는다고 기대하는 메서드나 생성자에도 그 사실을 확실히 문서화 해야 합니다.

#### 방어적 복사를 생략해도 되는 상황

 해당 클래스와 그 클라이언트가 상호 신뢰할 수 있을 때, 혹은 불변식이 깨지더라도 그 영향이 오직 호출한 클라이언트로 국한될 때로 한정해야 합니다.

 그 예시로 래퍼 클래스 패턴을 들 수 있습니다. 래퍼 클래스의 특성상 클라이언트는 래퍼에 넘긴 객체에 여전히 직접 접근할 수 있습니다. 따라서 래퍼의 불변식을 쉽게 파괴할 수 있지만 그 영향을 오직 클라이언트 자신만 받게 됩니다.

---

## 참고자료
