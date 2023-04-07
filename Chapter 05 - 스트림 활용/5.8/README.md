# 5.8 스트림 만들기

## 5.8.1 값으로 스트림 만들기
임의의 수를 인수로 받는 정적 메서드 Stream.of를 이용해서 스트림을 만들 수 있다.

```java
Stream<String> stream = Stream.of("Java 8 ", "Lambdas ", "In ", "Action");
stream.map(String::toUpperCase).forEach(System.out::println);
```

다음처럼 empty 메서드를 이용해서 빈 스트림을 만들 수도 있다.

```java
Stream<String> emptyStream = Stream.empty();
```

## 5.8.2 null이 될 수 있는 객체로 스트림 만들기
null이 될 수 있는 객체로 스트림을 만들 때는 Stream.ofNullable 메서드를 사용한다.   
ofNullable 메서드는 null 일때는 빈 스트림을 리턴, 아니면 스트림을 리턴한다.

```java
String value = null;
Stream<String> stream = Stream.ofNullable(value);
// Stream<String> nullableStream = value == null ? Stream.empty() : Stream.of(value);
```

## 5.8.3 배열로 스트림 만들기
배열로 스트림을 만들 때는 Arrays.stream 메서드를 사용한다.

```java
int[] numbers = {2, 3, 5, 7, 11, 13};
int sum = Arrays.stream(numbers).sum();
```

## 5.8.4 파일로 스트림 만들기
파일로 스트림을 만들 때는 Files.lines 메서드를 사용한다.

```java
long uniqueWords = 0;
try (Stream<String> lines = Files.lines(Paths.get("data.txt"), Charset.defaultCharset())) {
    uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" "))) // 띄어쓰기 기준으로 배열화
                       .distinct() // 중복 제거
                       .count();
} catch (IOException e) {
}
```

## 5.8.5 함수로 무한 스트림 만들기
iterate와 generate 메서드는 무한 스트림을 만들 수 있다.   
iterate와 generate에서 만든 스트림은 요청할 때마다 주어진 함수를 이용해서 값을 만든다. 따라서 무제한은 값을 계산할 수 있다. 하지만 보통 무한한 값을 출력하지 않도록 limit(n) 함수를 함께 연결해서 사용한다.

### iterate
iterate는 초깃값과 람다를 인수로 받아서 새로운 값을 끊임없이 생산할 수 있고, 초깃값은 스트림 첫번째 요소로 반환된다.

```java
Stream.iterate(0, n -> n + 2)
      .limit(10)
      .forEach(System.out::println);
```

기본적으로 iterate는 기존 결과에 의존해서 순차적으로 연산을 수행하고, 요청할 때마다 값을 생산할 수 있으며 끝이 없으므로 무한 스트림을 만든다. 이러한 스트림을 언바운드 스트림(unbounded stream)이라고 표현한다.

자바 9의 iterate 메서드는 두 번째 인수로 Predicate를 받는다. 이 Predicate는 스트림의 다음 요소를 생성할지 말지를 결정한다.

```java
Stream.iterate(0, n -> n < 100, n -> n + 2)
      .forEach(System.out::println);
```

위의 식을 takewhile을 이용하여 표현하면 다음과 같이 표현할 수 있다.    
    
```java
Stream.iterate(0, n -> n + 2)
      .takeWhile(n -> n < 100) 
      .forEach(System.out::println);
```

takeWhile 대신 filter 메서드를 사용하면 이 스트림은 종료되지 않는다. filter 메서드는 이 작업을 언제 중단해야 하는지를 알 수 없기 때문이다.

### generate
iterate와 비슷하게 generate도 요구할 때 값을 계산하는 무한 스트림을 만들 수 있다.

하지만 iterate와 달리 생성된 각 값을 연속적으로 계산하지 않는다.

```java
Stream.generate(Math::random)
      .limit(5)
      .forEach(System.out::println);
```

일반적으로 연속된 일련의 값을 만들 때는 iterate를 사용하고, 병렬 처리처럼 상태가 없어야 하는 상황에는 generate를 사용한다.   
스트림을 병렬로 처리하면서 올바른 결과를 얻으려면 상태가 없어야 하고, 만약 있다면 변경되어서는 안된다.


generate 메서드를 이용해서도 피보나치수열 집합 연산 코드를 작성할 수 있다.

```java
IntSupplier fib = new IntSupplier() {
		private int previous = 0;
    private int current = 1;

    @Override
    public int getAsInt() {
		    int oldPrevious = this.previous;
        int nextValue = this.previous + this.current;
        this.previous = this.current;
        this.current = nextValue;
        return oldPrevious;
    }
};

IntStream.generate(fib)
         .limit(10)
         .forEach(System.out::println
```

위 코드에서의 IntSupplier 인스턴스는 기존 피보나치 요소와 두 인스턴스 변수에 어떤 피보나치 요소가 들어있는지 추적하므로 가변(mutable) 상태 객체다. iterate를 사용했을 때는 각 과정에서 새로운 값을 생성하면서도 기존 상태를 바꾸지 않는 순수한 불변(immutable) 상태를 유지했다.    
→ 예제를 위해 가변 객체가 되는 코드를 작성한 것이지, 스트림을 병렬로 처리하면서 올바른 결과를 얻으려면 불변 상태 기법을 고수해야 한다.
