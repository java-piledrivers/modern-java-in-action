## 14.4 자바 모듈 시스템으로 애플리케이션 개발하기

### 1 - 애플리케이션 셋업

**ex) 비용처리 애플리케이션**

- 파일이나 URL에서 비용 목록을 읽는다
- 비용의 문자열 표현을 파싱한다
- 통계를 계산한다
- 유용한 요약 정보를 표시한다
- 각 태스크의 시작, 마무리 지점을 제공한다

 

**개념을 모델링 할 여러 클래스와 인터페이스 정의** 

- 다양한 소스에서 데이터를 읽음(ex. 지출) <u>(Reader, HttpReader, FileReader)</u>
- 도메인 객체를 구체화 <u>(Expense)</u>
- 다양한 포맷으로 구성된 데이터를 파싱 <u>(Parser, JSONParser, ExpenseJSON-Parser)</u>
- 통계를 계산하고 반환 <u>(SummaryCalculator, SummaryStatistics)</u>
- 다양한 기능을 분리조정 <u>(ExpensesApplication)</u>

**각 기능을 그룹화**

- expenses.readers
- expenses.readers.http
- expenses.readers.file
- expenses.parsers
- expenses.parsers.json
- expenses.model
- expenses.statistics
- expenses.application



이 예제는 모듈 시스템의 여러 부분이 두드러질 수 있도록 잘게 분해했다. 이렇게 잘게 분해하는 경우

단점 : 초기비용이 높아진다.

장점 : 프로젝트가 커지면서 캡슐화와 추론의 장점이 두드러진다.

### 2 - 세부적인 모듈화와 거친 모듈화

모듈화시 모듈의 크기를 결정해야 한다.

<u>세부적 모듈화 기법</u>은 모든 패키지가 자신의 모듈을 갖는다.

<u>거친 모듈화 기법</u>은 한 모듈이 시스템의 모든 패키지를 포함한다.

가장 좋은 방법은 이해하기 쉽고 고치기 쉽게 적절하게 모듈회되어있는지 주기적으로 확인하는 것이다.

 

### 3 - 자바 모듈 시스템 기초

모듈화 애플리케이션을 어떻게 실행할 수 있을까?

보통 IDE와 빌드 시스템에서 명령을 자동으로 처리하지만 어떤 동작을 수행하는지 살펴보자

```
expenses.application

L module-info.java
L com
  L example
   L expenses
    L application
     L ExpensesApplication.java 
```

최상위에 module-info.java 는 이름만 정의되어있고 아직 비어있다.

이 파일은 모듈 디스크립터로 모듈의 의존성, 외부에 어떤기능을 노출할지 정의한다.

 

**모듈 소스 디렉터리에서 다음 명령을 실행 =>** **자바 애플리케이션을 JAR로 패키징 하는 표준 방법**

```
javac module-info.java com/example/expenses/application/ExpensesApplication.java -d target

jar cvfe expenses-application.jar com.example.expenses.application.ExpensesApplication -D target
```

생성된 JAR을 모듈화 애플리케이션으로 실행

```
java --module-path expenses-application jar --module expenses/com.example.expenses.application.ExpensesApplication
```

 

**자바 .class 파일을 실행할 때 추가된 두가지 옵션**

- --module-path: 어떤 모듈을 로드할 수 잇는지 지정.

- --module: 실행할 메인 모듈과 클래스를 지정.

