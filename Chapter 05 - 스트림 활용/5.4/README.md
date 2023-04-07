# 검색과 매칭
* 스트림 API 제공 유틸리티 메서드 - anyMatch, allMatch, noneMatch, findAny, findFirst 
* 특정 속성이 데이터 집합에 있는지 여부를 검색하는 데이터 처리
<br/><br/>

## anyMatch
스트림의 요소 중 하나라도 프레디케이트와 일치 시 true를 반환하고 종료
```java
if( menu.stream().anyMatch(Dish::isVegetarian) ) {
    System.out.println( "채식 요리가 있다." );
}
```
menu의 요소 중 하나라도 채식인 경우, 그 시점에 true 반환 후 종료 - 쇼트서킷<br/>
-> 결과가 스트림이 아니므로 최종연산, p176 참고
<br/><br/>

## allMatch
스트림의 모든 요소가 프레디케이트와 일치 시 true를 반환, 하나라도 불일치 시 false 반환 후 종료
```java
boolean isHealthy = menu.stream()
                        .allMatch(dish -> dish.getCalories() < 1000);
```
menu의 모든 요소의 칼로리가 1000 미만인지 비교 후, true 반환<br/>
연산 중 칼로리 1000 이상인 요소가 있을 경우, 그 시점에 false 반환 후 종료
<br/><br/>

## noneMatch
스트림의 모든 요소가 프레디케이트와 불일치 시 true를 반환, 하나라도 일치 시 false 반환 후 종료 
```java
boolean isHealthy = menu.stream()
                        .noneMatch(dish -> dish.getCalories() >= 1000);  //위 allMatch 예제와 동일한 결과
```
menu의 모든 요소의 칼로리가 1000 이상인지 비교 후, true 반환<br/>
연산 중 칼로리 1000 미만인 요소가 있을 경우, 그 시점에 false 반환 후 종료
<br/><br/>
<br/>

## findFirst
스트림의 첫 번째 요소를 반환 
```java
List<Integer> someNumbers = Arrays.asList(1, 2, 3, 4, 5);
Optional<Integer> firstSquareDivisibleByThree = someNumbers.stream()
                                                           .map(n -> n * n)
                                                           .filter(n -> n % 3 == 0)
                                                           .findFirst();  //9
```
<br/><br/>

## findAny
스트림의 임의의 한 요소를 반환
```java
Optional<Dish> dish = menu.stream()
                          .filter(Dish::isVegetarian)
                          .findAny();
```
채식 메뉴인 요소 중 하나를 반환
<br/><br/>

## findFirst vs findAny
* 직렬 연산 시: 동일한 요소를 반환<br/>
* 병렬 연산 시: findFirst - 가장 첫 번째 요소를 반환<br/>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
  findAny   - 멀티 쓰레드에서 연산 중 가장 먼저 찾은 요소를 반환 -> 뒤쪽 요소가 반환될 가능성
```java
List<String> elements = Arrays.asList("a", "a1", "b", "b1", "c", "c1");

//findFirst
Optional<String> firstElement = elements.stream().parallel()
                                                 .filter(s -> s.startsWith("b"))
                                                 .findFirst();
                                                 
System.out.println(firstElement.get());  //findFirst 사용 시 항상 b 출력


//findAny
Optional<String> anyElement = elements.stream().parallel()
                                               .filter(s -> s.startsWith("b"))
                                               .findAny();
                                               
System.out.println(anyElement.get());  //findAny 사용 시 실행할 때마다 반환 값이 달라지며, b1 또는 b를 출력
```
<br/><br/>

<hr/>

## 쇼트서킷
불필요한 연산을 생략함으로써 성능을 개선하는 연산 방식
```java
void logicalOperations() { 
    System.out.println( false && returnBoolean() ); // 우항과 무관하게 false 이므로 returnBoolean()은 호출되지 않음
    System.out.println( true || returnBoolean() ); // 우항과 무관하게 true 이므로 returnBoolean()은 호출되지 않음
} 

private boolean returnBoolean() { 
    System.out.print("returnBoolean 호출"); 
    return true; 
}

출력
false
true
```
* 스트림에서의 쇼트서킷 - 반환할 스트림 요소가 확정될 경우 나머지 스트림 요소에 대한 연산은 생략
<br/><br/>

## Optional
값의 존재나 부재여부를 표현하는 컨테이너 클래스로, null은 쉽게 에러를 일으킬 수 있어 만들어짐 - 자세한 설명 11장<br/>
값이 존재하는지 확인 후, 없을 때 처리 방법을 강제하는 기능 제공
* isPresent() - Optional이 값을 포함하면 참(true) 을 반환, 값을 포함하지 않으면 거짓을 반환
* ifPresent(Consumer\<T\> block) - 값이 있으면 주어진 블록을 실행
* T get() - 값이 존재하면 값을 반환, 값이 없으면 NoSuchElementException
* T orElse(T default) - 값이 있으면 반환, 없으면 기본값
<br/><br/>
