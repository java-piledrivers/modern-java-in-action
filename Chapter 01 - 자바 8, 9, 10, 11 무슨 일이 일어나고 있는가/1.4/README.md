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
