#  4.1 스트림

자바 8 API에 새로 추가된 기능으로, 질의로 표현하는 방식인 선언형으로 컬렉션 데이터를 처리

## 스트림의 특징

-   **선언형**: 제어 블록(루프, 조건문)을 이용해서 어떻게 동작을 구현할지 지정할 필요 없이 동작의 수행 지정을 지정할 수 있다.
-   **조립 가능**: filter, sorted, map, collect와 같은 여러 빌딩 블록 연산을 연결해 복잡한 데이터를 처리하는 파이프라인을 만들 수 있다.
-   **병렬화**: filter와 같은 연산은 특정 스레딩 모델에 제한되지 않고 자유롭게 사용할 수 있다. 띠리서 데이터 처리 과정을 병렬화할 수 있다.

```java
//자바 7 코드
List<Dish> lowCaloricDishes = new ArrayList<>(); //중간 변수
for (Dish dish : menu) {
    if (dish.getCalories() < 400) {
        lowCaloricDishes.add(dish);
    }
}
Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
    public int compare(Dish dish1, Dish dish2) {
        return Integer.compare(dish1.getCalories(), dish2.getCalories());
    }
});

List<String> lowCaloricDishesName = new ArrayList<>();
for (Dish dish : lowCaloricDishes) {
    lowCaloricDishesName.add(dish.getName());
}
```

```java
//자바 8 코드
List<String> lowCaloricDishesName = menu.stream()
                                        .filter(d -> d.getCalories() < 400)
                                        .sorted(comparing(Dishes::getCalories))
                                        .map(Dish::getName)
                                        .collect(toList());
```

# 4.2 스트림 소개 : 스트림 시작하기

자바 8 컬렉션에는 스트림을 반환하는 stream 메서드가 추가됐다.

## 스트림이란?
데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소 (Sequence of elements)

### 정의
- 연속된 요소 : 컬렉션과 마찬가지로 스트림은 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스를 제공.   
  컬렉션은 자료구조이므로 컬렉션에서는 시간과 공간의 복잡성과 관련된 요소 저장 및 접근 연산이 주를 이룬다.    
  반면 스트림은 filter, sorted, map 처럼 표현 계산식이 주를 이룬다. **즉, 컬렉션의 주제는 데이터이고 스트림의 주제는 계산이다.**


- 소스 : 스트림은 컬렉션, 배열, I/O 자원 등의 데이터 제공 소스로부터 데이터를 소비한다. 정렬된 컬렉션으로 스트림을 생성하면 정렬이 그대로 유지된다.   
  즉, 리스트로 스트림을 만들면 스트림의 요소는 리스트의 요소와 같은 순서를 유지된다.


- 데이터 처리 연산 : 스트림은 filter, map, reduce, find, match, sort 등의 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산을 지원한다.     
  이 연산들은 스트림 파이프라인을 구성하는 기본 요소이다.

### 특징
- 파이프라이닝: 대부분 스트림 연산은 스트림 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 스트림 자신을 반환한다. 그 덕분에 게으름(laziness), 쇼트서킷(short-circuiting) 같은 최적화도 얻을 수 있다.
- 내부 반복: 반복자를 이용해서 명시적으로 반복하는 컬렉션과 달리 스트림은 내부 반복을 지원한다.

```java
List<String> threeHighCaloricDishNames = 
    menu.stream()
        .filter(d -> d.getCalories() > 300) // 고칼로리 필터링
        .map(Dish::getName) // 요리명 추출
        .limit(3) // 선착순 3개만
        .collect(toList()); // 리스트로 저장
```

마지막 collect(toList())를 호출하기 전까지는 menu에서 무엇도 선택되지 않으며 출력 결과도 없다. 
즉 collect가 호출되기 전까지 메서드 호출이 저장되는 효과가 있다.

# 4.3 스트림과 컬렉션

  스트림과 컬렉션은 순차적으로 값에 접근하여 값을 저장하는 자료구조의 인터페이스를 제공한다.



## 스트림과 컬렉션의 차이점

  언제 계산하느냐가 가장 큰 차이점 이다.

  컬렉션 : 모든 요소가 컬렉션에 들어가기 전에 계산이 되어야 한다.

  스트림 : 요청 할때만 요소를 계산한다.(게으르게 만들어진 컬렉션)



## 딱 한번만 탐색할수 있는 스트림


  한번쓰면 끝!! 
  
  ```
  List<String> title = Arrays.asList("JAVA8","In","Action");
  Stream<String> s = title.stream();
  s.forEach(System.out::println);  //출력됨
  s.forEach(System.out::println);  //안됨
  ```
  
  
## 외부 반복과 내부 반복
  
  외부반복 : 사용자가 직접 요소를 반복한다. (For-each)
  
  ```
  List<String> names = new ArrayList<>();
  for(Dish dish : menu){
    names.add(dish.getName());
  }
  ```
  
  내부반복 : 반복을 알아서 처리하고 결과 스트림 값을 저장한다.
  
  ```
  List<String> names = menu.steram().Map(Dish::getName).collect(toList());
  ```
  
 ## 내부반복을 사용하면 좋은점
  
  병렬성을 관리해주어야 하는 for-each 과 같은 외부 반복과 달리 
  
  내부 반복은 병렬성 구현을 자동으로 선택한다 



# 4.4 스트림 연산

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




