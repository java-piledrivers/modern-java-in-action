# 2.1 변화하는 요구사항에 대응

**우리가 어떤 상황에서 일을 하든 소비자 요구사항은 항상 바뀐다.**

→ 1. 개발자는 새로 추가한 기능은 쉽게 구현할 수 있어야 하며 장기적인 관점에서 유지보수가 쉬워야 한다 

→ 2. 엔지어링적인 비용이 가장 최소하 될 수 있으면 더 좋다.

1, 2를 충족시킬 수 있는 해결방법이 `동작 파라미터화`이다.

<br>
`동작 파라미터화`란 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록을 의미한다.

예를 들어 나중에 실행될 메서드의 인수로 코드 블록을 전달할 수 있다. 즉, 코드 블록에 따라 메서드의 동작이 파라미터화 된다. 

<br>
변화에 대응하는 코드를 구현하는 것은 어려운 일이다. 그 이유는 예시를 통해서 알아보자.

**예시 : 기존 농장 재고목록 애플리케이션에서 사과에 대한 필터링** 

<br></br>
### 첫 번째 상황 녹색 사과만 필터링

```java
enum color { RED, GREEN }

public static List<Apple> filterGreenApples(List<Apple> inventory){
        List<Apple> result = new ArrayList<>();
        for (Apple apple : inventory) {
            **if (GREEN.eqauls(apple.getColor())) {**
                result.add(apple);
            }
        }
        return result;
    }

```
<br></br>
### 두 번째 상황) 빨간 사과만 필터링

단순한 방법 : filterReadApples 메서드를 만들고 RED에 대해서 필터링을 할 것이다.

하지만, 농부가 옅은 녹색, 어두운 빨간색, 노란색 등 요구사항이 변경할 때마다 메서드를 반복해서 복사해야 하기 때문에 적절하지는 않다.

이런 상황에서는 다음과 같은 좋은 규칙이 있다.

**`거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화 한다.`**

<br></br>
### 파라미터화)

어떻게 해야 filterGreenApples의 코드를 반복 사용하지 않고 다양한 색깔의 사과를 구현할 수 있을까?

이것 또한 단순하게 생각해보면 파라미터에 색깔 추가하면 된다.

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color){
        List<Apple> result = new ArrayList<>();
        for (Apple apple : inventory) {
            if (color.eqauls(apple.getColor())) {
                result.add(apple);
            }
        }
        return result;
    }

```

이제 농부의 요구사항은 다음과 같은 코드를 통해 해결할 수 있다.

```java
List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
List<Apple> redApples = filterApplesByColor(inventory, RED);
```

<br>
그런데, 농부가 갑자기 색 이외에도 가벼운 사과와 무거운 사과로 구분할 수 있으면 좋겠습니다. 보통 무게가 150그램 이상인 사과가 무거운 사과입니다. 라고 요구한다.

여기서 눈치챘겠지만 소비자의 요구사항은 늘 변한다는 것을 알 수 있다. 

그러면 이러한 요구사항 또한 weight 파라미터를 추가하면 충분히 해결할 수 있을 것이다.

<br>
하지만 목록을 검색하고, 각 사과에 필터링 조건을 적용하는 부분의 코드가 색 필터링 코드와 대부분 중복될 것이다. 이는 소프트웨어 공학의 DRY(don’t repeat yourself, 같은 것을 반복하지 말것) 원칙을 어기는 것이다.

<br>
탐색 과정을 고쳐서 성능을 개선할면 무슨 일이 일어나야 할까? 한 줄이 아니라 메서드 전체 구현을 고쳐야 한다. 즉, 엔지니어링적으로 비싼 대가를 치러야 한다.

<br></br>
색과 무게를 filter라는 메서드로 합치는 방법도 있다. 그러면 어떤 기준으로 사과를 필터링할지 구분하는 또 다른 방법이 필요하다. 따라서 색이나 무게 중 어떤 것을 기준으로 필터링할지 가리키는 플래그를 추가할 수 있다(하지만 실전에서는 절대 이 방법을 사용하지 말아야 한다. 뒤에서 설명)

### 가능한 모든 속성으로 필터링

```java
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag){
        List<Apple> result = new ArrayList<>();
        for (Apple apple : inventory) {
            if (flag && color.eqauls(apple.getColor()) ||) {
                (!flag && apple.getWeight() > weight))
                result.add(apple);
            }
        }
        return result;
    }
```

이제 위 메서드를 사용할 수 있다

```java
List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);
List<Apple> heavyApples = filterApples(inventory, null, 150, false);
```

과연 이게 좋은 코드일까?

대체 true와 false는 뭘 의마하는 걸까? ( 이것은 final을 통해 이름을 정의하면 어느정도 해결은 된다)

게다가 앞으로 요구사항이 바뀌었을 때 유연하게 대응할 수도 없다. 예를 들어 사과의 크기, 모양, 출하지 등으로 사과를 필터링 하고 싶다면 어떻게 될까? 

결국 여러 중복된 필터 메섣르르 만들거나 아니면 모든 것을 처리하는 거대한 하나의 필터 메서드를 구현해야 한다.

<br>
문제가 잘 정의되어 있는 사황에서는 위처럼 해도 되기는 한다. 하지만 filterApples에 어떤 기준으로 사과를 필터링할 것인지 효과적으로 전달할 수 있다면 더 좋을 것이다.
<br></br>
이는 "동적 파라미터화"가 해결 해준다.


# 2.2 동작 파라미터화

동작파라미터화를 이용하면 자주 바뀌는 요구사항에 효과적으로 대응할 수 있다고 했다.

앞에서 파라미터를 추가하는 방법이 아니라 변화하는 요구사항에 좀 더 유연하게 대응할 수 있는 방법이 필요하다고 하였다.

유연하게 대응할 수 있는 방법으로 참 거짓을 반환하는 프레디케이트(함수형 인터페이스)를 이용할 수 있다.

* 함수형 인터페이스는 추상 메소드만 가지며, 람다 표현식과 함께 사용할 수 있다.
- 다른 함수형 인터페이스들
https://jjingho.tistory.com/80

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
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        List<Apple> apples = Arrays.asList(
                new Apple("GREEN", 100),
                new Apple("RED", 160),
                new Apple("GREEN", 200),
                new Apple("YELLOW", 120),
                new Apple("RED", 180)
        );

        Scanner scanner = new Scanner(System.in);
        int choice = 0;
        while (choice != 3) {
            System.out.println("전략 패턴 선택 (1-Heavy Apples, 2-Green Apples, 3-Exit):");
            choice = scanner.nextInt();

            List<Apple> filteredApples;

            switch (choice) {
                case 1:
                    filteredApples = filterApples(apples, new AppleHeavyWeightPredicate());
                    System.out.println("Heavy Apples: " + filteredApples);
                    break;
                case 2:
                    filteredApples = filterApples(apples, new AppleGreenColorPredicate());
                    System.out.println("Green Apples: " + filteredApples);
                    break;
                case 3:
                    System.out.println("Exiting...");
                    break;
                default:
                    System.out.println("올바르지 않은 값이 입력되었습니다.");
            }
        }
    }
}

```

이제 요구사항이 추가되더라도 쉽게 필터링 기능을 적용할 수 있다.

```java

public class AppleRedAndHeavyPredicate implements ApplePredicate {
    public bolean test(Apple apple) {
        return RED.equals(apple.getColor())
            && apple.getWeight() > 150 ;
    }        
}            
 ...
 
// 150 그램 이상이면서 빨간색 사과만 필터링하도록 한다.
List<Apple> heavyAndRedApples = filterApples(apples, new AppleHeavyWeightPredicate());
System.out.println("Heavy AND RED Apples: " + heavyAndRedApples);

```

지금까지 살펴본 것을 토대로 컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있는 것을 확인하였다.
이를 통해 유연한 API를 만들 수 있다.

하지만 여러 클래스를 구현하면서 인스턴스화 하는 과정이 거추장스럽게 느껴질 수 있으며,
이를 람다를 사용하여 간소화가 가능하다.


# 2.3 복잡한 과정 간소화

앞서 동작 파라미터를 사용하여 추상적 조건으로 필터링 해보았는데, 이때 `filterApples` 메서드로 새로운 동작을 전달하기 위해서는 `ApplePredicate` 
인터페이스를 구현하는 여러 클래스를 정의한 뒤 인스턴스화해야 했다.


```java

public interface ApplePredicate{
  public boolean test(Apple a);
}


public class FilteringApples {
  public static void main(String[] args) {
  
    List<Apple> inventory = Arrays.asList(new Apple(80, "green"), new Apple(155, "green"), new Apple(120, "red"));
  
    List<Apple> greenApples = filter(inventory, new AppleColorPredicate());
    List<Apple> heavyApples = filter(inventory, new AppleWeightPredicate());
  }  
  
  
  public static List<Apple> filter(List<Apple> inventory, ApplePredicate p){
    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory){
      if(p.test(apple)){
        result.add(apple);
      }
    }
    return result;
  }


  public class AppleWeightPredicate implements ApplePredicate{
    public boolean test(Apple apple){
      return apple.getWeight() > 150; 
    }
  }
  public class AppleColorPredicate implements ApplePredicate{
    public boolean test(Apple apple){
      return "green".equals(apple.getColor());
    }
  }
}
```

다시 설명해보자면 이 코드의 주요 로직은 test 메서드 내부의 동작이다. 이를 구현하기 위해서 ApplePredicate 인터페이스를 구현한 클래스를 만들었고, 인스턴스화하였다.  
보다시피 로직과 관련없는 코드가 포함되어있다.  

이러한 문제를 해결해주는 것이 익명 클래스이다.

## 익명 클래스 Anonymous class

* 자바의 지역 클래스(Local class, 블럭 내부에 선언된 클래스)와 비슷한 개념
* 클래스 이름을 가지고 있지 않음
* 선언과 인스턴스화가 동시에 가능


```java

List<APple> redApples = filterApples(inventory, new ApplePredicate() {
  public boolean test(Apple apple) {
    return "red".equals(apple.getColor());
    }
  });
  
```

블럭 내부에 구현했었던 클래스를 파라미터에 선언을 하였다. 이름이 없는 익명 클래스이다.  

익명 클래스는 자바 스윙이나 AWT 같은 GUI 애플리케이션에서 이벤트 핸들러 객체를 구현할 때 사용한다. 하지만 익명 클래스도 부족한 점이 있다.  
1. 여전히 많은 코드량
2. 가독성 저하
  
  
```java
public class MeaningOfThis {
    public final int value = 4;

    public void doIt() {
        final int value = 6;
        Runnable r = new Runnable() {
            public final int value = 5;
            public void run() {
                int value = 10;
                System.out.println(this.value);
            }
        };
        r.run();
    }
    
    public static void main(String[] args) {
      MeaningOfThis m = new MeaningOfThis();
      m.doIt();
    }
}

```
위의 예제에서 어떤 값이 출력될까?  

<br>

정답은 5이다.  
this.value 시점의 this 가 MeaningOfThis의 value 가 아니라 Runnable의 value 를 참조하고 있기 때문이다.  

코드의 가독성을 높혀줄 방법은 바로 아래에서 소개할 lambda 이다.
---


## lambda 

```java
List<Apple> result = filterApples(inventory, (Apple apple) -> "red".equals(apple.getColor()));;
```

코드가 깔끔해졌다.  
람다도 클래스와 익명클래스처럼 유연하지만 보다 더 간결하다.  


이제 람다를 사용하는 방법까지 살펴봤으니 추상화에 다가가보자.  
지금까지 `filterApples` 메서드는 오직 `Apple` 에 대해서만 동작하였다. 리스트를 추상화한다면 다른 과일뿐만 아니라 문자열이나정수에 대해서도 필터링할 수 있다.  

```java
public interface Predicate<T> {
  boolean test(T t);
}

public static <T> List <T> filter(List<T> list, Predicate<T> p) {
  List<T> result = new ArrayList<>();
  for (T e: list) {
    if(p.test(e)) {
      result.add(e);
      }
    }
  return result;
}

List<Apple> redApple = filter(inventory, (Apple apple) -> "red".equals(apple.getColor()));
List<Integer> evenNumbers = filter(number, (Integer i) -> i % 2 == 0);
```

