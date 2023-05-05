# 9.4 디버깅

디버깅 시 두가지를 확인해야 한다.
- 스택 트레이스
- 로깅

하지만 람다 표현식과 스트림은 기존의 디버깅을 무력화 한다.   
이를 디버깅 하는 방법을 살펴보자

## 9.4.1 스택 트레이스 확인
### 람다와 스택 트레이스
- 람다 표현식은 이름이 없기 때문에 복잡한 스택 트레이스가 생성된다.   
- 메서드 참조를 사용해도 스택 트레이스에는 메서드 명이 나타나지 않고, 임의의 이름을 만들어낸다.   

메서드 참조를 사용하는 클래스와 같은 곳에 선언되어있는 메서드를 참조할 때는 메서드 참조 이름이 스택 트레이스에 나타난다.
```java
public class Debugging {
  public static void main(String[] args) {
    List<Integer> numbers = Arrays.asList(1,2,3);
    numbers.stream().map(Debugging::devideByZero).forEach(System.out::println);
  }
  public static int divideByZero(int n) {
    return n / 0;
  }
}
```

위 처럼 작성한다면 스택 트레이스에 Debugging.divideByZero가 출력된다.

## 9.4.2 정보 로깅

```java
numbers.stream()
  .map(x -> x + 17)
  .filter(x -> x % 2 == 0)
  .limit(3)
  .forEach(System.out::println);

// 출력
// 20
// 22
```

위 연산에서 forEach를 호출하는 순간 전체 스트림이 소비된다.   
스트림 파이프라인에 적용된 각각의 연산(map, filter, limit)이 어떤 결과를 도출하는지 확인하려면 어떻게 해야할까?

바로 peek이라는 스트림 연산을 활용하면 된다.   
peek은 스트림의 각 요소를 소비한 것처럼 동작하지만, forEach처럼 실제로 소비하지는 않는다.

```java
numbers.stream()
  .peek(x -> System.out.println("from stream: " + x))
  .map(x -> x + 17)
  .peek(x -> System.out.println("after map: " + x))
  .filter(x -> x % 2 == 0)
  .peek(x -> System.out.println("after filter: " + x))
  .limit(3)
  .peek(x -> System.out.println("after limit: " + x))
  .collect(toList());

```

이제 각 단계별로 결과 값을 출력할 수 있다.
