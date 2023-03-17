# 1.1 역사의 흐름은 무엇인가?

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


# 1.2 왜 아직도 자바는 변화하는가?

언어는 마치 생태계와 닮았다. 진화하는 언어는 생존하고 그렇지 않은 언어는 도태되어 사장된다.   
자바는 어떻게 성공을 거두었나?

### 프로그래밍 언어 생태계에서 자바의 위치

많은 유용한 라이브러리를 포함하여 잘 설계된 객체지향 언어로 시작하였고,   
스레드와 락을 이용한 소소한 동시성도 지원함.

코드를 JVM 바이트 코드로 컴파일하는 특징때문에 처음에 인터넷 애플릿 프로그램의 주요 언어가 됨

> 애플릿 프로그램이란?
> 인터넷 애플릿은 웹 브라우저에서 실행되는 Java 가상 머신(JVM)을 사용하여 작동.   
> 이를 통해 사용자는 브라우저에서 자바 프로그램을 실행할 수 있으며,   
> 인터넷 애플릿을 사용하여 그래픽, 애니메이션, 데이터베이스 연결 등을 포함한 다양한 작업을 수행 가능.
> 최근에는 보안이슈와 웹 표준 발전으로 사용하지 않음

그리고 자바가 처음에 각광받은 것은 캡슐화와 객체지향 덕분.   
처음에는 C/C++ 비해서 앱을 실행하는데 추가적으로 드는 비용떄문에 자바에 대한 반감이 있었지만,   
점차 하드웨어 발전으로 프로그래머의 시간이 더욱 중요한 요소로 부각 되어짐.

자바 8 은 현재 언어 생태계에서 요구하는 기능을 효과적으로 제공한다.

### 스트림 처리

스트림이란 일련의 연속적으로 구성된 데이터 묶음이라고 보면 된다.   
따라서 스트림 처리란 이런 데이터 흐름들의 처리를 말하는 것이다.   
유닉스 명령어를 `|` 파이프를 이용해 연결하는 것도 스트림 처리이다.

리눅스 명령어 예시: `cat file1 file2 | tr "[A-Z]" "[a-z]" | sort | tail -3`

자바8에서 제공하는 스트림 API 가 이런 파이프라인을 만드는데 필요한 다양한 메소드를 제공한다고 생각하면 된다.   
스트림 API의 핵심은 기존엔 한번에 한항목을 처리하던 것을 고수준으로 추상화하여 일련의 스트림으로 만들어 처리가 가능하다는 것.   
또한 여러 CPU 코어를 쉽게 할당할 수 있고, 병렬성을 얻을 수 있다.    

### 동작 파라미터화로 메서드에 코드 전달

위 리눅스 명렁어 예시에서 예를 들어 sort 에 어떤 기준을 넣어 sorting 하고 싶을 수도 있다.   
자바8 이전에는 이를 수행하려면 1.1에서 본것처럼 Comparator 객체를 만들어 sort 에 넘겨주어야하지만 이는 너무 복잡하다
자바8 에서는 이런 기능을 제공해준다.   

이런 기능을 동작. 파라미터화(beavior parameterization)이라고 하는데,   
스트림 API 에서 연산의 동작을 파라미터화할 수 있는 코드를 전달한다는 사상에 기ㅊ초하기 때문에 중요하다

### 병렬성과 공유 가변 데이터

스트림 API 를 통해서 병렬성을 쉽게 얻을 수 있다.   
다른 코드와 동시에 실행하더라도 안전하게 실행할 수 있는 코드를 만드려면 공유된 가변 데이터에 접근하지 않아야 한다.

지금까지는(자바 8이전까지는) 독립적으로 실행 될 수 있는 다중 코드 사본과 관련된 병렬성을 고려했다.

> 다중 코드 사본 (multiple code copies) 를 이용한 sum 예제

```java
public class ParallelArraySum {
    private int[] array;
    private int numOfThreads;

    public ParallelArraySum(int[] array, int numOfThreads) {
        this.array = array;
        this.numOfThreads = numOfThreads;
    }

    public int sum() throws InterruptedException {
        int size = (int) Math.ceil(array.length * 1.0 / numOfThreads);
        int[] sums = new int[numOfThreads];
        Thread[] threads = new Thread[numOfThreads];

        for (int i = 0; i < numOfThreads; i++) {
            final int start = i * size;
            final int end = (i + 1) * size;
            threads[i] = new Thread(() -> {
                int sum = 0;
                for (int j = start; j < end && j < array.length; j++) {
                    sum += array[j];
                }
                sums[i] = sum;
            });
            threads[i].start();
        }

        for (int i = 0; i < numOfThreads; i++) {
            threads[i].join();
        }

        int sum = 0;
        for (int i = 0; i < numOfThreads; i++) {
            sum += sums[i];
        }

        return sum;
    }

    public static void main(String[] args) throws InterruptedException {
        int[] array = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
        ParallelArraySum parallelArraySum = new ParallelArraySum(array, 4);
        int sum = parallelArraySum.sum();
        System.out.println("Sum: " + sum);
    }
}
```
다중 코드 사본이란 하나의 프로그램에서 여러 개의 복사본을 만들어 각각의 복사본이 데이터를 처리하도록 하는 것이다.   
각각의 threads[i] 들은 똑같은 함수를 구현한 Thread 를 받고 start 하게 되는데, 이를 다중 코드 사본이라고 하는 것이다.

하지만 이런 코드는 공유된 변수나 객체가 있으면 병렬성에 문제가 발생한다

기존처럼 synchronized 를 이용해 공유 가변 데이터를 보호하는 규칙을 만들수는 있을것이지만, (synchronized 는 일반적으로 시스템 성능에 악영향을 미친다고 함).  
스트림 API 를 이용하면 쉽게 병렬성을 활용할 수 있다.


# 1.3 자바 함수  
프로그래밍 언어애서 함수라는 용어는 '메서드', 특히 정적 메서드와 같은 의미로 사용되지만 자바의 함수는 이에 더해 수학적인 함수처럼 사용되며 부작용을 일으키지 않는 함수를 의미한다.  
자바 8에서는 함수를 새로운 값의 형식으로 추가했는데, 이는 멀티코어에서 병렬 프로그래밍을 활용할 수 있는 스트림과 연계될 수 있기 위함이다.  
  
함수가 필요한 이유를 알아보기 전에 일급 값 또는 일급 시민이라고 불리는 용어의 의미를 알아보자.  
전통적으로 미국 시민 권리에서 유래한 해당 용어는 바꿀 수 있는 값을 일급 시민이라고 불러왔다. 예를 들어 int나 double 형식의 기본값 및 String 형식의 객체가 일급 시민에 해당한다. 하지만 클래스나 메서드와 같은 경우 값의 구조를 표현하는 데 도움이 될 수는 있지만, 그 자체로 값이 될 수는 없기 때문에 이급 시민에 해당한다.

이렇게 이급 시민인 메서드를 런타임에 전달할 수 있는 일급 시민으로 만든다면 프로그래밍에 유용하게 활용될 수 있다.
  
## 메서드와 람다를 일급 시민으로  
### 메서드 참조
자바 8 이전에는 메서드가 값이 될 수 없었기 때문에 메서드를 객체로 인스턴스화하여 전달해야 했다. 하지만 자바 8의 메서드 참조를 이용하면 메서드 자체가 함수의 파라미터로 전달될 수 있다.

해당 예시는 디렉터리의 숨겨진 파일을 찾는 코드이다.
```java
//자바 8 이전
File[] hiddenFiles = new File(".").listFiles(new FileFilter() {
    public boolean accept(File file) {
        return file.isHidden();
    }
})
```  
```java
//메서드 참조 적용
File[] hiddenFiles = new File(".").listFiles(File::isHidden);
```
이미 `isHidden()`이라는 메서드는 준비되어 있으므로 메서드 참조(::)를 이용해 값을 직접 전달할 수 있게 되었다.

### 람다(익명함수)
자바 8에서는 메서드를 일급 값으로 취급할 뿐 아니라 람다를 포함해서도 값으로 표현할 수 있는데, 직접 메서드를 정의하는 상황이 아니면서 이용할 수 있는 편리한 클래스나 메서드가 없을 경우 람다 문법을 이용하면 간결하게 코드를 작성할 수 있다.
람다 문법 형식으로 구현된 프로그램을 '함수로 일급 값으로 넘겨주는 프로그램을 구현한다'라고 한다. 


## 코드 넘겨주기 - 예제  
Apple 클래스와 getColor 메서드가 있고, Apples 리스트를 포함하는 변수 inventory가 있다고 가정할 때, 원하는 조건에 맞게 리스트를 반환하는 프로그램을 구현해 보자.
이때 누군가 150그램 이상인 사과만 필터링하고 싶다면 다음과 같이 코드를 작성할 수 있을 것이다.
```java
public static List<Apple> filterHeavyApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (apple.getWeight() > 150) {
            result.add(apple);
        }
    }
    return result;
}
```
만약 무게를 기준으로 하는 것이 아닌 색깔을 기준으로 한다면, if문 내부의 코드만 달라지고 완전히 똑같은 구조의 코드가 된다.
이는 복사&붙여넣기 식의 코드 작성이며 버그가 발생할 경우 모든 코드를 고쳐야 하는 문제점이 발생한다.

하지만 자바 8에서는 코드를 인수로 넘겨줄 수 있으므로 필터 메서드를 중복으로 구현할 필요가 없어 아래와 같이 구현할 수가 있다.
```java
public static boolean isHeavyApple(Apple apple) {
    return apple.getWeight() > 150;
}

static List<Apple> filterApples(List<Apple> inventory, Predicate<Apple> p) { //조건 판별 함수가 Predicate 파라미터로 전달
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (p.test(apple)) { //조건 부합 여부 확인
            result.add(apple);
        }
    }
    return result;
}
```
참고로 위에서 쓰인 Predicate는 인수로 값을 받아 참 또는 거짓을 반환하는 함수를 의미하는 용어이다.
이제 여러 가지 조건을 검증하기 위해서 똑같은 코드를 복사&붙여넣기 하는 식으로 작성할 필요 없이 검증 메서드만 추가적으로 작성하면 된다. `filterApples()`메서드는 다음과 같이 호출하여 조건을 판별할 수 있다.
```filterApples(inventory, Apple::isGreenApple)```


## 메서드 전달에서 람다로
한 두 번 사용할 메서드를 매번 정의하고 값으로 전달하는 것은 다소 번거로운 일이다. 자바 8에서는 이러한 문제를 해결하기 위해 다음과 같이 익명함수(람다)를 이용할 수 있다.
```java
filterApples(inventory, (Apple a) -> a.getWeight() > 150);
```
하지만 람다가 복잡한 동작을 수행하면서 몇 줄 이상으로 길어진다면, 람다보다는 코드가 수행하는 일을 잘 설명하는 이름을 가진 메서드를 정의하고 메서드 참조를 활용하는 것이 바람직하다. 코드의 명확성이 우선시되어야 하기 때문이다.

# 1.4 스트림

거의 모든 자바 애플리케이션은 컬렉션을 **만들고 활용**한다.   
하지만 리스트에서 고가의 트랜잭션만 필터링할 경우 다음 코드처럼 많은 기본코드를 구현해야한다.

```java
class LegacyTest {
  private static void groupTransaction() {
    // 그룹화된 트랜잭션을 더할 Map 생성
    Map<Currency, List<Transaction>> transactionsByCurrencies = new HashMap<>();
    // 트랜잭션의 리스트를 반복
    for (Transaction transcation : transactions) {
      // 고가의 트랜잭션을 필터링
      if (transaction.getPrice() > 1000) {
        // 트랜잭션의 통화 추출
        Currency currency = transaction.getCurrency();
        List<Transaction> transactionsForCurrency = transactionsByCurrencies.get(currency);
        // 현재 통화의 그룹화된 맵에 항목이 없으면 새로 만든다.
        if (transactionsForCurrency == null) {
          transactionsForCurrency = new ArrayList<>();
          transactionsByCurrencies.put(currency, transactionsForCurrency);
        }
        // 현재 탐색된 트랜잭션을 같은 통화의 트랜잭션 리스트에 추가한다.
        transactionsForCurrency.add(transaction);
      }
    }
  }
}
```

게다가 위 예제 코드에는 중첩된 제어 흐름 문장이 많아서 코드를 한번에 이해하기도 어렵다.   
스트림 API를 이용하면 다음처럼 문제를 해결할 수 있다.

```java

import static java.util.stream.Collectors.toList; 
class LegacyTest {
  private static void groupTransaction() {
    Map<Currency, List<Transaction>> transactionsByCurrencies = 
        transactions.stream()
            .filter((Transaction t) -> t.getPrice() > 1000) // 고가의 트랜잭션 필터링
            .collection(groupingBy(Transaction::getCurrency)); //통화로 그룹화함
  }
}
```

## 컬렉션
- for-each루프를 이용해 반복 과정을 직접 처리함. -> **외부 반복**
- 기본적으로 단일 CPU만 이용하여 순차적으로 처리한다.

## 스트림
- 라이브러리 내부에서 수행함. -> **내부 반복**
- 기본적으로 멀티 CPU를 이용하여 병렬로 처리한다.

# 멀티 스레드
멀티 스레딩 모델은 순차적인 모델보다 다루기 어렵다.   
두 스레드가 적절하게 제어되지 않은 상황에서 공유된 sum 변수에 숫자를 더하면 다음과 같은 문제가 있다.   
(책의 그림을 보는 것을 추천)

### 구성 환경
스레드 1, 스레드 2, 변수 sum (값 100)
1. 스레드 1이 sum 변수 값을 읽는다.
2. 스레드 2가 sum 변수 값을 읽는다.
3. 스레드 1이 sum 변수에 3을 더한다.
4. 스레드 2가 sum 변수에 5를 더한다.
5. 스레드 1이 sum 변수에 103을 저장한다.
6. 스레드 2가 sum 변수에 105를 저장한다.
결과 : 108이 아닌 105가 저장된다.

자바 8은 `컬렉션을 처리하면서 발생하는 모호함과 반복적인 코드 문제` 그리고 `멀티코어 활용 어려움`을 해결하기 위해 스트림 API를 도입했다.

## 컬렉션을 처리하면서 발생하는 모호함과 반복적인 코드 문제
기존의 컬렉션에서는 데이터를 처리할 때 반복적인 패턴이 너무 많았다.    
스트림 API는 라이브러리에서 이러한 반복되는 패턴을 처리하고, 개발자는 데이터를 처리하는 방법에 집중할 수 있도록 도와주게 되었다.   
데이터를 처리하는 방법은 다음과 같다.

```java
class Person {

  private String name;
  private int age;
}
```

- 필터링 (나이가 30살 이상인 사람만 필터링)
- 추출 (Person 객체에서 name만 추출)
- 그룹화 (나이가 같은 사람들을 그룹화)

## 멀티코어 활용의 어려움
또한 이러한 동작들을 쉽게 병렬화할 수 있다.   
[그림 1-6]에서 보여주는 것처럼 두 CPU를 가진 환경에서 리스트를 필터링할 때 한 CPU는 리스트의 앞부분, 다른 CPU는 리스트의 뒷 부분을 처리하게 할 수 있다.   
이 과정을 **포킹 단계**라고 한다.

컬렉션을 필터링할 수 있는 가장 빠른 방법은 
1. 컬렉션을 스트림으로 바꾸고
2. 병렬로 처리한 다음에
3. 리스트로 다시 복원
하는 것이다.

### 순차 처리 방식의 코드
```java
import static java.util.stream.Collectors.toList;
List<Apple> heavyApples =
    inventory.stream().filter((Apple a) -> a.getWeight() > 150)
                      .collect(toList());
```

### 병렬 처리 방식의 코드
```java
import static java.util.stream.Collectors.toList;
List<Apple> heavyApples =
    inventory.parallelStream().filter((Apple a) -> a.getWeight() > 150)
                              .collect(toList());
```

## 자바의 병렬성과 공유되지 않은 가변 상태
흔히 자바의 병렬성은 어렵고, synchronized는 쉽게 에러를 일으킨다고 생각한다.   
자바 8은 이를 해결하기 위해 두가지 방식을 사용한다.
1. 라이브러리에서 분할 처리를 한다. 큰 스트림을 병렬 처리할 수 있도록 작은 스트림으로 분할한다.
2. filter 같은 라이브러리 메서드로 전달된 메서드가 상호작용하지 않는다면 가변 공유 객체를 통해 병렬성을 누릴 수 있다.   
-> 여러 스레드가 동시에 가변 객체에 접근하더라도, 메서드가 상호작용하지 않기 때문에 안전하지만, 만약 전달되는 메서드가 공유되는 가변 객체를 변경한다면 문제가 발생한다. 이런 경우에는 스레드 동기화 기법을 사용하여 해결해야 한다.

함수형 프로그래밍에서 함수형이란 `함수를 일급 값으로 사용한다.`는 의미를 가지고 있지만 부가적으로 `프로그램이 실행되는 동안 컴포넌트 간에 상호작용이 일어나지 않는다` 라는 의미도 포함한다. 

---

### 참고
ArrayList와 같은 순차적인 자료구조는 내부적으로 인덱스를 사용하여 요소들에 접근하기 때문에 병렬 처리에 적합하지 않는다.    
따라서, 병렬 처리를 지원하는 자료구조를 사용해야 한다.   
병렬 처리가 항상 순차 처리보다 빠른 것은 아니므로, 성능 최적화를 위해서는 실제 데이터에 대한 측정과 분석이 필요하다.

# 1.5 인터페이스

자바에서 인터페이스는 클래스들이 필수로 구현해야 하는 추상 자료형입니다.

=> 하나의 설계도면
=> ex) 동물 있다면 동물에 대한 설계도(움직이기, 먹기, 소리내기)


1.
```java
public interface Animal {
    public static final String name = "동물";
    
    public abstract void move();
    public abstract void eat();
    public abstract void bark();
}
```


2.
```java
public class Dog implements Animal{
    
    @Override
    public void move() {
        System.out.println("슥슥~~");
    }
    
    @Override
    public void bark() {
        System.out.println("멍멍!");
    }
}
```


하지만 sleep() 이라는 기능을 추가싶다면 


1.
```java
public interface Animal {
    public static final String name = "동물";
    
    public abstract void move();
    public abstract void eat();
    public abstract void bark();
    public abstract  void sleep();
}
```

2.
```java
public class Dog implements Animal{
    
    @Override
    public void move() {
        System.out.println("슥슥~~");
    }
    
    @Override
    public void bark() {
        System.out.println("멍멍!");
    }

    @Override
    public void sleep() {
        System.out.println("쿨쿨zzzz");
    }
}
```

해당 sleep 이라는 추상 메서드를 구현해주어야한다.


지금은 Animal 인터페이스를 구현한 Dog 클래스가 하나라 그렇지만 10~ 100개라면 해당 추상메서드를 전부다 구현해야 된다


이때를 위해 나온 메서드가 default 메서드이다.



1.
```java
public interface Animal {
    public static final String name = "동물";
    
    public abstract void move();
    public abstract void eat();
    public abstract void bark();
    
    default void sleep(){
    System.out.println("쿨쿨zzzz");
    }
}
```

2.
```java
public class Dog implements Animal{
    
    @Override
    public void move() {
        System.out.println("슥슥~~");
    }
    
    @Override
    public void bark() {
        System.out.println("멍멍!");
    }
}
```

3. 
```java
public class Main {
    public static void main(String args[]){

    Dog dogTest = new Dog();
    dogTest.sleep();
    }
}
```


결과 : 쿨쿨zzzz



또한 인터페이스에서 구현된 default Method 는 구현클래스에서 재정의도 가능합니다.




=> 기존에 선언되고 구현된 메서드에 대하여 영향을 미치지 않기위해  => 하위호환성을 위해서 default Method 사용




그렇다면 추상클래스와 다를게 없지 않나요?

=>  클래스는 하나의 추상클래스를 상속 받지만 인터페이스는 여러개를 구현할수 있는 차이가 있습니다.


# 1.6 함수형 프로그래밍에서 가져온 다른 유용한 아이디어 

1.1~1.5 장을 통해 함수형 프로그램의 두 아이디어를 살펴보았다.

- 메서드와 람다를 일급값으로 사용하는 것
- 가변 공유 상태가 없는 병렬 실행을 이용해서 효율적이고 안전하게 함수나 메서드를 호출할 수 있다는 것

Stream API는 이 두 가지 아이디어를 모두 사용한다.



일반적인 함수형 언어도 프로그램을 돕는 여러 장치를 제공한다 -- 일례로 명시적으로 서술형의 데이터 형식을 이용해 null을 회피하는 기법이 있다.



자바8에서는 NullPointer 예외를 피할 수 있도록 도와주는 `Optional<T>` 클래스를 제공한다.

`Optional<T>`는 값을 갖거나 갖지 않을 수 있는 컨테이너 객체이며, 값이 없는 상황을 어떻게 처리할지 명시적으로 구현하는 메서드를 포함하고 있어 NullPointer 예외를 피할 수 있다.



`구조적 패턴 매칭 기법`도 있다. 패턴 매칭은 수학에서 다음 예제 처럼 사용한다.

````python
f(0) = 1
f(n) = n*f(n-1)
````

자바에서는 `if-then-else`    를 사용했을 것이다. 또한 다형성, 메서드 오버라이딩을 통해 비교문을 만들 수 있다.

하지만 다른 언어에서 패턴 매칭으로 더 정확한 비교를 할 수 있다는 것을 증명했으며, 우리는 기능 구현보다는 언어 설계를 논하고 있음을 기억하자.

아쉽게도 자바8은 아직 완벽히 패턴 매칭을 지원하고 있지 않아, 스칼라 프로그래밍 언어로 패턴 매칭 하는 방법을 살펴본다.

아래는 트리로 구성된 수식을 단순화 하는 프로그램이다.

````scala
def simplifyExpression(expr: Expr): Expr = expr match {
  case BinOp("+", e, Number(0)) => e // 0 추가
  case BinOp("-", e, Number(0)) => e // 0 빼기
  case BinOp("*", e, Number(1)) => e // 1 곱하기
  case BinOp("/", e, Number(1)) => e // 1 나누기
  case _ => expr // 표현식을 단순화 할 수 없음
}
````

스칼라의 expr match는 자바의 switch(expr)과 같은 기능을 수행한다.

패턴 매칭이 switch를 확장한 것으로 데이터 형식 분류와 분석을 한 번에 수행할 수 있다는 정도로만 생각하자.

왜 자바의 switch문에는 문자열과 기본값만 이용할 수 있는 걸까?

함수형 언어는 보통 패턴 매칭을 포함한 다양한 데이터 형식을 switch에 사용할 수 있다(스칼라에서는 match를 활용).

일반적으로 객체지향 설계에서 클래스 패밀리를 방문할 때 `방문자 패턴(visitor pattern)`을 이용해서 각 객체를 방문한 다음에 원하는 작업을 수행한다. 

> <u>**Visitor pattern, 방문자 패턴**</u>
>
> 실제 로직을 가지고 있는 객체(Visitor)가 로직을 적용할 객체(Element)를 방문하면서 실행하는 패턴이다.</br>
> 즉, 로직과 구조를 분리하는 패턴이라고 볼 수 있다.</br> 
> 로직과 구조가 분리되면 구조를 수정하지 않고도 새로운 동작을 기존 객체 구조에 추가할 수 있다.
>
> https://thecodinglog.github.io/design/2019/10/29/visitor-pattern.html 참고



패턴 매칭을 이용하면 "Brakes 클래스는 Car클래스를 구성하는 클래스 중 하나입니다. Brakes를 어떻게 처리해야 할지 설정하지 않았습니다" 와 같은 에러를 검출할 수 있다.


