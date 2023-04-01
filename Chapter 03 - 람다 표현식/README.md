# 3.1 람다란 무엇인가?

- 람다 클래식을 어떻게 만드는지 어떻게 사용하는지 어떻게 코드를 간결하게 하는지
- 책 전체에 광범위하게 사용하므로 완벽하게 이해 필요

3.1절에서 람다(lambda)에 대한 기본.    
람다는 Java 8부터 도입된 함수형 프로그래밍 기능 중 하나로, 간결한 코드 작성 가능하며, 코드 가독성과 유지보수 향상 도움이 됨.

### 특징

- 익명성: 람다는 이름 없는 함수임. 코드 간결하게 작성 가능.
- 함수형 인터페이스: 람다는 함수형 인터페이스 구현함. 함수형 인터페이스는 하나의 추상 메소드 가진 인터페이스임.
- 간결성: 람다는 기존 익명 클래스보다 간결한 문법 제공.
- 전달: 람다 표현식을 메서드 인수로 전달하거나 변수로 저장
- 람다 캡처링: 람다는 외부 변수 참조 가능. 참조되는 외부 변수는 final 또는 effective final이어야 함.

#### 람다 캡처링(Lambda Capturing)

람다 시그니처의 파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수를 **자유 변수(Free Variable)**라고 부름.   
람다 바디에서 자유 변수를 참조하는 행위를 **람다 캡처링(Lambda Capturing)**이라고 함.

```java
import java.util.function.Function;

public class LambdaCapturingExample {

    private String instanceVariable = "Instance variable";

    public static void main(String[] args) {
        final String localVar = "final variable";
        String anotherLocalVar = "Effectively final variable";

        // localVar must be final or effectively final
        Function<String, String> lambda1 = s -> {
            // Access local variable from the enclosing scope
            return s + " and " + localVar;
        };

        // anotherLocalVar is not modified and is effectively final
        Function<String, String> lambda2 = s -> {
            // Access effectively final local variable from the enclosing scope
            return s + " and " + anotherLocalVar;
        };

        LambdaCapturingExample example = new LambdaCapturingExample();

        // Access instance variable from the enclosing scope
        Function<String, String> lambda3 = s -> {
            return s + " and " + example.instanceVariable;
        };

        // Access static variable from the enclosing scope
        Function<String, String> lambda4 = s -> {
            return s + " and " + ExampleStaticClass.staticVariable;
        };

        System.out.println(lambda1.apply("Lambda with"));
        System.out.println(lambda2.apply("Lambda with"));
        System.out.println(lambda3.apply("Lambda with"));
        System.out.println(lambda4.apply("Lambda with"));
    }
}

class ExampleStaticClass {
    public static String staticVariable = "Static variable";
}
```

lambda1: enclosing 범위에서 지역 변수 localVar를 캡처. localVar는 final이거나 effectively final (final 처럼 다루어져야함). 이 경우 수정되지 않았으므로 final
lambda2: 또 다른 지역 변수 anotherLocalVar를 캡처하며, 수정되지 않았으므로 effectively final. 만약에 앞에 다른 값이 할당되었다면 컴파일 안됨

람다 바디 안에 local 변수가 포함되어있으면 실행이 끝나고 나면 변수 할당을 해제하는데,   
만약에 localVar 를 해제해버리면 다른 곳에서 사용을 못함. 그래서 람다에서 실행될때 그 변수를 복사해서 사용.   
복사된 변수는 변경을 허용하지 않는다. (컴파일 불가능)

> Q) 마음대로 변경을 하게 되면 문제가 있다는데 왜 문제가 생기는지 잘 모르겠음. 납득이 안됨

lambda3, lambda4: 인스턴스 변수와 static변수는 캡처하는 데에 제한이 없다. Heap 을 참조하기 때문

![2023-03-16_22-38-10](https://user-images.githubusercontent.com/59721293/225644400-3773ec7a-c1c9-4f6d-b5a4-0949a97ebc90.jpg)


### 사용법
- (a, b) -> a + b: 두 인자 받아 더하는 람다 표현식.
- () -> 42: 인자 없이 42 반환하는 람다 표현식.
- s -> s.length(): 문자열 인자 하나 받아 길이 반환하는 람다 표현식.
- 람다 표현식 사용하면 간결하고 가독성 높은 코드 작성 가능하며, Java에서 함수형 프로그래밍 패러다임 활용 가능.

개인적인 생각으론 사실 인텔리제이가 틀리면 알아서 에러를 주기 때문에 사용법은 익히려고 하지 않아도 잘할 수 있다고 생각


# 3.2 어디에, 어떻게 람다를 사용할까?

결론적으로는 함수형 인터페이스라는 문맥에서 람다 표현식을 사용할 수 있다.


## 함수형 인터페이스
**하나의 추상 메서드를 지정하는 인터페이스**이다.
```java
public interface Predicate<T> {
    boolean test(T t);
}

public interface Comparator<T> { //java.util.Comparator
    int compare(T o1, T o2);
}

public interface Runnable { //java.lang.Runnable
    void run();
}
```

참고로 인터페이스의 경우 디폴트 메서드(인터페이스의 메서드를 구현하지 않은 클래스를 고려해서 기본 구현을 제공하는 바디를 포함하는 메서드)를 포함할 수 있는데, 많은 디폴트 메서드가 있더라도 **추상 메서드가 오직 하나면 함수형 인터페이스**다.
```java
public interface Adder { //함수형 인터페이스
    int add(int a, int b);
}

public interface SmartAdder extends Adder { //두 추상 메서드를 포함하므로 함수형 인터페이스 X
    int add(double a, double b);
}

public interface Nothing { //추상 메서드가 없으므로 함수형 인터페이스 X
}
```


## 함수 디스크립터
함수형 인터페이스의 추상 메서드 시그니처(메서드 구조를 정의한 것)는 람다 표현식의 시그니처를 가리킨다. 함수 디스크립터는 **람다 표현식의 시그니처를 서술하는 메서드**를 의미한다.

예를 들어 Runnable 인터페이스의 유일한 추상 메서드인 `run()`은 인수와 반환값이 없으므로 해당 인터페이스는 인수와 반환값이 없는 시그니처로 생각할 수 있다.
```java
() -> void //파라미터 리스트가 없으며 void를 반환하는 함수를 의미
(Apple, Apple) -> int //두 개의 Apple을 인수로 받아 int를 반환하는 함수를 의미
```

람다 표현식의 형식 검사 원리에 대해서는 3.5절에서 자세하게 설명하므로 현재는 람다 표현식은 변수에 할당하거나 함수형 인터페이스를 인수로 받는 메서드로 전달할 수 있으며, 함수형 인터페이스의 추상 메서드와 같은 시그니처를 갖는다는 사실을 기억해 두자.

또한 '왜 함수형 인터페이스를 인수로 받는 메서드에만 람다 표현식을 사용할 수 있을까?'라는 의문이 생길 수 있다.
그 이유는 아래와 같다.
- 언어 설계자들은 자바에 함수 형식(람다 표현식을 표현하는 데 사용한 시그니처와 같은 특별한 표기법)을 추가하는 방법도 대안으로 고려했으나, 언어를 더 복잡하게 만들지 않는 현재 방법을 선택
- 대부분의 자바 프로그래머가 이벤트 처리 인터페이스와 같이 하나의 추상 메서드를 갖는 인터페이스에 이미 익숙하다는 점도 고려

# 3.3 람다 활용 : 실행 어라운드 패턴

자원 처리(예: 데이터베이스)에 사용하는 순환 패턴(recurrent pattern)은 자원을 열고, 처리한 다음에, 자원을 닫는 순서로 이루어진다.

```
A 작업: 초기화/준비 코드 -> 작업 A -> 정리/마무리 코드
B 작업: 초기화/준비 코드 -> 작업 B -> 정리/마무리 코드
```

- 설정(setup)과 정리(cleanup) 과정은 대부분 비슷하다. 실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태를 갖는다.
- 이와 같은 형식의 코드를 실행 어라운드 패턴(execute around pattern)이라고 부른다.


다음 예제는 파일에서 한 행을 읽는 코드다. try-with-resources 구문을 사용했다. 이를 사용하면 명시적으로 닫아줄 필요가 없으므로 간결한 코드를 구현하는데 도움을 준다.
```java
public String processFile() throws IOException {
  try (BufferedReader br = new BufferReader(new FileReader("data.txt"))) {
    return br.readline(); //파일에서 한 행을 읽는 코드
  }
}
```

## 1단계 : 동작 파라미터화를 기억하라
현재 코드는 한 번에 한 파일에서 한 줄만 읽을 수 있다. 이후 요구사항이 변경되어 한 번에 두 줄을 읽거나, 가장 자주 사용되는 단어를 리턴하려면 어떻게 해야 할까?

바로 processFile 메서드의 동작을 파라미터화하면 해결된다. processFile 메서드가 BufferedReader를 이용해서 다른 동작을 수행할 수 있도록 processFile 메서드로 동작을 전달해야 한다.

다음은 BufferedReader에서 두 행을 출력하는 코드이다.

```java
String result = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

## 2단계 : 함수형 인터페이스를 이용해서 동작전달
BufferedReader -> String 과 IOException을 던질 수 있는 시그니처와 일치하는 함수형 인터페이스를 만들어보자.

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}
```


정의한 인터페이스를 processFile 메서드의 인수로 전달할 수 있다.
```java
public String processFile(BufferedReaderProcessor p) throws IOException {
    ...
}
```

## 3단계 : 동작 실행
람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으며 전달된 코드는 함수형 인터페이스의 인스턴스로 전달된 코드와 같은 방식으로 처리한다.   
따라서 processFile 바디 내에서 BufferedReaderProcessor 객체의 process를 호출할 수 있다.
```java
public String processFile(BufferedReaderProcessor p) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
    return p.process(br); //BufferedReader 객체 처리
    }
}
```

## 4단계 : 람다 전달
이제 람다를 이용하여 다양한 동작을 processFile 메서드로 전달할 수 있다.
```java
//한 행을 처리하는 코드
String oneLine = processFile((BufferedReader br) -> br.readLine());

//두 행을 처리하는 코드
String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

# 3.4

함수형 인터페이스 : 오직 하나의 추상 메서드를 가진다.

함수형 인터페이스의 추상메서드 : 함수형 디스크립터

함수형 인터페이스의 추상메서드는 람다표현식의 시그니처를 묘사(메서드 명과 파라미터의 순서, 타입, 개수)


다양한 함수형 디스크립터

1. Predicate

 - T라는 객체를 받아서 test라는 메서드로 Boolean 값을 리턴한다.

@FunctionalInterface
public interface Predicate<T>{
    boolean test(T t);
}

public <T> List<T> filter(List<T> list, Predicate<T> p){
    List<T> results = new ArrayList<>();
    for(T t : list){
        if(p.test(t)){
            results.add(t);
        }
    }
    return results;
}
Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate)


2.Consumer
  
- T라는 객체를 받아서 accept라는 메서드로 void를 반환한다.
  
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}

public void <T> void forEach(List<T> list, Consumer<T> c) {
    for (T t : list) {
        c.accept(t);
    }
}

forEach(
  Arrays.asList(1,2,3,4,5),
  (Integer i) -> System.out.println(i); \
);


3.Function
  
-T라는 객체를 받아서 apply 라는 메서드로 제네릭 형식 R 객체를 반환한다.
  
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}

public <T, R> List<R> map(List<T> list, Function<T, R> f) {
    List<R> result = new ArrayList<>();
    for (T t : list) {
        result.add(f.apply(t));
    }
    return result;
}

List<Integer> l = map(
    Arrays.asList("lambdas", "in", "action"),
    (String s) -> s.length(); 
);

박싱

- 기본형 -> 참조형

언박싱

-참조형 -> 기본형

오토박
  싱
-박싱와 언박싱이 자유롭게 수행

이러한 변환과정을 비용이 소모 

자바 8에서는 기본형을 입출력으로 사용하는 상황에서 오토박싱을 피할수 있도록 함수형 인터페이스 제공

# 3.5 형식 검사, 형식 추론, 제약

람다로 함수형 인터페이스의 인스턴스를 만들 수 있다.<br>
람다 표현식 자체에서 람다가 어떤 함수형 인터페이스를 구현하는지의 정보가 포함되어 있지 않으므로 람다 표현식을 제대로 이해하기 위해서는 <u>람다의 실제 형식</u>을 파악해야 한다.

### 3.5.1 형식 검사

람다가 사용되는 내용(Context)을 바탕으로 람다의 형식(Type)을 추론할 수 있다.<br>
**대상 형식** : 람다가 전달될 메서드 파라미터나 람다가 할당되는 변수 등에서 기대되는 람다의 형식 (ex. `Predicate<T>` )<br>
예를 들어,<br>
filter() 는`Stream<T> filter(Predicate<? super T> predicate);` 이고, 따라서 `Predicate<T>`라는 대상 형식에 만족하는 람다 함수를 기대한다. 

**예제**

```java
List<Apple> heavierThan150g = filter(inventory, (Apple apple) -> apple.getWeight() > 150);
```

위 예제에서 컴파일러는 아래와 같은 순서로 형식 확인 과정을 거친다.

> (1) filter 메서드의 선언을 확인하고 정의를 확인한다.<br>
> (2) filter 메서드는 두번째 파라미터로 Predicate<Apple> 형식(대상 형식)을 기대한다. (T는 Apple 로 대치됨)<br>
> (3) Predicate은 test라는 한개의 추상 메서드를 정의하는 함수형 인터페이스다. (test()는 boolean을 반환한다)<br>
> (4) test 메서드는 Apple를 받아 boolean을 반환하는 함수 디스크립터를 묘사한다.<br>
> (5) filter 메서드로 전달된 인수는 Apple -> boolean을 만족한다.<br>
> (6) 형식 검사가 성공적으로 종료된다.<br>

### 3.5.2 같은 람다, 다른 함수형 인터페이스

대상 형식 이라는 특징 때문에 같은 람다 표현식이더라도 호환되는 추상 메소드를 가진 다른 함수형 인터페이스로 사용될 수 있다.<br>
ex) Callable, PrivilegedAction Interface는 인수를 받지 않고, 제네릭 형식 T를 반환하는 함수를 정의한다.<br>

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

Lambda body에 표현식 (expression)이 있으면, void를 반환하는 method descriptor와 호환된다.<br>
즉, void를 반환하는 signature의 경우 다른 타입도 받을 수 있다.

```java
Predicate<String> p = s -> list.add(s); // return boolean
Consumer<String> c = s -> list.add(s); // return void
```

list.add()는 return으로 boolean을 반환하지만 Consumer<T>: **(T) -> void** 에서도 사용할 수 있다.<br>
(하지만 명확한 함수 signature를 사용하는것을 권장한다.)

### 3.5.3 형식 추론

자바 컴파일러는 람다 표현식이 사용된 콘텍스트 (대상 형식)을 이용해서 람다 표현식과 관련된 함수형 인터페이스를 추론한다. <br>
또한 대상형식을 이용해서 함수 디스크립터를 알 수 있으므로 람다의 시그니처도 추론할 수 있다. <br>
**결과적으로 컴파일러는 람다 표현식의 파라미터 형식에 접근할 수 있으므로 람다 문법에서 이를 생략할 수 있다.**

```java
List<Apple> greenApples = filter(inventory, apple -> GREEN.equals(apple.getColor())); // Apple 형식 생략

Comparator<Apple> c = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()); // 형식을 추론하고있지않음

Comparator<Apple> c = (a1, a2) -> a1.getWeight().compareTo(a2.getWeight()); // type을 생략하여 형식을 추론
```

위 예제처럼 자바 컴파일러는 람다 파라미터 형식을 추론할 수 있다. <br>
상황에 따라 명시적으로 형식을 포함할때와 포함하지 않을때를 구분하여 코드의 가독성을 향상시키는 것이 좋다.<br>

### 3.5.4 지역변수의 사용

람다 캡쳐링 : 람다 표현식에서는 익명 함수가 하는 것처럼 자유 변수(파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수)를 활용할 수 있다.

```
int portNumber = 3306;
Runnable r = () -> System.out.println(test); // Error
test = 3307;
```

**람다에서는 불변한 지역변수만 사용이 가능하다**. (final로 선언되거나 또는 final로 선언된 변수와 똑같이 사용되어지는 변수)<br>
단 한번만 할당할 수 있는 지역변수를 사용할 수 있다.<br>
 
위 예제에서 지역변수 test는 final로 정의되어있지 않으며 값을 3306에서 3307로 변경하는 코드도 존재한다. <br>
따라서 final 처럼 사용되어지는 변수도 아니기 때문에 람다에서 사용된 test 코드에서 에러가 발생하게 된다.<br>

 

**그렇다면, 왜 람다에서는 지역변수의 사용에 제약이 있을까?**

내부적으로 인스턴스 변수와 지역 변수는 다르다. <br>
- 인스턴스 변수 : Heap 
- 지역 변수 : Stack <br> 

람다에서 지역 변수에 바로 접근할 수 있다는 가정을 했을 때 람다가 스레드에서 실행된다면 변수를 할당한 스레드가 사라져서 변수 할당이 해제되었는데도 람다를 실행하는 스레드에서는 해당 변수에 접근하려고 할 수 있다.<br> 
자바 구현에서는 원래 변수에 접근을 허용하는 것이 아닌 자유 지역 변수의 복사본을 제공한다.<br>
따라서, 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한번만 값을 할당한다라는 제약이 생긴것이다.<br>



**클로저**<br>
람다는 클로저의 범위에 부합할까?<br>
클로저란 함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스를 가르킨다. <br>
클로저는 다른 함수의 인수로 전달 할 수 있다. <br>
클로저는 클로저 외부에 정의된 변수의 값을 접근하고 값을 바꿀 수 있는데, <br>
자바 8의 람다와 익명 클래스는 클로저와 비슷한 동작을 수행한다. <br>
람다와 익명 클래스 모두 함수의 인자로 전달 될 수 있고 자신의 외부 영역에 변수에 접근 할 수 있다.<br>
단, <u>람다와 익명클래스는 람다가 정의된 메서드의 지역 변수의 값을 바꿀 수 없다</u>. <br>
람다가 정의된 메서드의 지역 변수값은 final 변수여야 한다. <br>
지역 변수값은 스택에 존재하므로 자신을 정의한 스레드와 생존을 같이 해야 하며 따라서 지역 변수는 final로 되어야 하는 것이다. <br>
가변 지역 변수를 새로운 스레드에서 캡처할 수 있다면 안전하게 동작하지 않을 가능성이 생긴다. <br>

# 3.6 메서드 참조

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

```java
public interface TriFunction<T, U, V, R> {
  R apply (T t, U u, V v);
}

TriFunction<Integer, Integer, Integer, Color> colorFactory = Color::new;
```


# 3.7 람다, 메서드 참조 활용하기

리스트를 다양한 정렬 기법으로 정렬하는 문제를, 더욱 간결하게 해결하는 방법을 정리한다.
2장부터 3장까지 다룬 동작 파라미터화, 익명 클래스, 람다 표현식, 메서드 참조를 통해 최종적으로 아래와 같은 코드가 만들어진다.

inventory.sort(comparing(Apple::getWeight));


해당 장의 예제에서는 List API의 sort 메서드를 사용한다.
sort 메서드는 아래와 같은 시그니처를 가지고 있다.

void sort(Comparator<? super E> c) ->오버로딩으로 여러개 존재하는 sort 메서드중 하나

Comparator 객체를 인수로 받아 두 사과를 비교하는데, Comparator를 어떻게 구현하는 방법에 따라 다양하게 정렬를 할 수 있다.

즉 2.2 에서 다룬 동작 파라미터화를 구현할 수 있다.


1. 동작 파라미터화(코드 전달) 예제 코드

```java


public class AppleComparator implements Comparator<Apple> {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
}

public class AppleComparatorReverse implements Comparator<Apple> {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight()) * -1 ;
    }
}

...
//1단계 동작 파라미터화를 통해 런타임에서도 원하는 정렬 전략을 선택할 수 있다.[2.2]
inventory.sort(new AppleComparator());
//정렬 전략을 변경하여 역순으로 정렬한다.
inventory.sort(new AppleComparatorReverse());
```

만약 Comparator가 한번만 사용된다면, 코드를 구현하는 것보단 익명 클래스를 이용하여 별도의 클래스 선언 없이 코드를 생성할 수 있다.

```java

//2단계: 익명 클래스를 통해 별도의 클래스를 만들 필요없이 정렬 전략을 전달한다.
inventory.sort(new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
}
```


3장에서 다룬 것처럼 이러한 익명클래스 역시 더욱 경량화된 문법인 람다 표현식을 사용하여 대체할 수 있다.

이미 이번장에서 다룬 것처럼 
함수형 인터페이스(functional interface)는 하나의 추상 메서드(abstract method)만을 가지는 인터페이스다. 

람다 표현식(lambda expression)은 이 함수형 인터페이스의 구현을 쉽게 할 수 있게 해주는 문법이다. 

람다 표현식을 사용하면 특정 동작을 파라미터로 전달하는 코드를 위처럼 간결하게 작성할 수 있다.

Comparator 인터페이스는 하나의 추상 메서드 compare만을 가지는 함수형 인터페이스며, 반환값이 int이다.

int compare(T o1, T o2)

함수 디스크립터는 (T, T) -> int 이며, 현재 우리의 코드는 (Apple, Apple) -> int 로 표현할 수 있다.


```java
//3단계: 람다 표현식 사용
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

//형식 추론을 사용한 코드
//자바 컴파일러는 람다 표현식이 사용된 콘텍스트를 활용해서 람다의 파라미터 형식을 추론한다.[3.5]
inventory.sort((a1, a2) -> a1.getWeight().compareTo(a2.getWeight());
```

*함수 디스크립터는 함수형 인터페이스의 추상 메서드의 입력과 출력을 간략하게 표현하는 것이다.


이렇듯 람다 표현식을 사용하면, 코드를 간략하게 작성할 수 있음을 확인했다.
하지만 메서드 참조[3.6]를 사용하면 코드를 더욱 간략하게 줄일 수 있다.
 
```java
//4단계: 메서드 참조
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

```
바로 앞장에서 다룬 것처럼 메서드 참조를 사용하려면 
java.util.Comparator.comparing을 정적으로 임포트하고, comparing 메서드를 사용해야한다.



# 3.8 람다 표현식을 조합할 수 있는 유용한 메서드
자바 8 API의 `Comparator`, `Function`, `Predicate` 와 같은 함수형 인터페이스는 람다 표현식을 조합할 수 있는 메서드를 제공한다.
람다 표현식을 조합할 수 있다는 것은 여러 개의 프레디케이트를 조합하여 하나의 큰 프레디케이트로 만들 수 있다는 것이다.   
[함수형 인터페이스](https://github.com/java-piledrivers/modern-java-in-action/tree/main/Chapter%2003%20-%20%EB%9E%8C%EB%8B%A4%20%ED%91%9C%ED%98%84%EC%8B%9D/3.2#%ED%95%A8%EC%88%98%ED%98%95-%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4)의
정의는 오직 하나의 추상 메서드를 지정하는 인터페이스인데 추가로 여러 메서드를 제공한다면 함수형 인터페이스의 정의에 어긋나다고 생각할 수 있지만 여기서 제공하는 
메서드는 **디폴트 메서드**(default method)임으로 정의의 어긋나지 않는다. 

<br>


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

<br>


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


<br>


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

# 3.9 비슷한 수학적 개념

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


