# 14.7 자동 모듈


`HttpReader`를 구현하지 않고 아파치의 `httpClient` 같은 라이브러리를 사용해 구현하려면 어떻게 추가해야 할까?  

module-info.java에 앞선 14.5 장에서 배운 requires 구문을 추가하고, 의존성을 기술하도록 `pom.xml`도 갱신해야 한다.

자바는 JAR을 자동 모듈이라는 형태로 적절히 변환한다.

모듈 경로상에 있으나 module-info.java 가 없는 모든 JAR은 자동 모듈된다. 
이 자동 모듈은 암묵적으로 자신의 모든 패키지를 노출시킨다. 자동 모듈의 이름은 JAR 이름을 이용해 정해진다. 
 

`--describe-module` 인수를 이용해 자동으로 정해지는 모듈이름을 바꿀수 있다.

```
jar --file=./expenses.reader/target/dependency/httpclient-4.5.3.jar \

     --describe-module httpclient@4.5.3 automatic
```
 

그리고 httpclient JAR을 모듈경로에 추가한 다음 애플리케이션을 실행한다.

```
java --module-path \

./expenses.application/target/expenses.application-1.0.jar \

./expenses.reader/target/expenses.reader-1.0.jar \

./expenses.readers/target/dependency/httpclient-4.5.3.jar \

 --moduel \ 

expenses.application/com.example.expenses.application.ExpensesApplication
```

