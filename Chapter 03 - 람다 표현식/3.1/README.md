## 람다

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
