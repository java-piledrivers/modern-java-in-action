# 14.8 모듈 정의와 구문들
- module 지시어를 이용해 모듈 정의.
```java
module com.iteratrlearning.application {
}
```

## 14.8.1 requires
- 컴파일 타임과 런타임에, 한 모듈이 다른 모듈에 의존함을 정의한다.
- com.iteratrlearning.application은 com.iteratrlearning.ui 모듈에 의존한다.
- 모듈명을 인수로 받는다.
```java
module com.iteratrlearning.application {
    requires com.iteratrlearning.ui;
}
```

## 14.8.2 exports
- 다른 모듈에서 접근할 수 있는 공개 형식(public) 패키지들을 지정한다.
- 아무 패키지도 공개하지 않는 것이 기본값이며, 어떤 패키지를 공개할지 명시적으로 지정해 캡슐화 수준을 높인다.
- 패키지명을 인수로 받는다.
```java
module com.iteratrlearning.ui {
    requires com.iteratrlearning.core;
    
    exports com.iteratrlearning.ui.panels;
    exports com.iteratrlearning.ui.widgets;
}
```

## 14.8.3 requires transitive
- requires 대신 requires transitive로 의존성 선언 시, 이 모듈을 의존하는 다른 모듈들도 이 키워드로 선언한 모듈들을 읽을 수 있다.
```java
module com.iteratrlearning.ui {
    requires transitive com.iteratrlearning.core;
    
    exports com.iteratrlearning.ui.panels;
    exports com.ireratrlearning.ui.widgets;
}

module com.iteratrlearning.application {
    requires com.iteratrlearning.ui;
}
```

## 14.8.4 exports to
- exports 대신 exports to 사용 시, 정확히 어떤 모듈이 패키지에 접근할 수 있는지 제한할 수 있다.
- 접근할 수 있는 모듈들은 콤마로 구분된 리스트로 작성할 수 있다.
```java
module com.iteratrlearning.ui {
    requires com.iteratrlearning.core;
    exports com.iteratrlearning.ui.panels;
    
    exports com.iteratrlearning.ui.widgets to   //to 뒤에 선언된 모듈들이 접근할 패키지
            com.iteratrlearning.ui2,            //to 앞에 선언된 패키지에 접근 가능한 모듈들
            com.iteratrlearning.ui3,
            com.iteratrlearning.ui4;
}
```

## 14.8.5 open과 opens
- 자바 9 이전에는 리플렉션을 사용하여 패키지의 모든 종류와 멤버들(private 포함)에 접근하는 것이 가능했다. -> 실질적으로 캡슐화는 존재하지 않았다.
- 자바 9부터는 외부 모듈들이 해당 모듈 내 클래스에 대한 리플랙션을 사용할 수 있는지의 여부를 명시적으로 표현해야 한다. 
```java
open module com.iteratrlearning.ui {
}
```
- 만약 해당 모듈 내 특정 패키지에 대해서만 리플렉션을 허용하고 싶을 경우, opens 구문을 사용한다.
```java
module com.iteratrlearning.ui {
    opens com.iteratrlearning.ui.panels;
}
```
- 특정 외부 모듈만 접근 허용을 원할 시 open to 구문을 사용한다.
```java
module com.iteratrlearning.ui {
    opens com.iteratrlearning.ui.panels to 
          module1, 
          module2;
}
```

## 14.8.6 uses와 provides
- 자세한 내용은 The Java Module System 참조
### uses
- 해당 모듈에서 사용하는 service를 지정한다.
- service란 인터페이스를 구현한 클래스, 혹은 추상클래스를 확장한 클래스의 객체(구현체)를 말한다.
- uses 키워드로 선언하는 인터페이스나 추상 클래스들은 해당 모듈 안에 포함돼있다.
```java
module com.iteratrlearning.ui {
    uses classpath.interfacename;
    uses classpath.abstractclassname;
}
```
### provides with
- provides에는 인터페이스나 추상클래스를 선언하고, with에는 이 인터페이스의 구현체를 선언한다.
- provides로 선언하는 인터페이스나 추상클래스들은 외부 모듈에 있다.
```java
module com.iteratrlearning.ui {
    provides classpath.interfacename with implementation;
    provides classpath.abstractclassname with extendedclassname;
}
```

# 14.9 더 큰 예제 그리고 더 배울 수 있는 방법
```java
module com.example.foo {
    requires com.example.foo.http;
    requires java.logging;
    
    requires transitive com.example.foo.network;
    
    exports com.example.foo.quux;
    exports com.example.foo.internal to com.example.foo.probe;
    
    opens com.example.foo.quux;
    opens com.example.foo.internal to com.example.foo.network, 
                                      com.example.foo.probe;
    
    uses com.example.foo.spi.Intf;
    provides com.example.foo.spi.Intf with com.example.foo.Impl;
}
```

<br><br>
#### 리플렉션이란
https://umanking.github.io/2019/08/24/java-reflection/
<br>
#### 패키지와 모듈의 차이
https://raspberrylounge.medium.com/%EC%9E%90%EB%B0%94%EC%97%90%EC%84%9C-%ED%8C%A8%ED%82%A4%EC%A7%80-package-%EC%99%80-%EB%AA%A8%EB%93%88-module-%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90-16b2eda177b4
