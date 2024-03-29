### 14.2 모듈화의 한계

자바 9 이전까지는 모듈화된 소프트웨어 프로젝트를 만드는데 한계 존재했다.
자바는 클래스, 패키지, JAR 세 가지 수준의 코드 그룹화를 제공하는데, 이 중 패키지와 JAR 수준에서는 캡슐화가 거의 지원되지 않았다.

 
 
### 제한된 가시성 제어

한 패키지의 클래스와 인터페이스를 다른 패키지로 공개하려면 public으로 선언했다.

결과적으로 이들 클래스와 인터페이스가 모두 공개되고, 사용자가 이 내부 구현을 마음대로 사용할 수 있게 된다.

 

### 클래스 경로

자바에서는 클래스를 모두 컴파일 한 다음 JAR 파일에 넣고 클래스 경로에 이 JAR 파일을 추가하여 번들로 사용할 수 있다.

이러한 클래스 경로와 JAR 조합에는 몇 가지 약점이 존재

1) 클래스 경로에는 같은 클래스를 구분하는 버전 개념이 없다. 
 - 예를들어 파싱 라이브러리의 JSONParser 클래스를 지정할 때 버전 1.0인지 2.0인지 지정할 수 없으므로 클래스 경로에 두 버전의 같은 라이브러리가 존재하면 어떤 문제가 발생할 지 예측할 수 없다.

2) 클래스 경로는 명시적인 의존성을 지원하지 않는다. 
 - 한 JAR가 다른 JAR에 포함된 클래스 집합을 사용한다고 명시적으로 의존하지 않기 때문에 누락된 JAR를 확인하기 어렵고, 충돌이 발생할 수 있다.


### 거대한 JDK
자바 개발 키트(JDK)는 자바 프로그램을 만들고 실행하는 데 도움을 주는 도구의 집합이다.

시간이 흐르면서 JDK의 덩치가 커졌고, 모바일이나 JDK 전부를 필요로 하지 않는 클라우드 환경에서 문제가 되었다.

자바8에서는 컴팩트 프로파일 기법으로 문제를 해결하려 했지만 이는 땜질식 처방 이었음

바의 낮은 캡슐화 지원 때문에 JDK의 내부 API가 외부에 공개되었고, 여러 라이브러리에서 JDK 내부 클래스를 사용했다.

결과적으로 호환성을 깨지 않고는 관련 API를 바꾸기 어려운 상황 이다.
