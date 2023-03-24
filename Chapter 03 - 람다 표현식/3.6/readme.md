# 메서드 참조

- 명시적으로 메서드명을 참조함으로써 가독성을 높일 수 있다.

메서드 참조는 특정 메서드만을 호출하는 람다의 축약형이라고 할 수 있으며, 메서드명 앞에 구분자(::)를 붙이는 방식으로 활용할 수 있다.

예를들어 람다 표현식 (Apple a) -> a.getWeight를 축약하여 Apple::getWeight로 Apple 클래스에 정의된 getWeight의 메서드 참조를 사용할 수 있다.

- 예제

```java
(Apple apple) -> apple.getWeight()
Apple::getWeight
```

```java
() -> Thread.currentThread().dumpStack()
Thread.currentThread()::dumpStack
```

```java
(Str, i) -> str.substring(i)
String::substring
```

```java
(String s) -> System.out.println(s)
System.out::println
```

```java
(String s) -> this.isValidName(s)
this::isValidaName
```

### 메서드 참조를 만드는 방법

1. 정적 메서드 참조 : Integer::parseInter
2. 다양한 형식의 인스턴스 메서드 참조 : String::length
3. 기존 객체의 인스턴스 메서드 참조
- Transaction 객체를 할당받은 expensiveTransaction 지역변수가 있고, 객체에 getValue 메서드가 있다면
- expensiveTransaction::getValue

### **3.6.2 생성자 참조**

ClassName::new 처럼 클래스명과 new 키워드를 이용해서 기존  생성자의 참조를 만들 수 있다.

```java
Supplier<Apple> c1 = () -> new Apple();
Supplier<Apple> c2 = Apple::new;

Apple a1 = c1.get();
Apple a2 = c2.get();
```

Apple(Integer weight) 라는 시그니처를 갖는 생성자는 Function 인터페이스와 시그니처가 같다. 따라서 다음과 같은 코드를 구현할 수 있다.

```java
Function<Integer, Apple> c3 = ( weight) -> new Apple(weight);
Function<Integer, Apple> c4 = Apple::new;

Apple a3 = c3.apply(110);
Apple a4 = c4.apply(110);
```

Apple(String color, Integer weight) 처럼 두 인수를 갖는 생성자는 Bifunction 인터페이스와 같은 시그니처를 가지므로 다음처럼 할 수 있다.

```java
BiFunction<Color, Integer, Apple> c5 = (color, weight) -> new Apple(color, weight);
BiFunction<Color, Integer, Apple> c6 = Apple::new;

Apple a5 = c5.apply(GREEN, 110);
Apple a6 = c6.apply(GREEN, 110);
```

Color(int, int, int) 처럼 인수가 세 개인 생성자를 사용하려면 직접 함수형 인터페이스를 생성해야 한다.
