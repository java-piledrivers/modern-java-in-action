# 람다 활용 : 실행 어라운드 패턴

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
