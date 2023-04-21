# 7.1 병렬 스트림

병렬 스트림이란 각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할한 스트림이다.   
따라서 병렬 스트림을 이용하면 모든 멀티코어 프로세서가 각각의 청크를 처리하도록 할당할 수 있어야 한다.

병렬 스트림을 알기 전에,   
**스트림의 여러 청크를 병렬로 처리하기 전에 병렬 스트림이 요소를 여러 청크로 분할하는 방법을 알아야한다.**

1부터 n까지 합을 더하는 예제 코드
```java
public long sequentialSum(long n) {
	return Stream.iterate(1L, i -> i + 1) // 무한 자연수 스트림 생성
                 .limit(n) // n개 이하로 제한
                 .reduce(0L, Long::sum); // 모든 숫자를 더하는 리듀싱 연산
}
```
**이 코드에서 n이 커지면 연산을 병렬로 처리하는 것이 좋을 것이다.**

병렬로 처리하기 위해서 다음과 같은 고민이 필요하다.

1. 어떻게 동기화해야 할까? [이슈](https://github.com/java-piledrivers/modern-java-in-action/issues/12#issuecomment-1515464287)
2. 몇개의 스레드를 사용해야할까?
3. 숫자는 어떻게 생성할까?
4. 생성된 숫자는 누가 더할까?

자바의 병렬 스트림을 이용하면 위 문제를 고민하지 않아도 된다.

##  병렬 스트림이 요소를 여러 청크로 분할하는 방법

병렬로 처리하는 1부터 n까지 합을 더하는 예제 코드
```java
public long sequentialSum(long n) {
	return Stream.iterate(1L, i -> i + 1) 
                 .limit(n) 
                 .parallel() // 스트림을 병렬 스트림으로 전환
                 .reduce(0L, Long::sum); 
}
```
이전 코드와 다른 점은 스트림이 여러 청크로 분할되어 있다는 점이다.   
`parallel()` 을 추가하면 위 리듀싱 연산이 여러 청크로 분할되어진다.   

리듀싱 연산이 여러 청크로 분할되어 진다는 것은 아래 그림처럼   
리듀싱 연산 자체를 나누어서 결과를 마지막에 다시 리듀싱한다는것이다.   

그림에서 `n = 8` 이다.

![2023-04-20_07-40-52](https://user-images.githubusercontent.com/59721293/233215430-d93c0a57-671d-4d3e-a813-1a7e91980e13.jpg)

> [Q) 순차,병렬 스트림에 같이 넣었을 때 244p](https://github.com/java-piledrivers/modern-java-in-action/issues/12#issuecomment-1517017089)

## 스트림 성능 측정

반복형, 순차 리듀싱, 병렬 리듀싱.. 어떤 방법이 가장 빠를까?

> 병렬화를 이용하면 순차나 반복 형식에 비해 성능이 더 좋아질 것이라 추측할 것이다. 
> 소프트웨어 공학에서 추측은 위험한 방법이다. 
> 특히 성능을 최적화 할때는 세 가지 황금 규칙을 기억해야 한다. 첫째도 측정, 둘째도 측정, 셋째도 측정!

JMG 플러그인 사용해서 아래처럼 벤치마크할 수 있다.
```java
@BenchmarkMode(Mode.AverageTime) // 평균 시간 측정
@OutputTimeUnit(TimeUnit.MILLISECONDS) // 밀리초단위로 출력
@Fork(2, jvmArgs={"-Xms4G", "-Xmx4G"}) // 4GB의 힙 공간을 제공한 환경에서 두번 벤치마크를 수행해 결과의 신뢰성확보
public class ParallelStreamBenchmark {
	private static final long N = 10_000_000L;

	@Benchmark // 벤치마크 대상 메서드
	public long sequentialSum() {
		return Stream.iterate(1L, i -> i + 1).limit(N)
		             .reduce(0L, Long::sum);
	}

	@TearDown(Level.Invocation) // 매 번 벤치마크를 실행한 다음에는 gc 동작 시도
	public void tearDown() {
		System.gc();
	}
}
```

[Q) 왜 iterate 문을 병렬로 처리한 것은 스트림을 병렬로 처리할 수 없었을까? 249p](https://github.com/java-piledrivers/modern-java-in-action/issues/12#issuecomment-1517034159)

### 더 특화된 메서드 사용

```java
@Benchmark
public long rangedSum() {
	return LongStream.rangeClosed(1, N)
	                 .parallel() // 넣고 벤치마크, 안넣고 벤치마크 시도
	                 .reduce(0L, Long::sum);
}
```

결과적으로 parallel() 넣었을 때 더 빠른 속도를 보여준다.
LongStream.rangeClosed 는 기본형 long 을 직접사용하여 박싱 언박싱 오버헤드가 사라지고 쉽게 청크를 분할할 수 있는 숫자 범위를 생성한다. (iterate 도 limit 하는데 왜 이거만 청크로 분할할수 있다는 건지 모르겠음)

상황에 따라서는 어떤 알고리즘을 병렬화하는 거보다 적절한 자료구조를 선택하는 것이 병렬화에 더 중요하다는 것을 알 수 있다.
함수형 프로그래밍을 올바로 사용하면 반복적으로 코드를 실행하는 방법에 비해 멀티 코어 CPU 를 사용하는 병렬 실행의 힘을 느낄 수 있다.

하지만 병렬화를 이용하려면 스트림을 재귀적으로 분할해야하고 각 서브스트림을 서로 다른 스레드의 리듀싱 연산으로 할당하고, 이들 결과를 하나의 값으로 합쳐야한다.
멀티코어 간의 데이터 이동은 우리 생각보다 비싸다. 코어 간에 데이터 전송시간보다 훨씬 오래걸리는 작업만 병렬로 다른 코어에서 수행하는 것이 바람직하다. 

## 병렬 스트림의 올바른 사용법
```java
public long sideEffectSum(long n) {
	Accumulator accmulator = new Accumulator();
	LongStream.rangeClosed(1, n).forEach(accumulator::add);
	return accumulator.total;
}

public class Accumulator {
	public long total = 0;
	public void add(long  value) { 
		total += value;
	}
}
```
병렬 실행시 공유된 변수 사용 피해야한다.

## 병렬 스트림 효과적으로 사용하기

"몇 개 이상" 에선 병렬을 사용하라 같은 기준은 없다.

1. 확신이 서지 않으면 직접 비교 측정하라. 무조건 병렬 스트림을 바꾸는 것은 능사가 아니다. 적절한 벤치마크로 직접 성능 측정이 바람직하다
2. 박싱을 주의하라. 성능을 크게 저하시킨다.
3. 순차 보다 병렬에서 성능이 떨어지는 연산을 피해라. limit, findFirst 같은 요소의 순서에 의존하는 연산을 병렬 스트림에서 하려면 비싼 비용을 치러야한다. 예를 들어 findANy는 요소의 순서와 상관없이 연산해서 findFirst보다 성능이 좋다, 
4. 스트림에서 수행하는 전체 파이프라인 연산 비용을 고려하라. 한 요소를 처리하는 데에 드는 비용이 높다면 병렬 스트림으로 성능 개선가능하다.
5. 소량의 데이터는 도움이 되지 않는다.  병렬화 과정에서 생기는 부가 비용을 상쇄할 만큼 이득을 얻지 못한다
6. 스트림을 구성하는 자료구조가 적절한지 확인하라.
7. 254p 에서 더 보기.

마지막으로 병렬 스트림이 수해되는 내부 인프라구조도 살펴봐야한다.
