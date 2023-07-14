# 스트림과 게으른 평가

스트림은 데이터 컬렉션을 처리하는 편리한 도구이지만, 단 한 번만 소비할 수 있다는 제약이 있어 재귀적으로 정의할 수 없다. 이번 절에서는 이와 같은 제약으로 인해 발생하는 문제에 대해 살펴본다.

## 자기 정의 스트림

소수로 나눌 수 있는 수를 제외하는 코드를 다음과 같은 과정을 통해 구현할 수 있다.

1. 스트림 숫자 얻기 - IntStream.iterate 메서드를 이용하면 2부터 시작하는 무한 숫자 스트림을 생성할 수 있다.
  
2. 머리 획득 - IntStream은 첫 번째 요소를 반환하는 findFirst라는 메서드를 제공한다.
  
3. 꼬리 필터링 - 스트림의 꼬리를 얻는 메서드를 정의하여 획득한 머리로 나누어지지 않는 숫자를 필터링한다.
  
4. 재귀적으로 소수 스트림 생성 - 반복적으로 머리를 얻어서 스트림을 필터링한다.
  

이러한 로직을 구현할 경우, 4단계 코드를 실행 시 `java.lang.IllegalStateException: stream has already been operated upon or closed.`라는 에러가 발생하게 된다. 이는 스트림이 완전 소비되어 발생하는 오류이다.

4단계 코드에서 발생하는 또 다른 문제가 있는데, IntStream.concat을 이용하는 과정에서 메서드의 인수가 무한 숫자 스트림을 재귀적으로 호출하면서 무한 재귀에 빠지게 된다는 점이다.

결론적으로는 재귀적으로 호출하는 인수를 게으르게 평가하는 방식으로 문제를 해결할 수 있는데, 이는 소수를 처리할 필요가 있을 때만 스트림을 실제로 평가하는 것을 의미한다.

## 게으른 리스트 만들기

자바 8의 스트림에 일련의 연산을 적용하면 연산이 수행되지 않고 일단 저장해두었다가, 스트림에 최종 연산을 적용해서 실제 계산을 해야 하는 상황에서만 실제 연산이 이루어진다.

게으른 리스트는 다음과 같이 구현할 수 있다.

```java
class LazyList<T> implements MyList<T> {
    final T head;
    final Supplier<MyList<T>> tail;

    public LazyList(T head, Supplier<MyList<T>> tail) {
        this.head = head;
        this.tail = tail;
    }

    public T head() {
        return head;
    }

    public MyList<T> tail() { //head와 달리 tail에서는 Supplier로 게으른 동작 생
        return tail.get();   
    }

    public boolean isEmpty() {
        return false;
    }    
}
```

앞서 살펴본 소수 생성 예시로 돌아와보면 게으른 리스트를 이용해 다음과 같이 구현할 수 있다.

```java
public static MyList<Integer> primes(MyList<Integer> numbers) {
    return new LazyList<>(
        numbers.head(),
        () -> primes(
            numbers.tail()
                    .filter(n -> n % numbers.head() != 0) //해당 메서드는 추가적으로 정의해주어야 함.
        )
    );    
}
```

```java
LazyList<Integer> numbers = from(2);
int two = primes(numbers).head(); //2
int three = primes(numbers).tail().head(); //3
int five = primes(numbers).tail().tail().head(); //5
```
