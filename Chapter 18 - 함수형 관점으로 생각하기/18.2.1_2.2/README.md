# 18.2 함수란 프로그래밍이란 무엇인가?

- 함수형 프로그래밍에서 함수란 수학적인 함수와 같다.
- 0개 이상의 인수를 가지며, 한 개 이상의 결과를 반환하지만 부작용이 없어야한다.
- 수학적 함수에서는 인수가 같다면 반복적으로 호출했을 때 항상 같은 결과가 반환된다.
    - 순수 함수형 프로그래밍 - '함수 그리고 if-then-else 등의 수학적 표현만 사용'하는 방식
    - 함수형 프로그래밍 - 시스템의 다른 부분에 영향을 미치지 않는다면 내부적으로는 함수형이 아닌 기능도 사용하는 방식

## 18. 2. 1 함수형 자바
- 자바에서는 순수 함수형이 아닌 함수형 프로그래밍을 구현
- 함수나 메서드는 지역변수만을 변경해야 하며 참조하는 객체가 있다면 불변객체여야 한다.
- 메서드 내에서 생성한 객체의 필드는 갱신할 수 있지만, 외부에 노출되어서는 안된다.
- 또한 함수나 메서드가 어떤 예외도 일으키지 않아야 한다. 예외가 발생하면 return으로 결과를 반환할 수 없기 때문이다.
- 어떤 수를 0으로 나눈다든가 하는 함수의 경우 예외가 필요할 수 있다. 이럴 경우 Optional을 사용하면 예외 없이도 연산을 성공적으로 수행했는지 확인할 수 있다.
- 그리고 비함수형 동작을 감출 수 있는 상황에서만 부작용을 포함하는 라이브러리 함수를 사용해야 한다.

# 18. 2. 2 참조 투명성
- '부작용을 감춰야한다'라는 제약은 참조 투명성 개념으로 귀결된다.
- 같은 인수로 함수를 호출했을 때 항상 같은 결과를 반환한다면 참조적으로 투명한 함수라고 표현한다.
- List를 반환하는 메서드를 두 번 호출한다고 가정하자.
- 결과 리스트가 가변 객체라면 반환된 두 리스트는 같은 객체라 할 수 없다. 따라서 이 메서드는 참조적으로 투명한 메서드가 아니다.
- 하지만 결과 리스트를 불변 값으로 사용할 것이라면 두 리스트는 같은 객체라 볼 수 있으므로 참조적으로 투명한 것으로 간주할 수 있다.
- 참조 투명성은 함수가 **함수 외부의 영향을 받지 않는 것**을 의미한다.


# ETC

## 프로그래밍에서의 참조 투명성

```java
int add(int a, int b) { return a + b } 
int mult(int a, int b) { return a * b; } 

int x = add(2, mult(3, 4));
```

- mult(3, 4) : 순수함수의 실행부분을 실행결과 값인 12로 대체해도 프로그램에 전혀 문제가 없으므로 mult(3, 4)는 참조 투명하다.

## 참조 투명하지 않은 예

```java
int add(int a, int b) { 
	int result = a + b; 
	System.out.println("Returning " + result); 
	return result; 
} 

var result = add(3, 4);
```

- add(3, 4); 부분을 12로 바꾸게 되면 로그가 찍히지 않기 떄문에 프로그램의 결과가 달라진다.
- 따라서 위의 add 함수는 참조 투명하지 않다.
