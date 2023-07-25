# 자바 10 지역 변수형 추론

자바에서 변수가 메서드를 정의할 때 형식을 지정해야 한다.

```java
double convertUSDToGBP(double money) {ExchangeRage e = ...;}
```

해당 예시는 convertUSDToGBP의 결과, 인수 money, 지역 변수 e라는 세 가지 형식이 사용되었다.

하지만 시간이 지나면서 엄격한 형식 지정이 조금은 느슨해진 결과, 컨텍스트로 형식을 유추할 수 있는 상황에서는 일부 표현을 생략할 수 있게 되었다.

```java
Map<String, List<String>> myMap = new HashMap<String, List<String>>();
Map<String, List<String>> myMap = new HashMap<>();

Function<Integer, Boolean> p = (Integer x) -> booleanExpression;
Function<Integer, Boolean> p = x -> booleanExpression;
```

형식이 생략되면 컴파일러가 생략된 형식을 추론하는데, 한 개의 식별자로 구성된 형식에 형식 추론을 사용하게 될 경우 한 형식을 다른 형식으로 교체할 때의 편집 작업이 줄어든다는 점과 형식의 크기가 커서 제네릭이 다른 제네릭 형식에 의해 파라미터화될 때 가독성이 좋아질 수 있다는 장점이 있다.

자바 10에는 지역 변수형 추론이라는 기능이 추가되었는데, 지역 변수의 형식을 var 키워드로 대체하면 컴파일러가 변수 할당문의 오른쪽 내용을 기초로 형식을 추론하는 것이다. 하지만 초깃값이 없을 때는 var 키워드를 사용할 수 없음이 공식적으로 지정되어 있다.

예를 들어 myMap을 다음과 같이 선언할 수 있다.

```java
var myMap = new HashMap<String, List<String>>();
```
