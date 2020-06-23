# Explain 실행 계획

## explain 명령어

 select문을 어떠한 방식으로 수행하여 데이터를 가져올 것인지에 대한 실행 계획을 출력하는 명령어입니다. 사용 방법은 질의하고자 하는 select 쿼리 앞에 explain 키워드를 붙여서 사용합니다. (ex : explain select \* from table)

![explain](./images/explain.png)

 실행 계획은 아래와 같은 형식으로 표현됩니다. 이제 각 컬럼별로 어떤 의미를 나타내는지 살펴보겠습니다.

### id

 select 쿼리별로 부여되는 식별자 값입니다. 만약 하나의 select 문에서 여러 개의 테이블을 조인하면 조인되는 테이블의 개수만큼 실행 계획 레코드가 출력되지만 같은 id가 부여됩니다. 하지만 쿼리 문장이 서로 다른 select 문(쿼리 안에 서브쿼리 구성 등..)으로 구성되어 있으면 각 레코드의 id 컬럼이 각기 다른 값을 부여받게 됩니다.

### select\_type

 select 문의 유형을 나타냅니다.

-   SIMPLE
    -   서브쿼리나 UNION이 없는 가장 단순한 형태의 테이블을 말합니다.
-   PRIMARY
    -   가장 바깥의 select문을 말합니다.
-   DERIVED
    -   from 절에 사용된 서브쿼리로부터 발생한 임시 테이블을 말합니다. 임시 테이블은 메모리에 저장될 수도 있고, 디스크에 저장될 수도 있습니다. **일반적으로 메모리에 저장하는 경우에는 성능에 큰 영향을 미치지 않지만, 데이터의 크기가 커서 임시 테이블을 디스크에 저장할 경우 성능이 떨어지게 됩니다.**
-   SUBQUERY
    -   from 절 이외에서 사용되는 서브쿼리를 의미합니다. 서브쿼리는 사용되는 위치에 따라 각각 다른 이름을 가지고 있습니다.
        -   중첩된 쿼리 (Nested Query) : select 되는 컬럼에 사용된 서브쿼리를 말합니다.
        -   서브 쿼리 (Sub Query) : where 절에서 사용된 경우에는 일반적으로 그냥 서브 쿼리라고 말합니다.
        -   파생 테이블 (Derived) : from 절에서 사용된 서브 쿼리를 말합니다.
-   DEPENDENT SUBQUERY
    -   서브 쿼리가 바깥쪽 select 쿼리에서 정의된 컬럼을 사용하는 경우를 말합니다. **이는 서브 쿼리가 먼저 실행되지 못하고 서브 쿼리가 외부 쿼리 결과에 의존적이기 때문에 전체 쿼리의 성능을 느리게 만듭니다.** 서브 쿼리가 외부의 쿼리의 값을 전달받고 있는지 검토해서, 가능하다면 외부 쿼리와의 의존도를 제거하는 것이 좋습니다.
-   UNCACHEABLE SUBQUERY
    -   쿼리의 from 절 이외의 부분에서 사용되는 서브 쿼리는 가능하면 MySQL 옵티마이저가 캐싱하여 최대한 재사용 될 수 있게 유도합니다. 하지만 사용자 변수나 일부 함수가 사용된 경우에는 이러한 캐시 기능을 사용할 수 없게 만듭니다. 이런 실행 계획이 사용된다면 사용자 변수를 제거하거나 다른 함수로 대체해서 사용할 수 있을지 검토해보는 것이 좋습니다.
-   UNION
    -   union으로 결합하는 단위 select 쿼리 가운데 첫 번째를 제외한 두 번째 이후의 단위 select 쿼리의 select\_type은 UNION으로 표시됩니다.
-   DEPENDENT UNION
    -   UNION select\_type과 같지만 union으로 결합된 단위 쿼리가 바깥쪽 쿼리에 의존적이어서 외부의 영향을 받고 있는 경우를 말합니다.

---

## 참고자료

[https://denodo1.tistory.com/306](https://denodo1.tistory.com/306)

[https://12bme.tistory.com/168](https://12bme.tistory.com/168)

[https://idea-sketch.tistory.com/48](https://idea-sketch.tistory.com/48)