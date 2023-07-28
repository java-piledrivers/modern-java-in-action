# 21.4 자바의 미래
자바의 기능으로 채택되지 못한 기능들을 살펴보자

## 21.4.1 선언 사이트 변종
자바에서는 제네릭 서브 형식을 와일드카드로 지정할 수 있는 유연성(사용 사이트 변종)을 허용한다.
```java
List<? extends Number> numbers = new ArrayList<Integer>();
```

하지만 다음처럼 ? extends가 없으면 컴파일 에러가 발생한다.
```java
List<Number> numbers = new ArrayList<Integer>(); // 컴파일 에러
```
선언 사이트 변종에서는 ? extends나 ? super를 사용할 필요가 없지만 자바는 아직 적용되지 않았다.


## 21.4.2 패턴 매칭
```java
# if문에서 타입을 비교했음에도, 타입을 캐스팅 하며 사용해야한다.
if(op instanceof BinOp) {
    BinOp binOp = ((BinOp) op).getLeft();
}

```

만약 패턴 매칭이 도입 됐다면?
```java
if(op instanceof BinOp binOp) {
  BinOp binOp = op.getLeft(); 
}
```
타입캐스팅하지 않고 바로 사용할 수 있다. 하지만 아직 자바에는 적용되지 않았다.


## 21.4.3 풍부한 형식의 제네릭
### 구체화된 제네릭
자바는 타입 소거(Type Erasure)를 사용하여 제네릭을 구현한다.    
이는 컴파일 시점에만 제네릭 타입 정보를 사용하고, 런타임 시에는 해당 정보가 삭제되는 것을 말한다.    
따라서 자바의 제네릭은 컴파일 시점에 타입 안정성을 제공하지만, 런타임에는 제네릭 타입 정보를 알 수 없다.

```java
List<String> strings = new ArrayList<String>();
List<Integer> integers = new ArrayList<Integer>();
// 컴파일 시에는 List<String>과 List<Integer>이지만, 런타임에는 둘 다 List로 보인다.
```

만약 자바가 List<int>를 지원한다고 생각해보자. 문제는 없어보인다.       
List<int>에서는 기본형 42를 얻을 수 있고, List<String>에서는 문자열 객체를 얻을 수 있다면 걱정할 필요가 없지 않을까?      
하지만 가비지 컬렉션(Garbage Collection)을 사용하는 언어에서는 기본형과 객체를 구분하는 것이 중요하다.   
List<int>에 42를 추가하면, 값인 42는 가비지 컬렉션의 대상이 될 필요없이 메서드 호출이 종료되면 사라지지만, 참조 타입인 List<String>에 문자열을 추가하면, 가비지 컬렉터에 의해 문자열이 제거될 때까지 메모리에 남아있다.   


### 제네릭이 함수 형식에 제공하는 문법적 유연성
많은 함수형 언어와 자바를 비교해보자
```
// 함수형 언어
(Integer, Double) => String
// 자바
BiFunction<Integer, Double, String>

// 함수형 언어
Integer => String
// 자바
Function<Integer, String>

// 함수형 언어
() => String
// 자바
Supplier<String>
```
자바가 비교적 덜 유연하다


### 기본형 특화와 제네릭
자바에서는 기본형과 참조형을 구분한다. 기본형은 int, double, boolean 등이 있고, 참조형은 Integer, Double, Boolean 등이 있다.   
이 부분 때문에 더 혼란스러워진 부분이 있다.   
자바에서는 왜 `Function<Apple, Boolean>`이 아닌 `Predicate<Apple>`를 사용할까?       
바로 Predicate가 기본형인 boolean을 반환하기 때문이다. Boolean은 객체이므로 메모리 사용량이 더 크고 언박싱 과정이 필요하다.   


## 21.4.4 더 근본적인 불변성 지원
final은 기본형 값이 바뀌는 것을 막을 수 있지만 객체 참조에서는 효과가 없다.
```java
final int[] arr = {1,2,3};
final List<T> list = new ArrayList<>();
```
첫 번째 행에서 arr[1] = 2 처럼 요소를 바꿀 수 있다.   
두 번째 행에서 list에 다른 값을 할당할 수 없지만 다른 메서드에서 list의 요소 수를 바꿀 수 있다. (list.add() 등)


## 21.4.5 값 형식
객체지향 프로그래밍에 객체가 필요한 것처럼 함수형 프로그래밍에는 값이 필요하다.   


### 컴파일러가 Integer와 int를 같은 값으로 취급할 수는 없을까?
```java
Integer a = 42;
int b = 42;
```
Integer와 int는 같은 값으로 취급할 수 없다.   
기본 형에서는 비트를 비교해서 같음을 판단하지만, 객체에서는 참조로 같음을 판단한다.   
따라서 Integer와 int를 같은 값으로 취급하려면 Integer를 int로 변환해야 한다.


### 변수형: 모든 것을 기본형이나 객체형으로 양분하지 않는다.
다음과 같은 자바의 가정을 조금 바꾸면 문제를 해결할 수 있을 것이다.
1. 기본형이 아닌 모든 것은 객체형이므로 Object를 상속받는다.
2. 모든 참조는 객체 참조다.

위 가정을 좀 더 자세히 살펴보자. 값에는 두 가지 형식이 존재한다.
- 객체형은 (final로 정의되지 않았을 때) 변경할 수 있는 필드를 포함하며 ==로 값이 같음을 검사할 수 있다.
- 값 형식은 불변이며 참조 식별자를 포함하지 않는다. 기본형 값은 넓은 의미에서 값 형식의 일종이다.


### 박싱, 제네릭, 값 형식: 상호 의존 문제
자바에 값 형식이 지원 된다면 기본형도 일종의 값 형식이 될 것이며 현재의 제네릭은 사라질 것이다.      
(참조로 같음을 구별하는) 객체형을 삭제하는 것은 쉽지 않으므로 Integer처럼 기본형 int를 (박싱한) 객체 버전은 계속 컬렉션과 자바 제네릭에 사용될 것이다.      
