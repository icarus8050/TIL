# Item 26. 로 타입은 사용하지 말라

#### 타입 매개변수(type parameter)

 클래스와 인터페이스에 타입 매개변수(type parameter)가 쓰이면, 이를 **제네릭 클래스** 혹은 **제네릭 인터페이스**라고 합니다. 예를 들면, List 인터페이스는 원소의 타입을 나타내는 타입 매개변수 E를 받습니다. 제네릭 클래스와 제네릭 인터페이스를 통틀어 **제네릭 타입(generic type)**이라고 합니다.

 각각의 제네릭 타입은 일련의 매개변수화 타입(parameterized type)을 정의합니다.매개변수화 타입은 클래스(혹은 인터페이스) 이름이 나오고, 이어서 꺽쇠괄호 안에 실제 타입 매개변수들을 나열합니다. 예를 들면, List<String>은 원소의 타입이 String인 리스트를 뜻하는 매개변수화 타입입니다.

#### 로 타입(raw type)

 로 타입이란 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말합니다. 예를 들면, List<E.의 로 타입은 List입니다.

 제네릭을 지원하기 전에는 컬렉션을 다음과 같이 선언했습니다.

```
// Stamp 인스턴스만 취급한다.
private final Collection stamps = ...;
```

 위 코드는 Stamp 대신 전혀 다른 타입을 넣어도 아무 오류 없이 컴파일되고 실행될 것입니다.

```
stamps.add(new Coin(...)); //"unchecked call" 경고를 내뱉는다.
```

 컬렉션에서 Coin 인스턴스를 다시 꺼내기 전에는 오류를 알아채지 못하게 됩니다.

```
for (Iterator i = stamps.iterator(); i.hasNext(); ) {
    Stamp stamp = (Stamp) i.next(); //ClassCastException을 던진다.
    stamp.cancel();
}

```

 이러한 코드의 문제점은 런타임에서 문제를 알아채게 됩니다. 이는 원인을 제공한 코드가 해당 시점에 물리적으로 상당히 떨어져 있을 가능성이 커지게 합니다.

**매개변수화된 컬렉션 타입을 통해 타입 안정성 확보**

```
private final Collection<Stamp> stamps = ...;
```

 위와 같이 매개변수화 타입을 사용하면 컴파일러가 컬렉션에 어떤 타입을 넣어야 하는지 인지할 수 있게 됩니다. 컴파일러는 컬렉션에서 원소를 꺼내는 모든 곳에 보이지 않는 형변환을 추가하여 절대 실패하지 않음을 보장합니다(컴파일러 경고가 나지 않았고 경고를 숨기지도 않았을 경우).

#### 로 타입의 호환성 문제

** 로타입은 제네릭이 안겨주는 안전성과 표현력을 모두 잃게 됩니다.** 이러한 문제점에도 불구하고 로 타입이 존재하는 이유는 호환성 때문입니다. 자바가 제네릭을 지원하기 전에는 로 타입으로 개발된 코드가 상당히 많았습니다. 그래서 기존 코드를 모두 수용하면서 제네릭을 사용하는 새로운 코드와도 호환이 되도록 하려면 로 타입을 계속해서 돌아가도록 헤야만 했습니다. 이러한 호환성을 위해서 로 타입을 지원하고 제네릭 구현에는 **소거(erasure) 방식**을 사용하기로 한 것입니다. 실제로 런타임에는 제네릭 타입이 사라지고 로 타입을 통해 코드가 동작하게 됩니다.

#### 로 타입인 List와 매개변수화 타입인 List<Object>의 차이

 List는 제네릭 타입에서 완전히 발을 뺀 것이고, List<Object>는 모든 타입을 허용한다는 의사를 컴파일러에 명확히 전달한 것입니다.

 매개변수로 List를 받는 메서드에 List<String>을 넘길 수 있지만, List<Object>를 받는 메서드에는 넘길 수 없습니다. 이는 제네릭의 하위 타입 규칙 때문입니다. 즉, List<String>은 로 타입인 List의 하위 타입이지만, List<Object>의 하위 타입은 아닙니다. 그 결과, **List<Object> 같은 매개변수화 타입을 사용할 때와 달리 List 같은 로 타입을 사용하면 타입 안전성을 잃게 됩니다.**

```
public class Main {
    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();
        unsafeAdd(strings, Integer.valueOf(10));
        String s = strings.get(0); // 컴파일러가 자동으로 형변환 코드를 넣어준다.
    }

    private static void unsafeAdd(List list, Object o) {
        list.add(o);
    }
}

```

 위 코드는 컴파일은 되지만 로 타입인 List를 사용하여 타입 안정성을 잃었습니다. 이 프로그램이 실행된다면 strings.get(0)의 결과를 형변환하려 할 때 ClassCastException을 던집니다.

#### 로 타입과 비한정적 와일드카드 타입(unbounded wildcard type)

```
    private static int numElementsInCommon(Set s1, Set s2) {
        int result = 0;
        for (Object o1 : s1) {
            if (s2.contains(o1))
                result++;
        }
        return result;
    }
    
```

 위와 같은 코드는 아까 보았던 것과 마찬가지로 로 타입으로 인해 타입 안전성이 보장되지 않습니다. 실제 타입의 매개변수가 무엇인지 신경 쓰고 싶지 않다면 물음표(?)를 이용한 비한정적 와일드 카드를 사용하면 됩니다.

```
private static numElementsInCommon(Set<?> s1, Set<?> s2) { ... }
```

#### 비한정적 와일드카드 타입인 Set<?>와 로 타입인 Set의 차이

 와일드카드 타입은 안전하고, 로 타입은 안전하지 않습니다. 로 타입 컬렉션에는 아무 원소나 넣을 수 있으니 타입 불변식을 훼손하기 쉽습니다. 반면, **Collection<?>에는 (null 외에는) 어떤 원소도 넣을 수 없습니다.** 다른 원소를 넣으려 하면 컴파일 할 때 오류가 발생합니다. 즉, 컬렉션의 타입 불변식을 훼손하지 못하게 막을 수 있습니다.

 컬렉션에서 꺼낼 수 있는 객체의 타입을 전혀 할 수 없다는 제약을 피하고 싶다면 제네릭 메서드나 한정적 와일드카드 타입을 사용하면 됩니다.

#### 로 타입을 사용하는 경우

 class 리터럴에는 로 타입을 써야 합니다. 자바 명세는 class 리터럴에 매개변수화 타입을 사용하지 못하게 했습니다(배열과 기본 타입은 허용). 예를 들어 List.class, String\[\].class, int.class는 허용되고 List<String>.class와 List<?>.class는 허용하지 않습니다.

 런타임에는 제네릭 타입 정보가 지워지므로 instanceof 연산자는 비한정적 와일드카드 타입 이외의 매개변수화 타입에는 적용할 수 없습니다. 그리고 로 타입이든 비한정적 와일트카드 타입이든 instanceof는 완전히 똑같이 동작하므로 <?>와 같이 아무런 역할 없이 코드를 지저분하게 만들지 않도록 로 타입을 쓰도록 합니다.

```
if (o instanceof Set) { // 로 타입
    Set<?> s = (Set<?>) o: //와일드카드 타입
    //...
}

```

 o 타입이 Set임을 확인한 다음 와일드카드 타입인 Set<?>으로 형변환해야 합니다. 이는 검사 형변환(checked cast)이므로 컴파일러 경고가 뜨지 않습니다.

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
