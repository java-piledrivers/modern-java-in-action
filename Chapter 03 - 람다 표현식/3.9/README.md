# 비슷한 수학적 개념

람다 표현식과 함수전달은 비슷하다.

공학에서는 함수가 차지하는 영역을 묻는 질문이 자주 등장한다.   
이것을 적분이라고 하는 듯.  

$$\int_{3}^{7} (x + 10) dx$$

![KakaoTalk_Photo_2023-03-25-08-00-49](https://user-images.githubusercontent.com/59721293/227658749-13326cca-b9ac-43e2-bd97-b857bacb8558.jpeg)


사다리꼴 범위를 구하는 방법은 다음과 같다

`1/2 x ((3+10) + (7+10)) x (7-3) = 60`

이것을 함수로 나타내면 `integrate(f,3,7)` 처럼 사용할 수 있다.

이것을 다시 자바 버전으로 바꾸면 `integrate((double x) -> x + 10, 3, 7)` 로 바꿀 수 있다.

`(double x) -> x + 10` 는 `Function<T, R>` 을 사용해서 메소드 시그니쳐는 `public double integrate(Function<T,R> f, double a, double b)` 가 된다.

사다리꼴 범위를 구하는 방법을 구현코드로 바꾸면 아래처럼 된다.

```java
public double integrate(Function<T,R> f, double a, double b) {
	return (f.apply(a) + f.apply(b)) * (b - a) / 2.0;
}
```

수학처럼 f(a) 라고 표현을 할 수 없고 f.apply(a) 라고 구현하는 것은 자바가 진정으로 함수를 허용하지 않고 모든 것을 객체로 여기는것을 포기할 수 없기 때문이다.
