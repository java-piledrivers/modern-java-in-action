## 4.4 스트림 연산

스트림 인터페이스의 연산을 크게 두 가지로 나눌 수 있다.

![image](https://user-images.githubusercontent.com/43192041/229140607-c60ae37f-db33-4294-bbf1-8b3fd42b0f49.png)
### 4.4.1 중간 연산

- **다른 스트림을 반환한다.**

- 연산 결과를 스트림으로 반환하기 때문에, 연속적으로 여러 번 수행할 수 있다.
- 단말 연산을 스트림 파이프라인에 실행하기 전까지는 아무 연산도 수행하지 않는다 - 게으르다.
- 중간 연산을 합친 다음에 합쳐진 중간 연산을 최종 연산으로 **한 번에 처리한다.**

#### 중간 연산의 게으른 특성으로 얻은 최적화 효과
```java
List<Stirng> names = menu.stream() // 스트림 open
		.filter(dish -> dish.getCalories > 300) // 중간 연산 시작
		.map(Dish::getname)
		.limit(3) // 중간 연산 끝, short-circuit
		.collect(toList()); // 최종 연산
```
- 쇼트 서킷
  - 모든 연산을 다 해보기 전에 조건을 만족하면 추가적인 불필요한 연산은 하지 않는다.
  - 위의 예시에서는 limit 연산이 쇼트 서킷 연산에 해당된다.
  - 3개의 결과를 얻은 후 앞선 filter와 map연산은 더 이상 수행할 필요가 없어 빠르게 최종 연산을 수행한다.

- 루프 퓨전
  - 위의 예시 코드에서 filter와 map 연산에 값을 print 하는 과정을 추가한다면(책 p.151 참고) filter와 map이 다른 연산이지만 한 과정으로 병합되어 처리됨을 확인할 수 있다.
  - 루프 퓨전은 이렇게 둘 이상의 연산이 합쳐 하나의 연산으로 처리됨을 말한다.


### 4.4.2 최종 연산

- 스트림 파이프라인에서 결과를 도출한다.
- 한 번만 연산 가능하다.
- 보통 최종 연산에 의해 List, Integer, void 등 스트림 이외의 결과가 반환된다.


### 4.4.3 스트림 이용하기
**스트림 이용 과정**
- 질의를 수행할 (컬렉션 같은) 데이터 소스
- 스트림 파이프라인을 구성할 중간 연산 연결
- 스트림 파이프라인을 실행하고 결과를 만들 최종 연산
* * *
### 참고
### 중간 연산 종류 - 가공
  * 스트림 필터링 : filter(), distinct()
  * 스트림 변환 : map(), flatMap()
  * 스트림 제한 : limit(), skip()
  * 스트림 정렬 : sorted()
  * 스트림 연산 결과 확인 : peek()
 
#### 스트림 필터링 - filter, distinct

```java
IntStream stream1 = IntStream.of(1, 2, 3, 4, 5, 5, 6, 6, 1, 3, 4);
IntStream stream2 = IntStream.of(1, 2, 3, 4, 5, 5, 6, 6, 1, 3, 4);

// 홀수만을 골라내기
stream1.filter(n -> n % 2 != 0).forEach(System.out::print); // 135513

// 중복된 요소 제거
stream2.distinct().forEach(System.out::print); // 123456
```

* filter() : 해당 스트림에서 주어진 조건(predicate)에 맞는 요소만으로 구성된 새로운 스트림을 반환한다.
* distinct() : 내부적으로 Object 클래스의 `equals()` 메서드를 사용하여 요소의 중복을 비교한다.

<br>

#### 스트림 변환 - map, flatMap

> filter가 조건을 충족시키는 아이템들만으로 새로운 스트림을 생성한다면,
>
>  `Map`은 각각의 item을 변경하여 새로운 컨텐츠를 생성하는 기능이다.
>
>  filter와 마찬가지로 `map(함수)`으로 어떻게 아이템을 변경시킬 지 함수로 정의한다.

```java
Stream<String> stream = Stream.of("JAVA", "C", "C++", "PYTHON");
stream.map(s -> s.length()).forEach(System.out::println); // 4 1 3 6
// stream.map(s -> s.toLowerCase()).forEach(System.out::println); // 소문자로 전환
```

* map() : 해당 스트림의 요소들을 주어진 함수에 인수로 전달하여, 그 반환값들로 이루어진 새로운 스트림을 반환한다.
  * 즉, 스트림 내 요소들을 하나씩 특정 값으로 변환해준다. 값을 변환하기 위한 람다를 인자로 받는다.

> FlatMap은 여러개의 스트림을 한개의 스트림으로 합쳐준다.
>
> 복잡한 스트림을 간단한 스트림으로 변경되는데 사용할 수 있다.

```java
Stream<String> stream = Stream.of("I study hard", "You study JAVA", "I am hungry");
stream.flatMap(s -> Stream.of(s.split(" "))).forEach(System.out::println);

// result
I
study
hard
You
study
JAVA
I
am
hungry

```

<br>

#### 스트림 제한 - limit, skip

```java
IntStream stream1 = IntStream.range(0, 10);
IntStream stream2 = IntStream.range(0, 10);

stream1.skip(4).forEach(n -> System.out.print(n + " ")); // 4 5 6 7 8 9
stream2.limit(5).forEach(n -> System.out.print(n + " ")); // 0 1 2 3 4
```

* limit() : 해당 스트림의 첫 번째 요소부터 전달된 개수만큼의 요소만으로 이루어진 새로운 스트림을 반환한다.
* skip() : 해당 스트림의 첫 번째 요소부터 전달된 개수만큼의 요소를 제외한 나머지 요소만으로 이루어진 새로운 스트림을 반환한다.

<br>

#### 스트림 정렬 - sorted

```java
Stream<String> stream = Stream.of("JAVA", "C", "C++", "PYTHON");

stream.sorted().forEach(s -> System.out.print(s + " ")); // C C++ JAVA PYTHON
```

* sorted() : 해당 스트림을 주어진 비교자(`comparator`)를 이용하여 정렬한다.
* 비교자를 전달하지 않으면 기본적으로 사전 편찬 순으로 정렬하게 된다.

<br>



### 최종 연산 종류 - 결과
  * 요소의 출력 : forEach()
  * 요소의 소모 : reduce()
  * 요소의 검색 : findFirst(), findAny()
  * 요소의 검사 : anyMatch(), allMatch(), noneMatch()
  * 요소의 통계 : count(), min(), max()
  * 요소의 연산 : sum(), average()
  * 요소의 수집 : collect()


#### 요소의 출력 - forEach

```java
IntStream stream = IntStream.of(1, 2, 3, 4);
stream.forEach(System.out::println);
```

* forEach() : 스트림의 각 요소를 소모하여 명시된 동작을 수행한다. (for문과 같다.)
  * 반환 타입이 void이므로 보통 스트림의 모든 요소를 출력하는 용도로 많이 사용된다.

<br>

#### 요소의 소모 - reduce

```java
Stream<Integer> numbers = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
Optional<Integer> sum = numbers.reduce((x, y) -> x + y);
sum.ifPresent(s -> System.out.println("sum: " + s));
```

* reduce() : 첫 번째와 두번째 요소를 가지고 연산을 수행한 뒤, 그 결과와 세 번째 요소를 가지고 또 다시 연산을 수행한다. 이런식으로 해당 스트림의 모든 요소를 소모하여 연산을 수행하고, 그 결과를 반환하게 된다.

<br>

#### 요소의 검색 - findFirst, findAny

```java
IntStream stream1 = IntStream.of(4, 2, 7, 3, 5, 1, 6);
IntStream stream2 = IntStream.of(4, 2, 7, 3, 5, 1, 6);

OptionalInt result1 = stream1.sorted().findAny();
System.out.println(result1.getAsInt()); // 1

OptionalInt result2 = stream2.sorted().findFirst();
System.out.println(result1.getAsInt()); // 1
```

* findFirst(), findAny() : 두 메서드 모두 스트림에서 첫 번째 요소를 참조하는 `Optional` 객체를 반환한다.
* 차이점
  * 병렬 스트림인 경우에는 findAny() 메서드를 사용해야만 정확한 연산 결과를 반환할 수 있다.

<br>

#### 요소의 검사 - anyMatch, allMatch, noneMatch

```java
IntStream stream1 = IntStream.of(4, 2, 7, 3, 5, 1, 6);
IntStream stream2 = IntStream.of(4, 2, 7, 3, 5, 1, 6);

System.out.println(stream1.anyMatch(n -> n > 5)); // true
System.out.println(stream2.allMatch(n -> n > 5)); // false
```

* anyMatch() : 해당 스트림의 **일부 요소**가 특정 조건을 **만족할 경우**에 true를 반환함
* allMatch() : 해당 스트림의 **모든 요소**가 특정 조건을 **만족할 경우**에 true를 반환함.
* noneMatch() : 해당 스트림의 **모든 요소**가 특정 조건을 **만족하지 않을 경우**에 true를 반환함.

* 공통점
  * 세 메서드 모두 인수로 Predicate 객체를 전달받으며, 요소의 검사 결과는 boolean 값으로 반환된다.

<br>

#### 요소의 통계 - count, min, max

```java
IntStream stream1 = IntStream.of(4, 2, 7, 3, 5, 1, 6);
IntStream stream2 = IntStream.of(4, 2, 7, 3, 5, 1, 6);

System.out.println(stream1.count());           // 7
System.out.println(stream2.max().getAsInt());  // 7
```

* count() : 해당 스트림의 요소의 총 개수를 long 타입의 값으로 반환한다.
* min(), max() : 해당 스트림의 요소 중에서 가장 큰 값과 가장 작은 값을 가지는 요소를 참조하는 Optional 객체를 얻을 수 있다.

<br>

#### 요소의 연산 - sum, average

```java
IntStream stream1 = IntStream.of(4, 2, 7, 3, 5, 1, 6);
IntStream stream2 = IntStream.of(4, 2, 7, 3, 5, 1, 6);

System.out.println(stream1.sum());                   // 28
System.out.println(stream2.average().getAsDouble()); // 4.0
```

* IntStream이나 DoubleStream과 같은 기본 타입 스트림에는 해당 스트림의 모든 요소에 대해 합과 평균을 구할 수 있는 sum()과 average() 메서드가 각각 정의되어 있다.
* average() 메서드는 Double로 래핑 된 Optional 객체를 반환한다.

<br>

#### 요소의 수집 - collect

```java
Stream<String> stream = Stream.of("넷", "둘", "하나", "셋");

List<String> list = stream.collect(Collectors.toList());

Iterator<String> iter = list.listIterator();
while (iter.hasNext()) {
    System.out.print(iter.next() + " ");
}
```

* collect() : 인수로 전달되는 Collectors 객체에 구현된 방법대로 스트림의 요소를 수집한다.
* Collectors : 스트림 요소의 수집 용도로 사용하는 클래스. 미리 정의된 다양한 방법이 클래스 메서드로 정의되어 있다.
  * 스트림을 배열이나 컬렉션으로 변환 : toArray(), toCollection(), toList(), toSet(), toMap()
  * 요소의 통계와 연산 메서드와 같은 동작을 수행 : counting(), maxBy(), minBy(), summingInt(), averagingInt()
  * 요소의 소모와 같은 동작을 수행 : reducing(), joining()
  * 요소의 그룹화와 분할 : groupingBy(), partitioningBy()
  * 사용자가 직접 Collector 인터페이스를 구현하여 자신만의 수집 방법을 정의할 수 있다.

```java
Stream<String> stream = Stream.of("JAVA", "C", "C++", "PYTHON");

Map<Boolean, List<String>> patition 
    = stream.collect(Collectors.partitioningBy(s -> (s.length() % 2) == 0));

List<String> oddLengthList = patition.get(false);
System.out.println(oddLengthList);               // C, C++

List<String> evenLengthList = patition.get(true);
System.out.println(evenLengthList);               // JAVA, PYTHON
```

<br>

