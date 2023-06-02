# 14.6 컴파일과 패키징

이제 빌드 도구를 사용하여 프로젝트 컴파일을 하는 방법을 살펴보자.  

먼저 각 모듈에 `pom.xml`을 추가한다. 각 모듈은 독립적으로 컴파일되므로 자체적으로 각각이 한 개의 프로젝트다. 그리고 전체 프로젝트 빌드를 조정할 수 있도록 모든 모듈의 부모 모듈에도 `pom.xml`을 추가한다. 



메이븐 디렉터리 프로젝트 구조

```txt
|-- pom.xml
|-- expenses.application
	|-- pom.xml
    |-- src
    	|-- main
        	|-- java
            	|-- module-info.java
                |-- com
                	|-- example
                    	|-- expenses
                        	|-- application
                            	|-- ExpensesApplication.java
|-- expenses.readers
	|-- pom.xml
    |-- src
    	|-- main
        	|-- java
            	|-- module-infojava
                |-- com
                	|-- expenses
                    	|-- readers
                        	|-- Reader.java
                        |-- file
                        	|-- FileReader.java
                        |-- http
                        	|-- HttpReader.java


```

`expenses.application`과 `expenses.readers` 그리고 최상의에 `pom.xml` 까지 총 세 개의 `pom.xml` 파일로 구성되어 있다. 


**`expenses.readers` 의 `pom.xml` 일부**


```xml
<groupId>com.example</groupId>
<artifactId>expenses.readers</artifactId>
<parent>
  <groupId>com.example</groupId>
  <artifactId>expenses</artifactId>
</parent>

```

xml 코드를 살펴보면 순조롭게 빌드될 수 있도록 명시적으로 부모 모듈을 지정하였다.  

**`expenses.application` 의 `pom.xml` 일부**


```xml
<groupId>com.example</groupId>
<artifactId>expenses.application</artifactId>
<parent>
  <groupId>com.example</groupId>
  <artifactId>expenses</artifactId>
</parent>

<dependencies>
  <dependency>
    <groupId>com.example</groupId>
	<artifactId>expenses.readers</artifactId>
  </dependency>
</dependencies>
```
`expenses.application`는 ExpenseApplication이 필요로 하는 클래스와 인터페이스가 있으므로 `expenses.readers`를 의존성으로 추가해야 한다.  


**전역 `pom.xml` 일부**

```xml
<groupId>com.example</groupId>
<artifactId>expenses.application</artifactId>
<modules>
  <module>expenses.application</module>
  <module>expenses.readers</module>
<modules>

  
<bulid>
  중략
</build>
```

두 개의 자식 `application`과 `readers`을 참조하도록 완성한 `pom.xml` 이다.  

이제 mvn clean package 명령을 통해서 프로젝트의 모듈을 jar로 만들 수 있다.  

```
./expenses.application/target/expenses.application-1.0.jar

./expenses.reader/ target/expenses.reader-1.0.jar
```

두개의 JAR 만들어진다.

두 JAR를 모듈경로에 포함해서 모듈 애플리케이션을 실행할 수 있다.  

```
java --module-path \

./expenses.application/target/expenses.application-1.0.jar \

./expenses.reader/ target/expenses.reader-1.0.jar \

 --moduel \ 

expenses.application/com.example.expenses.application.ExpensesApplication
```
