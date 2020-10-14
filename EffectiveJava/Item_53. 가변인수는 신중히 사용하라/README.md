# Item 53. 가변인수는 신중히 사용하라

 가변인수 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있습니다. 가변인수 메서드를 호출하면, 인수의 개수와 길이가 같은 배열을 만들고 인수들을 배열에 저장하여 가변인수 메서드에 건네줍니다.

**가변인수를 통해 인수들의 합을 반환하는 메서드**

```
static int sum(int... args) {
    int sum = 0;
    for (int arg : args)
        sum += arg;
    return sum;
}
```

#### 인수가 1개 이상이어야 하는 경우

 아래의 코드는 가변인수를 받아서 최솟값을 찾는 메서드입니다.

```
static int min(int... args) {
    if (args.length == 0)
        throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
    int min = args[0];
    for (int i = 1; i < args.length; i++)
        if (args[i] < min)
            min = args[i];
    return min;
}
```

 이 방식에는 인수를 0개만 넣어 호출하면 컴파일타임이 아닌 런타임에 실패한다는 문제점이 있습니다. args 유효성 검사를 명시적으로 해야 하고, min의 초기값을 Integer.MAX\_VALUE로 설정하지 않고는 (더 명료한) for-each 문도 사용할 수 없습니다.

 이러한 문제를 해결할 수 있는 더 나은 방법은 다음 코드처럼 매개변수를 2개 받도록 하는 것입니다. 첫 번째 매개변수는 평범한 매개변수를 받고, 가변인수는 두 번째로 받으면 앞서의 문제가 해결됩니다.

```
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs)
        if (arg < min)
            min = arg;
    return min;
}
```

#### 다중정의를 이용하는 방법

```
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```

 성능이 민감한 상황에서는 가변인수 메서드가 걸림돌이 될 수 있습니다. 가변인수 메서드는 호출할 때마다 배열을 새로 할당하고 초기화하기 때문입니다. 위의 패턴은 가변인수 생성 비용을 감당할 수는 없지만 가변인수의 유연성이 필요할 때 사용할 수 있는 패턴입니다.

 예를 들어 해당 메서드 호출의 95%가 인수를 3개 이하로 사용한다고 생각해보겠습니다. 그렇다면 위와 같이 인수가 0개인 것부터 4개인 것까지, 총 5개를 다중정의 합니다. 그리고 마지막 다중정의 메서드가 인수 4개 이상인 5%의 호출을 담당하는 것입니다. 따라서 메서드 호출 중 단 5%만이 이 배열을 생성합니다.

EnumSet의 정적 팩토리도 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화 합니다.

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
