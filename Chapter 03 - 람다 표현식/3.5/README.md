 ## 3.5 형식 검사, 형식 추론, 제약

람다 표현식을 처음 설명할 때 람다로 함수형 인터페이스의 인스턴스를 만들 수 있다.
람다 표현식 자체에서 람다가 어떤 함수형 인터페이스를 구현하는지의 정보가 포함되어 있지 않으므로 람다 표현식을 제대로 이해하기 위해서는 <u>람다의 실제 형식</u>을 파악해야 한다.

### 3.5.1 형식 검사

람다가 사용되는 context를 이용해서 람다의 형식을 추론할 수 있다.
**대상 형식** : 람다가 전달될 메서드 파라미터나 람다가 할당되는 변수 (ex. `Predicate<T>` )
예를 들어,
filter() 는`Stream<T> filter(Predicate<? super T> predicate);` 이고, 따라서 `Predicate<T>`라는 대상 형식에 만족하는 람다 함수를 기대한다. 

**예제**

```java
List<Apple> heavierThan150g = filter(inventory, (Apple apple) -> apple.getWeight() > 150);
```

위 예제에서 컴파일러는 아래와 같은 순서로 형식 확인 과정을 거친다.

> (1) filter 메서드의 선언을 확인하고 정의를 확인한다.
> (2) filter 메서드는 두번째 파라미터로 Predicate<Apple> 형식(대상 형식)을 기대한다. (T는 Apple 로 대치됨)
> (3) Predicate은 test라는 한개의 추상 메서드를 정의하는 함수형 인터페이스다. (test()는 boolean을 반환한다)
> (4) test 메서드는 Apple를 받아 boolean을 반환하는 함수 디스크립터를 묘사한다.
> (5) filter 메서드로 전달된 인수는 Apple -> boolean을 만족한다.
> (6) 형식 검사가 성공적으로 종료된다.

### 3.5.2 같은 람다, 다른 함수형 인터페이스

대상 형식 이라는 특징 때문에 같은 람다 표현식이더라도 호환되는 추상 메소드를 가진 다른 함수형 인터페이스로 사용될 수 있다.
ex) Callable, PrivilegedAction Interface는 인수를 받지 않고, 제네릭 형식 T를 반환하는 함수를 정의한다.

**예제**

하나의 람다 표현식을 다양한 함수형 인터페이스에 사용할 수 있다.

```java
Comparator<Apple> c1 = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
ToIntBiFunction<Apple, Apple> c2 = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
BiFunction<Apple, Apple, Integer> c3 = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

**다이아몬드 연산자 (Java 7 부터 추가)**

Java 7 부터 추가된 기능
`<>`로 컨텍스트에 따른 제네릭 형식을 추론할 수 있다.

```java
Map<String, List<String>> anagrams = new HashMap<String, List<String>>(); // Java 7 이전
Map<String, List<String>> anagrams = new HashMap<>(); // Java 7 이후
```

**void 호환 규칙**

Lambda body에 표현식 (expression)이 있으면, void를 반환하는 method descriptor와 호환된다.
즉, void를 반환하는 signature의 경우 다른 타입도 받을 수 있다.

```java
Predicate<String> p = s -> list.add(s); // boolean
Consumer<String> c = s -> list.add(s); // void
```

list.add()는 return으로 boolean을 반환하지만 Consumer<T>: **(T) -> void** 에서도 사용할 수 있다.
(하지만 명확한 함수 signature를 사용하는것을 권장한다.)

### 3.5.3 형식 추론

자바 컴파일러는 람다 표현식이 사용된 콘텍스트 (대상 형식)을 이용해서 람다 표현식과 관련된 함수형 인터페이스를 추론한다. 
또한 대상형식을 이용해서 함수 디스크립터를 알 수 있으므로 람다의 시그니처도 추론할 수 있다. 
**결과적으로 컴파일러는 람다 표현식의 파라미터 형식에 접근할 수 있으므로 람다 문법에서 이를 생략할 수 있다.**

```java
List<Apple> greenApples = filter(inventory, apple -> GREEN.equals(apple.getColor())); // Apple 형식 생략

Comparator<Apple> c = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()); // 형식을 추론하고있지않음

Comparator<Apple> c = (a1, a2) -> a1.getWeight().compareTo(a2.getWeight()); // type을 생략하여 형식을 추론
```

위 예제처럼 자바 컴파일러는 람다 파라미터 형식을 추론할 수 있다. 
상황에 따라 명시적으로 형식을 포함할때와 포함하지 않을때를 구분하여 코드의 가독성을 향상시키는 것이 좋다.

### 3.5.4 지역변수의 사용

람다 캡쳐링 : 람다 표현식에서는 익명 함수가 하는 것처럼 자유 변수(파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수)를 활용할 수 있다.

**람다에서는 불변한 지역변수만 사용이 가능하다**. (final로 선언되거나 또는 final로 선언된 변수와 똑같이 사용되어지는 변수)
단 한번만 할당할 수 있는 지역변수를 사용할 수 있다.

```
int portNumber = 3306;
Runnable r = () -> System.out.println(test); // Error
test = 3307;
```

위 예제에서 지역변수 test는 final로 정의되어있지 않으며 값을 3306에서 3307로 변경하는 코드도 존재한다. 
따라서 final 처럼 사용되어지는 변수도 아니기 때문에 람다에서 사용된 test 코드에서 에러가 발생하게 된다.

 

**그렇다면, 왜 람다에서는 지역변수의 사용에 제약이 있을까?**

> 내부적으로 인스턴스 변수와 지역 변수는 다르다. 인스턴스 변수는 힙에 저장되고, 지역 변수는 스택에 저장된다. 
> 람다에서 지역 변수에 바로 접근할 수 있다는 가정을 했을때 람다가 스레드에서 실행된다면 변수를 할당한 스레드가 사라져서 변수 할당이 해제되었는데도 람다를 실행하는 스레드에서는 해당 변수에 접근하려고 할 수 있다. 
> 자바 구현에서는 원래 변수에 접근을 허용하는 것이 아닌 자유 지역 변수의 복사본을 제공한다. 
> 따라서, 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한번만 값을 할당한다라는 제약이 생긴것이다.



**클로저**
클로저란 함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스를 가르킨다. 
클로저는 다른 함수의 인수로 전달 할 수 있다. 
클로저는 클로저 외부에 정의된 변수의 값을 접근하고 값을 바꿀 수 있다. 
자바 8의 람다와 익명 클래스는 클로저와 비슷한 동작을 수행한다. 
람다와 익명 클래스 모두 함수의 인자로 전달 될 수 있다. 자신의 외부 영역에 변수에 접근 할 수 있다.
단, <u>람다와 익명클래스는 람다가 정의된 메서드의 지역 변수의 값을 바꿀 수 없다</u>. 
람다가 정의된 메서드의 지역 변수값은 final 변수여야 한다. 
지역 변수값은 스택에 존재하므로 자신을 정의한 스레드와 생존을 같이 해야 하며 따라서 지역 변수는 final로 되어야 하는 것이다. 
가변 지역 변수를 새로운 스레드에서 캡처할 수 있다면 안전하게 동작하지 않을 것이다. 
