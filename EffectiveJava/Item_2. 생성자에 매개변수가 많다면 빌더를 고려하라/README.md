# Item 2. 생성자에 매개변수가 많다면 빌더를 고려하라

 정적 팩토리와 생성자에는 선택적 매개변수가 많을 때 적절히 대응하기 어렵다는 단점이 있습니다. 이 경우 프로그래머는 **점층적 생성자 패턴**을 사용할 수도 있지만, 매개변수 개수가 많아지면 코드를 작성하거나 읽기 어렵습니다.

**점층적 생성자 패턴**

```
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

 이와 같은 코드는 각 값의 의미가 무엇인지 헷갈리고, 실수로 매개변수의 순서가 바뀌어도 컴파일러가 알아채지 못하여 런타임 시에 엉뚱한 동작을 하게 될 위험이 있습니다.

 다른 방법으로는 **자바빈즈 패턴(JavaBeans pattern)**이 있습니다. 매개변수가 없는 생성자로 객체를 만든 후, setter 메서드들을 호출하여 원하는 매개변수의 값을 설정하는 방식입니다.

**자바빈즈 패턴**

```
public class NutritionFactsForSetter {
    private int servingSize = -1;
    private int servings = -1;
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public void setServingSize(int servingSize) {
        this.servingSize = servingSize;
    }

    public void setServings(int servings) {
        this.servings = servings;
    }

    public void setCalories(int calories) {
        this.calories = calories;
    }

    public void setFat(int fat) {
        this.fat = fat;
    }

    public void setSodium(int sodium) {
        this.sodium = sodium;
    }

    public void setCarbohydrate(int carbohydrate) {
        this.carbohydrate = carbohydrate;
    }
}
```

 자바빈즈 패턴은 점층적 생성자 패턴의 단점들이 더 이상 보이지 않지만, 객체 하나를 생성하려면 여러 메서드가 호출되어야 하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 됩니다. 일관성이 무너지는 문제 때문에 자바빈즈 패턴에서는 클래스를 불변으로 만들 수 없으며 스레드 안전성을 얻으려면 프로그래머가 별도의 작업을 해주어야 합니다.

#### 빌더 패턴(Builder pattern)

 빌더 패턴은 클라이언트가 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자(혹은 정적 팩토리)를 호출해 얻는 빌더 객체를 통해 일종의 세터 메서드들로 원하는 선택 매개변수들을 설정하고, build 메서드를 호출하여 필요한 객체를 얻을 수 있는 패턴입니다.

```
public class NutritionFactsForBuilder {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        //필수 매개변수
        private final int servingSize;
        private final int servings;

        //선택 매개변수
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFactsForBuilder build() {
            return new NutritionFactsForBuilder(this);
        }
    }

    private NutritionFactsForBuilder(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}

```

 인스턴스는 메서드 체이닝을 통해 아래와 같이 생성이 가능합니다.

```
NutritionFactsForBuilder foo = new NutritionFactsForBuilder.Builder(240, 8)
                .calories(100).sodium(100).carbohydrate(100).build();
```

 빌더 패턴은 빌더 하나로 여러 객체를 순회하면서 만들 수 있고, 빌더에 넘기는 매개변수에 따라 다른 객체를 유연하게 만들 수 있습니다. 하지만 단점도 있습니다. 객체를 만들려면, 그에 앞서 빌더부터 만들어야 합니다. 빌더 생성 비용은 크지 않지만 성능에 민감한 상황에서는 문제가 될 수 있습니다. 또한 점증적 생성자 패턴보다 코드가 장황해집니다.

---

## 참고자료

[http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LEa&Kc=)