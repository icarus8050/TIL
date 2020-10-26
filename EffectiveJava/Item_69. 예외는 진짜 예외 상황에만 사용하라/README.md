# Item 69. 예외는 진짜 예외 상황에만 사용하라

#### 예외의 사용

 **예외는 (그 이름이 말해주듯) 오직 예외 상황에서만 써야 합니다. 절대로 일상적인 제어 흐름용으로 쓰여선 안됩니다.**

 잘 설계된 API라면 클라이언트가 정상적인 제어 흐름에서 예외를 사용할 일이 없게 해야 합니다. 특정 상태에서만 호출할 수 있는 '상태 의존적' 메서드를 제공하는 클래스는 '상태 검사' 메서드도 함께 제공해야 합니다. Iterator 인터페이스의 next와 hasNext가 각각 상태 의존적 메서드와 상태 검사 메서드에 해당합니다.

**상태 검사 메서드를 이용한 for 관용구 코드**

```
for (Iterator<Foo> i = collection.iterator(); i.hasNext(); ) {
    Foo foo = i.next();
    //Something job...
}
```

(for-each도 내부적으로 hasNext를 사용합니다.)

 Iterator가 hasNext를 제공하지 않았다면 그 일을 클라이언트가 대신해야만 합니다.

```
// 컬렉션을 이런 식으로 순회해서는 안됩니다!
try {
    Iterator<Foo> i = collection.iterator();
    while (true) {
        Foo foo = i.next();
        // Something job...
    }
} catch (NoSuchElementException e) {
}
```

 반복문에 예외를 사용하여 순회하면 코드가 장황해지고, 속도도 느립니다. 그리고 엉뚱한 곳에서 발생한 버그를 숨길 가능성도 있습니다.

#### 상태 검사 메서드 대신 사용할 수 있는 선택지

 상태 검사 메서드를 사용하는 방법 말고도 빈 옵셔널이나 null 같은 특수한 값을 반환하는 방법도 있습니다.

**상태 검사 메서드, 옵셔널, 특정 값 중 하나를 선택하는 지침**

-   외부 동기화 없이 여러 스레드가 동시에 접근할 수 있거나 외부 요인으로 상태가 변할 수 있다면 옵셔널이나 특정 값을 사용합니다. 상태 검사 메서드와 상태 의존적 메서드 호출 사이에 객체의 상태가 변할 수 있기 때문입니다.
-   성능이 중요한 상황에서 상태 검사 메서드가 상태 의존적 메서드의 작업 일부를 중복 수행한다면 옵셔널이나 특정 값을 선택합니다.
-   다른 모든 경우엔 상태 검사 메서드 방식이 조금 더 낫다고 할 수 있습니다. 상태 검사 메서드 호출을 깜빡 잊었다면 상태 의존적 메서드가 예외를 던져 버그를 확실히 드러내지만 특정 값은 검사하지 않고 지나쳐도 발견하기가 어렵습니다(옵셔널에는 해당하지 않음).

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
