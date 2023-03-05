## 1.1 역사의 흐름은 무엇인가?

- 가장 큰 변화는 자바 8
  - 자바 9, 10 큰 변화는 없었음
- 예를 들면 아래와 같은 코드
```java
Collections.sort(inventory, new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeigt().compareTo(a2.getWeight());
    }
})
```
```java
inventory.sort(comparing(Apple::getWeight));
```

### 병렬 실행 자바의 흐름

- 자바 8 이전에는 하나만 사용하여 다른 코어는 유휴 idle 상태로 둠
- 자바 8 이전에 여러 cpu 를 사용하려면 스레드를 사용해야함
  - 관리가 어렵고 많은 문제 발생
- **이런 병렬 실행환경을 쉽게 관리하고 에러가 덜발생하는 방향으로 진화**
  - 자바 1.0: 스레드와 락, 메모리 모델까지 지원 -> 활용하기 힘들었음
  - 자바 5: 스레드 풀, 병렬 실행 컬렉션(concurrent collection)
  - 자바 7: 포크/조인 프레임워크 제공 Q) 무슨 프레임워크?
  - 자바 8: 병렬 실행을 새롭고 단순한 방법으로 제공
    - 책에서 주요적으로 다룸
  - 자바 9: 리액티브 프로그래밍 병렬 실행 기법
    - 고성능 병렬 시스템에서 RxJava(리액티브 스트림 툴킷)을 사용한다고 하는데 이를 표준방식으로 제공

### 자바 8

**간결한 코드, 멀티코어 프로세서의 쉬운 할용**이 핵심이다.
간단히 기능이 어떤게 있는지 살펴보자.

스트림이 데이터베이스에서 표현식을 처리하는 것처럼 병렬 연산을 한다고 한다.

> Q)쿼리에 논리적 쿼리 실행 순서가 있는 것처럼 스트림도 똑같을까?

아마 데이터베이스가 쿼리의 실행 계획을 생성하는 것처럼 스트림또한  
책의 표현대로 "최적의 저수준 실행 방법"을 선택하여 동작하는 것으로 보인다.   
또한 스트림을 사용하면 비용이 비싼 synchronized 를 사용하지 않아도 된다.   

조금 다른 관점에서 보면 결국 스트림 API 가 추가되어서 코드를 전달하는 간결 기법(메서드 참조와 람다),   
인터페이스의 디폴트 메서드가 존재하는 것이 아닌가라는 의문이 생김.

그렇게 생각하면 "코드를 전달하는 간결 기법"의 활용성을 제한하는 생각이다.

에를 들어 아래와 같이 약간 다른 동작을 하는 메소드가 있다고 생각해보자.
```java
void print0() {
    System.out.println("hello");
}

void print1() {
    System.out.println("world");
}
```
똑같이 print 해주는 메소드이기 때문에 하나의 메소드로 합치는 것이 바람직 할 수 있다

익명 클래스를 알고 있다면 아래처럼 동작 파라미터화(behavior paramaterization)하여 구현할 수 있을 것이다.
```java
class Main {
    interface Foo {
        String pickWord(int val);
    }

    public static void print(int val Foo foo) {
        String result = foo.pickWord(val);
        System.out.println(result);
    }

    public static void main(String[] args) {
        int val = Integer.parseInt(args[0]);
        print(val, new Foo() {
            @Override
            public String pickWord(int val) {
                switch (val) {
                    case 0:
                        return "hello";
                    case 1:
                        return "world";
                    default:
                        return "error";
                }
            }
        });
    }
}
```
단순히 스트림 API 가 추가되어 메서드 참조와 람다가 생겼다고 생각하면   
메인메서드는 아래에서 생각이 그칠수있다.

```java
print(val, bar -> {
    switch (bar) {
        case 0:
            return "hello";
        case 1:
            return "world";
        default:
            return "error";
    }
});

```

하지만 그보다 더 넓은 활용성이 있다는 것을 기억해야한다.

아래처럼 활용하면 어떨까?

```java
public class Main {

    public static void print(int val, Function<Integer, String> wordPicker) {
        String result = wordPicker.apply(val);
        System.out.println(result);
    }

    public static void main(String[] args) {
        int val = Integer.parseInt(args[0]);

        Function<Integer, String> wordPicker = (v) -> {
            switch (v) {
                case 0:
                    return "hello";
                case 1:
                    return "world";
                default:
                    return "error";
            }
        };

        print(val, wordPicker);
    }
}
```

스트림API 에서 사용하는 것처럼 단순히 메소드에 파라미터를 넣는 것이아니라  
메서드에 코드를 전달할 뿐만 아니라 결과를 반환하고 다른 자료구조로 전달할 수도 있는 것이라는  
더 넓은 활용성이 있다는 것을 기억해야한다



