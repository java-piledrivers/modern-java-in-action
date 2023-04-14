# Collector 인터페이스

Collector 인터페이스는 리듀싱 연산을 어떻게 구현할지 제공하는 메서드 집합으로 구성된다.

Collector의 시그니쳐와 메서드는 아래의 코드와 같다.


```java
public interface Collector<T, A, R> {
    
    Supplier<A> supplier();

    BiConsumer<A, T> accumulator();

    BinaryOperator<A> combiner();

    Function<A, R> finisher();

    Set<Characteristics> characteristics();
}

```


`T`: 수집될 항목의 제너릭 형식  
`A`: 누적자, 수집 과정에서 중간 결과를 누적하는 객체의 형식  
`R`: 수집 연산 결과 객체의 형식. (대게 컬렉션 형식)  


---



이제 각각의 메서드가 어떤 일들을 수행해주는지 살펴보자.

예를 들어 Stream의 모든 요소를 List로 수집하는 ToListCollector라는 클래스를 구현할 수 있다.


```java
public class ToListCollector<T> implements Collector<T, List<T>, List<T>>

```
  
## supplier 메서드: 새로운 결과 컨테이너 만들기

<br>

supplier 메서드는 한마디로 작업 결과를 저장할 공간을 제공해주는 역할을 한다.


```java
public Supplier<List<T>> supplier() {
    return () -> new ArrayList<>();
}
```

람다를 사용하면 아래와 같이 줄일수 있다.

```java
public Supplier<List<T>> supplier() {
    return ArrayList::new;
}
```


## accumulator 메서드: 결과 컨테이너에 요소 추가하기

accumulator 메서드는 리듀싱 연산을 수행하는 함수를 반환한다. 스트림에서 n번째 요소를 탐색할 때, n-1 번까지의 항목을 수집한 결과에(누적자) n번째 요소를 함수에 적용한다. 아무것도 반환하지 않는다.

```java
public BiConsumer<List<T>, T> accumulator() {
	return (list, item) -> list.add(item);
}
```

이 코드 역시 람다로 간결하게 표현 가능하다. 

```java
public BiConsumer<List<T>, T> accumulator() {
    return List::add;
}
```


## finisher 메서드: 최종 변환값을 결과 컨테이너로 적용하기

finisher 메서드는 스트림 탐색을 끝내고 누적자 객체를 최종 결과로 변환하면서 누적 과정을 끝낼 때 호출할 함수를 반환해야 한다.
만약 누적자 객체가 이미 최종 결과라면 변환 과정이 필요하지 않으므로 항등 함수를 반환하면 된다. 

```java
public Function<List<T>, List<T>> finisher() {
	return Function.identity(); // 항등 함수 return x -> x; 와 동일
}
```

![순차 리듀싱 과정의 논리적 순서](https://github.com/java-piledrivers/modern-java-in-action/blob/main/Chapter%2006%20-%20%EC%8A%A4%ED%8A%B8%EB%A6%BC%EC%9C%BC%EB%A1%9C%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%88%98%EC%A7%91/6.5/%EC%88%9C%EC%B0%A8%20%EB%A6%AC%EB%93%80%EC%8B%B1%20%EA%B3%BC%EC%A0%95.png)


## combiner 메서드: 두 결과 컨테이너 병합


combiner 메서드는 스트림의 서로 다른 서브파트를 병렬로 처리할 때 누적자가 이 결과를 어떻게 처리할 지 정의한다.


```java
public BinaryOperator<List<T>> combiner() {
	return (list1, list2) -> {
    	list1.addAll(list2);
        return list1;
    }
}
```

<br>

위와 같이 코드를 작성하면, 스트림의 두 번째 서브 파트에서 수집한 항목 리스트를 첫 번째 서브파트 결과 리스트의 뒤에 추가할 수 있다.

<br>

## Characteristics 메서드

Characteristics 메서드는 컬렉션의 연산을 정의하는 Characteristics 형식의 불변 집합을 반환한다.  

다른 메서드는 모두 반환타입이 함수형 인터페이스였지만 characteristics 메서드는 Set을 반환한다. Characteristics 에는 아래와 같은 enum 으로 구성되어 있다. 

```java
enum Characteristics {

    CONCURRENT,
    UNORDERED,
    IDENTITY_FINISH

}
```


`UNORDERED`: 리듀싱 결과는 스트림 요소의 방문 순서나 누적 순서에 영향을 받지 않는다.  
`CONCURRENT`: 다중 스레드에서 accumulator 함수를 동시에 호출할 수 있으며 이 컬렉터는 스트림의 병렬 리듀싱을 수행할 수 있다. 컬렉터의 플래그에 UNORDERED를 함께 설정하지 않았다면 데이터 소스가 정렬되어 있지 않은 상황에서만 병렬 리듀싱을 수행할 수 있다.  
`IDENTITY_FINISH`: finisher 메서드가 반환하는 함수는 단순히 identity를 적용할 뿐이므로 이를 생략할 수 있다. 따라서 리듀싱 과정의 최종 결과로 누적자 객체를 바로 사용할 수 있다. 또한 누적자 A를 결과 R로 안전하게 형변환 할 수 있다. 


지금까지 개발한 ToListCollector에서 스트림의 요소를 누적하는 데 사용한 리스트가 최종 결과 형식이므로 추가 변환이 필요 없다. 따라서 ToListCollector는 `IDENTITY_FINISH`다. 하지만 리스트의 순서는 상관이 없으므로 `UNORDERED`다. 마지막으로 ToListCollector는 `CONCURRENT`다.


---

지금까지 알아본 메서드를 다시 살펴보자. 


```java
import java.util.*;
import java.util.function.*;
import java.util.stream.Collector;
import static java.util.stream.Collector.Characteristics.*;

public class ToListCollector<T> implements Collector<T, List<T>, List<T>> {

    @Override
    public Supplier<List<T>> supplier() {
        return ArrayList::new; // 수집 연산의 시발점
    }

    @Override
    public BiConsumer<List<T>, T> accumulator() {
        return List::add;  // 탐색한 항목을 누적하고 바로 누적자를 고친다.
    }

    @Override
    public Function<List<T>, List<T>> finisher() {
        return Function.identity();  // 항등 함수
    }

    @Override
    public BinaryOperator<List<T>> combiner() {
        return (list1, list2) -> {  // 두 번째 콘텐츠와 합쳐서 첫 번째 누적자를 고친다.
            list1.addAll(list2);    // 변경된 첫 번째 누적자를 반환한다.
            return list1;
        };
    }

    @Override
    public Set<Characteristics> characteristics() {
        return Collections.unmodifiableSet(EnumSet.of(IDENTITY_FINISH, CONCURRENT));  // 컬렉터의 플래그를 IDENTITY_FINISH, CONCURRENT로 설정한다. 
    }
}
```

위 구현이 Collections.toList 메서드가 반환하는 결과와 완전히 같은 것은 아니지만 사소한 최적화를 제외하면 대체로 비슷하다. 

