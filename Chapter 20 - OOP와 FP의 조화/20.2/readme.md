## 함수

스칼라의 함수는 어떤 작업을 수행하는 일련의 명령어 그룹이다. 스칼라의 함수는 일급값이다.

```
def isJavaMentioned(tweet: String) : Boolen = tweet.contains("Java")
def isShortTweet(tweet: String) : Boolean = tweet.lenght < 20
```

### 익명함수

스칼라는 익명 함수의 개념을 지원한다. 스칼라는 람다 표현식과 비슷한 문법을 통해 익명 함수를 만들 수 있다.

```
(x: Int) => x * x // 제곱
```

위의 익명함수는 사실 아래와 같은 정의를 단순화 한 것이다.

```
new Funtion1[Int, Int] {
	def apply(x: Int): Int = x * x
}
```

이때 apply 메서드는 스칼라의 트레이트(trait) 통해 지원되는 메서드이다. 스칼라에서는 Function0(인수가 없으며 결과를 반환)에서 Function22(22개의 인수를 받음)를 제공한다(모두 apply 메서드를 정의한다).

### 클로저(closure)

클로저란 함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스를 가리킨다. 자바의 람다 표현식에는 람다가 정의된 메서드의 지역 변수를 고칠 수 없다는 제약이 있으며, 암시적으로 final로 취급된다.

스칼라의 익명 함수는 값이 아니라 변수를 캡처할 수 있다.

```
def main(args: Array[String]) {
	var count = 0
	val inc = () => count+=1
	inc() // count를 캡처하고 증가시키는 클로저
	println(count)
	inc()
	println(count)
}
```

### 커링(currying)

커링은 x와 y라는 두 인수를 받는 함수 f를 한개의 인수를 받는 g라는 함수로 대체하는 기법이다. 스칼라는 커링을 자동으로 처리하는 특수 문법을 제공한다. 그렇기에 커리된 함수를 직접 만들어 제공할 필요가 없다.

`def multiplyCurry(x: Int)(y: Int) = x * y
val r1 = multiplyCurry(2)(10) // result = 20
val multiplyByTwo : Int => Int = multiplyCurry(2) // 부분 적용된 함수라 부른다.
val r2 = multiplyByTwo(10) // result = 20`
