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
