# Item 48. 스트림 병렬화는 주의해서 적용하라

## 동시성 프로그래밍

  자바 8부터 parallel() 메서드만 한 번 호출하면 파이프라인을 병렬 실행할 수 있는 스트림을 지원하기 시작했습니다.

 동시성 프로그래밍을 할 때는 안전성(safety)과 응답 가능(liveness) 상태를 유지해야 하는데, 병렬 스트림 파이프라인 프로그래밍에서도 다르지 않습니다.

```
import java.math.BigInteger;
import java.util.stream.Stream;

import static java.math.BigInteger.ONE;
import static java.math.BigInteger.TWO;

public class MersenneMain {

    public static void main(String[] args) {
        primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                //.parallel()
                .filter(mersenne -> mersenne.isProbablePrime(50))
                .limit(20)
                .forEach(System.out::println);
    }

    public static Stream<BigInteger> primes() {
        return Stream.iterate(TWO, BigInteger::nextProbablePrime);
    }
}

```

 위 코드는 메르센 소수를 출력하는 프로그램입니다. 여기서 만약 속도를 높이고자 스트림 파이프라인의 parallel() 주석을 지우고 병렬화 시킨다면 아무것도 출력하지 못하면서 CPU 점유율만 높아지는 상태가 무한히 계속됩니다(응답 불가; liveness failure).

 병렬화를 시켰는데도 불구하고 느려진 원인은 스트림 라이브러리가 이 파이프라인을 병렬화하는 방법을 찾아내지 못했기 때문입니다. 환경이 아무리 좋더라도 **데이터 소스가 Stream.iterate거나 중간 연산으로 limit를 쓰면 파이프라인 병렬화로는 성능 개선을 기대할 수 없습니다.**

** 파이프라인 병렬화는 limit를 다룰 때 CPU 코어가 남는다면 원소를 몇 개 더 처리한 후 제한된 개수 이후의 결과를 버려도 아무런 해가 없다고 가정합니다.**

(단순하게 생각해서, 파이프라인 병렬화가 작업을 CPU 코어 수만큼 병렬로 수행한다고 생각해보겠습니다. 위의 병렬화한 코드를 쿼드 코어 시스템에서 수행하면 19번째 계산까지 마치고 마지막 20번째 계산이 수행하는 시점에는 CPU 코어 3개가 한가할 것입니다. 따라서 21, 22, 23번째 메르센 소수를 찾는 작업이 병렬로 시작되는데, 20번째 계산이 끝나더라도 이 계산들은 끝나지 않습니다. 각각 20번째 계산보다 2배, 4배, 8배의 시간이 더 필요하기 때문입니다.)

## 병렬화 효율이 좋은 자료구조

#### ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int 범위, long 범위

 대체로 **스트림의 소스가 위와 같을 때 병렬화의 효과가 가장 좋습니다.** 이 자료구조들은 모두 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어서 일을 다수의 쓰레드에 분배하기에 좋다는 특징이 있습니다.

 작업을 나누는 작업은 Pliterator가 담당하며, Spliterator 객체는 Stream이나 Iterable의 spliterator 메서드로 얻어올 수 있습니다.

 이 자료구조들은 원소들을 순차적으로 실행할 때의 참조 지역성(locality of reference)이 뛰어납니다. 이웃한 원소의 참조들이 메모리에 연속해서 저장되어 있다는 뜻입니다. 하지만 참조들이 가리키는 실제 객체가 메모리에서 서로 떨어져 있을 수 있는데, 그러면 참조 지역성이 나빠집니다.

 참조 지역성이 낮으면 스레드는 주 메모리에서 캐시 메모리로 전송되어 오기를 기다리며 대부분 시간을 멍하니 보내게 됩니다. 따라서 참조 지역성은 다량의 데이터를 처리하는 벌크 연산을 병렬화할 때 아주 중요한 요소로 작용합니다.

 **참조 지역성이 가장 뛰어난 자료구조는 기본 타입의 배열**입니다. 기본 타입 배열에서는 (참조가 아닌) 데이터 자체가 메모리에 연속해서 저장되기 때문입니다.

## 스트림 파이프라인의 명세 규약

 스트림을 잘못 병렬화하면 (응답 불가를 포함해) 성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나 예상 못한 동작이 발생할 수 있습니다. 결과가 잘못되거나 오동작하는 것은 안전 실패(safety failure)라고 합니다. 안전 실패는 병렬화한 파이프라인이 사용하는 mappers, filters, 혹은 프로그래머가 제공한 다른 함수 객체가 명세대로 동작하지 않을 때 벌어질 수 있습니다.

 Stream 명세는 이때 사용되는 함수 객체에 관한 엄중한 규약을 정의해놨습니다. 예를 들어 Stream의 reduce 연산에 건네지는 accumulator(누적기)와 combiner(결합기) 함수는 반드시 결합법칙을 만족하고(associative), 간섭받지 않고(non-interfering), 상태를 갖지 않아야(stateless) 합니다.

## 스트림 병렬화를 적용하기 전에 고려 사항

 스트림 병렬화는 오직 성능 최적화 수단임을 기억해야 합니다. 다른 최적화와 마찬가지로 변경 전후로 반드시 성능을 테스트하여 병렬화를 사용할 가치가 있는지 확인해야 합니다. 이상적으로는 운영 시스템과 흡사한 환경에서 테스트하는 것이 좋습니다.

 보통은 병렬 스트림 파이프라인도 공통의 포크-조인풀에서 수행되므로(즉, 같은 쓰레드 풀을 사용하므로), 잘못된 파이프라인 하나가 시스템의 다른 부분의 성능에까지 악영향을 줄 수 있음을 유념해야 합니다.

## 무작위 수들로 이뤄진 스트림의 병렬화

 무작위 수들로 이뤄진 스트림을 병렬화 할 때, ThreadLocalRandom(혹은 구식인 Random)보다는 SplittableRandom 인스턴스를 이용하는 것이 좋습니다. SplittableRandom은 정확히 이럴 때 쓰고자 설계된 것이라 병렬화하면 성능이 선형으로 증가합니다.

 ThreadLocalRandom은 단일 쓰레드에서 쓰고자 만들어졌습니다. 병렬 스트림용 데이터 소스로도 사용할 수는 있지만 SplittableRandom만큼 빠르지는 않습니다.

 Random은 모든 연산을 동기화하기 때문에 병렬 처리하면 최악의 성능을 보입니다.

---

## 참고자료

[www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)
