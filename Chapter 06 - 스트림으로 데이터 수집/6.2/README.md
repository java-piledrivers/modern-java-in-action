## 6.2 리듀싱과 요약

- Stream.collect() 로 스트림의 항목을 컬렉션으로 재구성 할(하나로 합칠) 수 있다.

## 6.2.1 스트림값에서 최댓값과 최솟값 검색

`maxBy` , `minBy` : Stream의 요소를 비교하는 데 사용할 Comparator를 인수로 받음.

```java
Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalrories);

Optional<Dish> mostCalorieDish = menu.stream().collect(maxBy(dishCaloriesComparator));
```

## 6.2.2 요약 연산

Collectors 클래스는 여러 특별한 요약 팩토리 메소드를 제공한다.

- 단순 합계
    - `summingInt` : 객체를 int로 매핑하는 함수를 인수로 받아 sum 작업을 수행한다.
    - 기타 : summingLong, summingDouble..
- 평균
    - averagingInt, averagingLong, averagingDouble..
    
    ```java
    double avgCalrories = menu.stream().collect(averagingInt(Dish::getCalrories))
    ```
    
- 여러 정보를 구해야 할 때
    - `IntSummaryStatistics` : 이 클래스 안에 count, sum, min 등 정보를 수집시킬 수 있다.
    
    ```java
    IntSummaryStatistics menuStatistics = menu.stream().collect(summarizingInt(Dish::getCalrories));
    // menuStatistics : IntSummaryStatistics{count=9, sum=4300, min=120, average=477.778, max=800}
    ```
    
    - LongSummaryStatistics, DoubleSummaryStatistics …

## 6.2.3 문자열 연결

`joining` : 스트림의 각 객체에 toString 메서드를 호출해서 추출한 모든 문자열을 하나의 문자열로 연결해서 반환한다.

```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining(", "))
```

## 6.2.4 범용 리듀싱 요약 연산

앞에서 살펴본 모든 컬렉터를 `Collectors.reducing` 로 정의할 수 있다.

```java
double avgCalrories = menu.stream().collect(averagingInt(Dish::getCalrories))

// reducing
// public static <T, U> Collector<T, ?, U> reducing(
//																U identity,
//                                Function<? super T, ? extends U> mapper,
//                                BinaryOperator<U> op)
double avgCalrories = menu.stream().collect(reducing(0, Dish::getCalrories, (i, j) -> i+j));
```

******세 개의 인수를 갖는 reducing******

- 첫 번째 인수 : 리듀싱 연산의 시작값이거나, 스트림에 인수가 없을 때 반환값
- 두 번째 인수 : 요리를 칼로리 정수로 변환 시 사용하는 변환함수
- 세 번째 인수 : BinaryOperator

**********한 개의 인수를 갖는 reducing**********

```java
// public static <T> Collector<T, ?, Optional<T>> reducing(BinaryOperator<T> op)

Optional<Dish> mostCalrorieDish = menu.stream().collect(reducing(d1, d2) -> d1.getCalrories() > d2.getCalrories() ? d1 : d2));
```

- 초기 값 : 첫 번째 stream 의 인수
- 인수는 자신을 그대로 반환하는 항등 함수
- 따라서 스트림이 비어있을 경우를 대비하여 Optional로

### collect 와 reduce

- 이 두가지 메서드로 같은 기능을 구현할 수 있지만 의미론적인 문제와 실용성 문제 등 몇가지 문제가 있다.

```java
Stream<Integer> stream = Arrays.asList(1, 2, 3, 4, 5, 6).stream();

// collect로 만든 리스트
List<Integer> collectedList = stream.collect(toList());

// reduce로 만든 리스트
List<Integer> reducedList = stream.reduce(
                new ArrayList<>(), 
								(List<Integer> l, Integer e) -> { // 누적자
                    l.add(e);
                    return l;},
								(List<Integer> l1, List<Integer> l2) -> { // 결합자
                    l1.addAll(l2);
                    return l1;});
```

- 의미론적인 문제
    - **collect 메서드는 도출하려는 결과를 누적하는 컨테이너를 바꾸도록 설계된 메서드**
        - 누적자로 사용된 리스트를 변환시키므로 reduce를 잘못 활용한 예에 해당
    - reduce 메서드는 두 값을 하나로 도출하는 불변형 연산
- 실용성 문제
    - 여러 스레드가 동시에 같은 데이터 구조체를 고치면 리스트 자체가 망가져버리므로 리듀싱 연산을 병렬로 수행할 수 없다.
    - 이를 피하기 위해 매번 리스트를 새로 할당하고 객체를 할당하느라 성능이 낮을 것 이다.
    

### 컬렉션 프레임워크 유연성 : 같은 연산도 다양한 방식으로 수행할 수 있다.

```java
// 1. reduce 메소드만 사용
int totalCalories = menu.stream().collect(reducing(0, Dish::getCalories, Integer::sum));

// 2. Optional<int> 사용
int totalCalories = menu.stream().map(Dish::getCalories).reduce(Integer::sum).get();

// 3. intStream 사용
int totalCalories = menu.stream().mapToInt(Dish::getCalories).sum();
```

1. reduce만 사용해서 값을 구할 수 있다.
2. map을 사용하여 Optional을 사용해서 예외 처리를 할 수 있다. orElse, orElseGet 등을 사용할 수 있다.
3. intStream으로 합계를 구할 수 있다.

### 자신의 상황에 맞는 최적의 해법 선택

함수형 프로그래밍(특히 자바8의 컬렉션 프레임워크에 추가된 함수형 원칙에 기반한 새로운 API)에서는 하나의 연산을 다양한 방법으로 해결할 수 있다.

또한 스트림 인터페이스에서 직접 제공하는 메서드를 이용하는 것에 비해 컬렉터를 이용하는 코드가 더 복잡하다.

코드가 복잡한 대신 재사용성과 커스터마이즈 가능성을 제공하는 높은 수준의 추상화와 일반화를 얻을 수 있다.

→ 문제를 해결할 수 있는 다양한 해결 방법을 확인한 다음에 가장 일반적으로 문제에 특화된 해결책을 고르는 것이 바람직하다. 이렇게 해서 가독성과 성능이라는 두 마리 토끼를 잡을 수 있다.
