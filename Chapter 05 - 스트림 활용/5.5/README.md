# 5.5 리듀싱

## 왜 이름이 리듀싱?

- "리듀싱"이란 단어는 "줄이다"는 의미를 가짐 
- 스트림에서 리듀싱은 스트림의 모든 요소를 `하나의 값`으로 줄이는(reduce) 작업을 수행하기 때문

## 사용 목적

- 리듀싱은 대개 스트림의 모든 요소를 하나의 값으로 합치기 위해 사용됨
  - 총합
  - 평균값
  - 최댓값
  - 최솟값

## 리듀싱과 for-loop

```java
int sum = 0;
for (int x : numbers) {
	sum += x;
}
```

for-loop 에서 사용된 파라미터는 `sum`, 리스트의 모든 요소를 조합하는 연산(+)이다.

곱하는 연산을 하려면 코드를 복붙해야 함.

```java
int result = 1;
for (int x : numbers) {
	result *= x;
}
```

이를 reduce 를 사용하면 간단하게 반복되는 패턴을 추상화 가능

```java
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
int result = numbers.stream().reduce(1, (a, b) -> a * b);
```

## 리듀싱 연산 인터페이스

1. `T reduce(T identity, BinaryOperator<T> accumulator)` : 초기값(identity)을 제공하고, 각 요소를 누적해 합칩니다.

```java
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
int result = numbers.stream().reduce(1, (a, b) -> a * b);
```

2. `Optional<T> reduce(BinaryOperator<T> accumulator)` : 초기값을 제공하지 않고, 각 요소를 누적해 합칩니다. 결과가 없을 수 있으므로 Optional 클래스를 반환합니다.

```java
Optional<Integer> min = numbers.stream().reduce(Integer::min);
```

3. `<U> U reduce(U identity, BiFunction<U,? super T,U> accumulator, BinaryOperator<U> combiner)` : 병렬 처리를 위한 reduce 메소드이며, 컨테이너와 요소를 연결하는 accumulator와 병렬처리를 위해 컨테이너를 병합하는 combiner를 인수로 받습니다.

### combiner 함수가 무엇인지 자세히 알아보자

combiner 함수는 스트림의 병렬 처리 과정에서 사용되는 함수로, 스트림의 요소를 여러 개의 작은 조각으로 나누고, 그 작은 조각들을 각각 병렬로 처리한 후, 처리된 결과를 다시 병합하여 하나의 최종 결과로 만드는 작업에서 사용된다.

예를 들어, 1부터 10까지의 숫자가 들어있는 리스트가 있다고 가정해보자. 이 리스트를 병렬 처리하기 위해서, 이 리스트를 여러 개의 작은 리스트로 나누고, 각각의 작은 리스트를 병렬로 처리한다. 이렇게 처리된 작은 리스트들의 결과를 병합하여 최종 결과를 만들어내는 것이 병렬 처리의 핵심이다.

[1.2 에서 언급한 다중 코드 사본 예시 코드](https://github.com/java-piledrivers/modern-java-in-action/tree/main/Chapter%2001%20-%20%EC%9E%90%EB%B0%94%208%2C%209%2C%2010%2C%2011%20%EB%AC%B4%EC%8A%A8%20%EC%9D%BC%EC%9D%B4%20%EC%9D%BC%EC%96%B4%EB%82%98%EA%B3%A0%20%EC%9E%88%EB%8A%94%EA%B0%80/1.2#%EB%B3%91%EB%A0%AC%EC%84%B1%EA%B3%BC-%EA%B3%B5%EC%9C%A0-%EA%B0%80%EB%B3%80-%EB%8D%B0%EC%9D%B4%ED%84%B0)

위 코드에서 마지막에 sum 을 만드는 로직이 combiner 이다.

```java
        int sum = 0;
        for (int i = 0; i < numOfThreads; i++) {
            sum += sums[i];
        }
```
