# 2.2 동작 파라미터화

동작파라미터화를 이용하면 자구 바뀌는 요구사항에 효과적으로 대응할 수 있다고 했다.
앞에서 파라미터를 추가하는 방법이 아니라 변화하는 요구사항에 좀 더 유연하게 대응할 수 있는 방법이 필요하다고 하였다.

유연하게 대응할 수 있는 방법으로 참 거짓을 반환하는 프레디케이트를 이용할 수 있다.

```java

public interface ApplePredicate{
  boolean test(Apple a);
}

// 무거운 사과만 선택
public class AppleHeavyWeightPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
}

//녹색 사과만 선택
public class AppleGreenColorPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return GREEN.equals(apple.getColor());
    }
}

```

```java

//필터링 방법으로 코드를 수정
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
    List<Apple> result = new ArrayList<>();
    for(Apple apple: inventory) {
        if(p.test(apple: inventory)) {
            result.add(apple);
        }
    }
    return result;
}

```



이러한 방식을 사용하면 조건에 따라 filter 메서드를 선택할 수 있다.
이러한 방식을 전략 디자인 패턴이라 부른다.

전략(Strategy) 패턴:

알고리즘을 객체로 캡슐화하고, 이를 컨텍스트에서 동적으로 바꾸어 실행할 수 있게 합니다. 
전략 패턴은 인터페이스를 통해 알고리즘을 정의하고, 다양한 전략 클래스로 구현하며, 이를 실행 시간에 바꾸어 사용할 수 있습니다.


아래는 실제 만들어진 예제 코드를 사용하는 코드이다.
```java

import java.util.Arrays;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Apple> apples = Arrays.asList(
                new Apple("GREEN", 100),
                new Apple("RED", 160),
                new Apple("GREEN", 200),
                new Apple("YELLOW", 120),
                new Apple("RED", 180)
        );

        // 무거운 사과만 선택하기 위한 전략을 설정한다.
        List<Apple> heavyApples = filterApples(apples, new AppleHeavyWeightPredicate());
        System.out.println("Heavy Apples: " + heavyApples);

        // 녹색 사과만 선택하기 위한 전략을 설정한다.
        List<Apple> greenApples = filterApples(apples, new AppleGreenColorPredicate());
        System.out.println("Green Apples: " + greenApples);
    }
}


```

이제 요구사항이 추가되더라도 쉽게 필터링 기능을 적용할 수 있다.

```java

public class AppleRedAndHeavyPredicate implements ApplePredicate {
    public bolean test(Apple apple) {
        return RED.equals(apple.getColor())
            && apple.getWeight() > 150 ;
            
            
 ...
 
         // 150 그램 이상이면서 빨간색 사과만 필터링하도록 한다.
        List<Apple> heavyAndRedApples = filterApples(apples, new AppleHeavyWeightPredicate());
        System.out.println("Heavy AND RED Apples: " + heavyAndRedApples);

```

지금까지 살펴본 것을 토대로 컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있는 것을 확인하였다.
이를 통해 유연한 API를 만들 수 있다.

하지만 여러 클래스를 구현하면서 인스턴스화 하는 과정이 거추장스럽게 느껴질 수 있으며,
이를 람다를 사용하여 간소화가 가능하다.


