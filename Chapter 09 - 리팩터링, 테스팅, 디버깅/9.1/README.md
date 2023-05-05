## 9.1 가독성과 유연성을 개선하는 리팩터링
람다, 메서드 참조, 스트림 등의 기능을 이용해 기존 코드를 가독성과 유연성이 높도록 리팩터링.
 <br> <br>
 
### 9.1.1 코드 가독성 개선
코드 가독성 - 내가 구현한 코드를 다른 사람이 쉽게 이해할 수 있는 정도. <br>
가독성을 높이려면 코드의 문서화 및 표준 코딩 규칙을 준수해야 한다.
 <br> <br>
 
### 9.1.2 익명 클래스를 람다 표현식으로 리팩터링하기
익명 클래스는 코드가 길고 쉽게 에러를 일으키게 만듦. <br>
보다 간결하고 가독성이 좋은 람다 표현식으로 리팩터링. <br> <br>

**모든 익명 클래스를 람다 표현식으로 변환할 수 있는 것은 아니다.**
1. 익명 클래스에서 this는 익명 클래스 자신을 가리키지만 람다에서 this는 람다를 감싸는 클래스를 가리킨다.
2. 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있지만 (섀도잉; 바깥 범위의 변수와 동일한 이름으로 변수 선언),
<br>람다 표현식으로는 가릴 수 없다.
3. 익명 클래스를 람다 표현식으로 바꾸면 콘텍스트 오버로딩에 따른 모호함이 초래될 수 있다.<br>
익명 클래스는 인스턴스화 할때 명시적으로 형식이 정해지는 반면 람다의 형식은 콘텍스트에 따라 달라지기 때문이다.<br>
명시적 형변환으로 타입을 명시해 해결 가능하다.

```java
//2
int a = 10;

Runnable r2 = new Runnable(){
    public void run(){
        int a = 2; //정상 동작
        System.out.println(a);
    }
};

Runnable r1 = () -> {
    //int a = 2; //컴파일 에러 - 섀도잉 불가
    System.out.println(a);
};


//3
interface Task {
    void process();

    static void subProcess(Runnable runnable) {
        runnable.run();
    }

    static void subProcess(Task task) {
        task.process();
    }
}

public class Example {
    public static void main(String[] args) {
        //Task.subProcess(() -> System.out.println("##")); //컴파일 에러
        Task.subProcess((Task)() -> System.out.println("##"));
    }
}
```
<br>
<br>

### 9.1.3 람다 표현식을 메서드 참조로 리팩터링하기
람다 표현식은 길어질 경우 코드의 의도를 한 눈에 파악하기 힘들다.<br>
메서드 참조로 리팩터링 시 메서드명을 통해 의도를 명확히 알릴 수 있다. -> 코드 가독성 상승
```java
//원본
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel =
    menu.stream().collect(
                        groupingBy(dish -> {
                            if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                            else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                            else return CaloricLevel.FAT;
                        }));

//칼로리 수준 추출 메서드 분리
public class Dish{
    ...
    public CaloricLevel getCaloricLevel() {
        if (dish.getCalories() <= 400) return CaloricLevel.DIET;
        else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
        else return CaloricLevel.FAT;
    }
}

Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = 
    menu.stream().collect(
                        groupingBy(Dish::getCaloricLevel)
                  );
            


//static 메서드 사용
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
inventory.sort(comparing(Apple::getWeight));

//내장 컬렉터 사용
int totalCalories = menu.stream().map(Dish::getCalories).reduce(0, (c1, c2) -> c1 + c2);
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```
<br>
<br>

### 9.1.4 명령형 데이터 처리를 스트림으로 리팩터링하기
데이터 처리 파이프라인의 **의도**를 명확히 보여주며, <br>
쇼트서킷, 게으름 등의 최적화와 간편한 병렬 처리 지원.
```java
//원본
List<String> dishNames = new ArrayList<>();
for(Dish dish : menu){
    if(dish.getCalories() > 300){ //필터링
        dishNames.add(dish.getName()); //추출
    }
}

//스트림 사용
menu.parallelStream()
    .filter(d -> d.getCalories() > 300)
    .map(Dish::getName)
    .collect(toList());
```
<br>
<br>

### 9.1.5 코드 유연성 개선
람다 표현식 사용 시, <br>
다양한 람다를 전달하여 다양한 동작을 표현할 수 있고, 요구사항 변화에 대응하기 쉽다. <br>
또한, 함수형 인터페이스(1개의 추상 메서드를 갖는 인터페이스)가 필요하다.<br>
<br>
**조건부 연기 실행**<br>
제어 흐름문이 복잡하게 얽힌 코드를 흔히 볼 수 있다. 다음은 내장 자바 Logger 클래스를 사용하는 예제다.<br>
```java
if (logger.isLoggable(Log.FINER)) {
    logger.finer("Problem: " + generateDiagnostic());
}
//위 코드는 logger의 상태가 isLoggable 메서드에 의해 클라이언트 코드로 노출된다.
//또한 메시지를 로깅할때마다 logger의 객체를 매번 확인해야 한다
```

```java
logger.log(Lelvel.FINER, "Problem : " + generateDiagnostic());
//불필요한 if문을 제거하고 logger의 상태를 노출할 필요도 없어졌지만,
//logger가 활성화되어 있지 않더라도 항상 로깅 메시지를 평가하게 된다.
```

```java
logger.log(Level.FINER, () ->  "Problem : " + generateDiagnostic());
//자바8 API가 제공하는 supplier를 인수로 갖는 오버로드된 log 메서드를 통해 위와 같이 해결할 수 있다.

//아래는 log 메서드의 내부 구현 코드다.
public void log(Level level, Supplier<String> msgSupplier) {
    if(logger.isLoggable(level)) {
        log(level, msgSupplier.get()); //람다 실행
    }
}
```
<br>
**실행 어라운드**<br>
매번 같은 준비, 종료 과정을 반복적으로 수행하는 코드가 있다면 이를 람다로 변환할 수 있다.<br>
준비, 종료 과정 처리 로직을 재사용해 중복 코드를 줄일 수 있다.
이 코드는 파일을 열고 닫을 때 같은 로직을 사용했지만, 람다를 이용해 다양한 방식으로 파일을 처리할 수 있도록 파라미터화 되었다.
```java
String oneLine = processFile((BufferedReader b) -> b.readline()); //람다 전달
String twoLine = processFile((BufferedReader b) -> b.readline() + b.readline()); //다른 람다 전달

public static String processFile(BufferedReaderProcessor p) throws IOException {
    try(BufferedReader br = new BufferedReader(new fileReader("data.txt"))) {
        return p.process(br); //인수로 전달된 BufferedReaderProcessor 실행
    }
}

public interface BufferedReaderProcessor {
    string process(BufferedReader b) throws IOException;
}
```
