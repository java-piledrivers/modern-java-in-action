# 람다 표현식을 조합할 수 있는 유용한 메서드
자바 8 API의 `Comparator`, `Function`, `Predicate` 와 같은 함수형 인터페이스는 람다 표현식을 조합할 수 있는 메서드를 제공한다.
람다 표현식을 조합할 수 있다는 것은 여러 개의 프레디케이트를 조합하여 하나의 큰 프레디케이트로 만들 수 있다는 것이다.   
[함수형 인터페이스]([https://github.com/java-piledrivers/modern-java-in-action/tree/main/Chapter%2003%20-%20%EB%9E%8C%EB%8B%A4%20%ED%91%9C%ED%98%84%EC%8B%9D/3.4](https://github.com/java-piledrivers/modern-java-in-action/tree/main/Chapter%2003%20-%20%EB%9E%8C%EB%8B%A4%20%ED%91%9C%ED%98%84%EC%8B%9D/3.2#%ED%95%A8%EC%88%98%ED%98%95-%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4))의
정의는 오직 하나의 추상 메서드를 지정하는 인터페이스인데 추가로 여러 메서드를 제공한다면 함수형 인터페이스의 정의에 어긋나다고 생각할 수 있지만 여기서 제공하는 
메서드는 **디폴트 메서드**(default method)임으로 정의의 어긋나지 않는다. 


## Comparator 조합

Comparator 를 사용하면 정렬을 수행할 수 있다.  


```java
Comparator<Apple> c = Comparator.comparing(Apple::getWeight);
```

만약에 오름차순이 아니라 내림차순으로 정렬하고자 한다면 인터페이스 자체에서 제공하는 *reverse*라는 디폴트 메서드를 사용하여 역정렬을 할 수 있다. 
이전 코드에서 디폴트 메서드만 추가해주면 되기에 다른 Comparator 인스턴스를 만들 필요가 없다. 


```java
inventory.sort(comparing(Apple::getWeight).reversed());
```

이제는 무게가 같은 사과에서는 원산지 국가별로 사과를 정렬하고 싶다. 이럴때에는 *thenComparing* 메서드를 사용하여 두 번째 비교자를 만들 수 있다. 

```java
inventory.sort(comparing(Apple::getWeight)
          .reversed()     // 내림차순 정렬
          .thenComparing(Apple::getCountry));     무게가 같다면 원산지 국가별로 정렬
```



## Predicate 조합
Predicate 인터페이스에서는 논리 게이트와 같이 negate, and, or 메서드를 제공한다.  

예를 들어 "빨간색이 아닌 사과"처럼 특정 프레디케이트를 반전시킬 때에는 negate 메서드를 사용하면 된다.  

```java
Predicate<Apple> notRedApple = redApple.negate();
```

무거운 사과와 빨간색 사과를 동시에 선택하고 싶으면 and 메서드를 사용하면 된다.  
```java
Predicate<Apple> redAndHeavyApple = redApple.and(apple -> apple.getWeight() > 150);
```

혹은 무거운 사과이면서 빨간색 사과 또는 그냥 녹색 사과와 같이 다양한 조건을 만들 수 있다.
```java
Predicate<Apple> redAndHeavyApple = redApple.and(apple -> apple.getWeight() > 150)
                                            .or(apple -> GREEN.equals(a.getColor()));
```

연산 순서는 왼쪽에서 오른쪽순이다. 

## Function 조합

Function 인터페이스에서는 Function 인스턴스를 반환하는 *andThen*, *compose* 두 가지 디폴트 메서드를 제공한다.   

*andThen* 메서드는 주어진 함수를 먼저 적용한 결과를 다른 함수의 입력으로 전달하는 함수를 반환하는 반면, *compose* 메서드는 인수로 주어진 함수를 먼저 실행한 다음에
그 결과를 외부 함수의 인수로 제공한다.  

```java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;

Function<Integer, Integer> h1 = f.andThen(g);
Function<Integer, Integer> h2 = f.compose(g);

int result1 = h1.apply(1); // (1+1)*2 => 4를 반환
int result2 = h2.apply(1); // (1*2)+1 => 3을 반환
```


