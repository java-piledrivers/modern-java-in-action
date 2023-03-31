스트림의 요소를 선택하는 방법

- Predicate을 이용한 필터링
- 고유 요소 필터링
    - Predicate을 이용한 필터링(filter)
    
    스트림 인터페이스가 지원하는 filter 메서드는 Predicate(boolean반환 함수)을 받아서 이에 부합하는 요소를 포함하는 스트림을 반환한다.
    
    ```java
    List vegetarianMenu = menu.stream()
    	.filter(Dish::isVegetarian)
        .collect(toList());
    ```
    
    - 고유 요소 필터링(distinct)
    
    중복요소가 있는 경우, 고유한 요소로 이루어진 스트림을 반환
