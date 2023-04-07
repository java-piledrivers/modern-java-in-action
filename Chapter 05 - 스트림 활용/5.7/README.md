# 숫자형 스트림

아래와 같은 코드를 통해 메뉴의 칼로리 합계를 구할 수 있다.

```java
int calories = menu.stream()
                   .map(Dish::getCalories)
                   .reduce(0, Integer::sum);
```

하지만 이 코드는 합계를 계산하기 전에 Integer를 기본형으로 언박싱하는 과정을 거치기 때문에 박싱 비용이 추가된다.  
그 이유는 map 메서드가 Stream를 생성하기 때문인데, 이러한 상황에서는 스트림 API 숫자 스트림을 효율적으로 처리할 수 있도록 기본형 특화 스트림을 제공한다.

## 기본형 숫자 스트림

자바 8에서는 세 가지 기본형 특화 스트림을 제공하는데, int 요소에 특화된 IntStream, double 요소에 특화된 DoubleStream, long 요소에 특화된 LongStream이 그것이다. 각 인터페이스는 sum, max 같이 자주 사용하는 숫자 관련 리듀싱 연산 수행 메서드를 제공하며 필요 시 다시 객체 스트림으로 복원하는 기능도 제공하고 있다.

### 숫자 스트림으로 매핑

스트림을 특화 스트림으로 변환할 때는 mapToInt, mapToDouble, mapToLong과 같은 메서드를 사용하며, 반환 값으로 특화된 스트림을 반환한다.

### 객체 스트림으로 복원하기

숫자 스트림을 원상태인 특화되지 않은 스트림으로 복원할 수도 있는데, boxed 메서드를 이용해서 특화 스트림을 일반 스트림으로 변환할 수 있다.

### 기본값 : OptionalInt

스트림에 요소가 없는 상황과 실제 최댓값이 0인 상황을 구별하기 위해서 컨테이너 클래스인 Optional을 이용할 수 있는데, Optional을 Integer, String 등의 조 형식으로 파리미터화할 수 있으며 OptionalInt와 같이 기본형 특화 스트림 버전도 제공한다.  
아래 코드를 통해 OptionalInt를 이용해서 IntStream의 최댓값 요소를 찾을 수 있다.

```java
OptionalInt maxCalories = menu.stream()
                              .mapToInt(Dish::getCalories)
                              .max();

maxCalories.orElse(1); //값이 없을 때 기본 최댓값을 명시적으로 설정
```

## 숫자 범위

자바 8의 IntStream과 LongStream에서는 range와 rangeClosed라는 두 가지 정적 메서드를 제공하는데 공통적으로 첫 번째 인수로 시작 값을, 두 번째 인수로 종료 값을 갖는다. 차이점이 있다면 range는 시작 값과 종료 값이 결과에 포함되지 않는 반면 rangeClosed는 결과에 포함된다는 점이다.

```java
IntStream evenNumbers = IntStream.rangeClosed(1, 100)
                                 .filter(n -> n % 2 == 0)
System.out.println(evenNumbers.count()); //50개의 짝수 출력
```

## 숫자 스트림 활용 : 피타고라스 수

```java
Stream<double[]> pythagoreanTriples2 = IntStream.rangeClosed(1, 100) //1부터 100까지 b값 생성
                                                .boxed()
                                                .flatMap(a -> IntStream.rangeClosed(a, 100)) //생성된 각각의 스트림을 하나의 평준화된 스트림으로 변환
                                                .mapToObj(
                                                    b -> new double[]{a, b, Math.sqrt(a*a + b*b)}) //스트림의 각 요소를 double로 반환하기 위함
                                                .filter(t -> t[2] % 1 == 0);
```
