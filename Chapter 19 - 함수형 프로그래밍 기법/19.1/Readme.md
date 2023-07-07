## 함수는 모든 곳에 존재한다
함수형 언어 프로그래머는 함수를 값으로 취급할 수 있음을 의미하는 일급 함수 개념을 폭넓게 사용.

자바 8은 이전 버전과 구별되는 특징 중 하나로 일급 함수를 지원. 

이를 위해 메서드 참조나 람다 표현식으로 함숫값을 직접 표현하고, 함수를 인수나 결과로 사용할 수 있다

```java
Function<String, Integer> strToInt = Integer::parseInt;
```

### 19.1.1 고차원 함수

Comparator.comparing처럼 다음 중 하나 이상의 동작을 수행하는 함수를 고차원 함수라 부른다.

- 하나 이상의 함수를 인수로 받음
- 함수를 결과로 반환

```java
Comparator<Apple> c = comparing(Apple::getWeight);

// 함수를 조립해서 연산 파이프라인을 만들 때 위 코드와 비슷한 기능을 활용
Function<String, Integer> transformationPipeline = addHeader.andThen(Letter::checkSpelling)
                                                            .andThen(Letter::addFooter);
```

### 19.1.2 커링

커링은 x와 y라는 두 인수를 받는 함수 f를 한 개의 인수를 받는 g라는 함수로 대체하는 기법이다.

```java
static double converter(double x, double f, double b) {
  return x * f + b;
}
...
double gbp = converter(1000, 0.6, 0);

//커링을 활용한 팩토리로 정의static DoubleUnaryOperator curriedConvergter(double f, double b) {
  return (double x) -> x * f + b;
}
...
DoubleUnaryOperator convertUSDtoGBP = curriedConvergter(0.6, 0);
double gbp = convertUSDtoGBP.applyAsDouble(1000);
```

첫 번째 함수는 인수 x, f, b를 항상 입력해줘야 한다.

두 번째 함수는 f, b 두 가지 인수만 함수에 요청하고, 반환된 함수에 인수 x를 이용해서 결과를 얻을 수 있다.
