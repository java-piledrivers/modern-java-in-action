
### 1) Predicate 필터링

스트림 인터페이스는 filter 메소드를 지원하는데, 이 filter 메소드는 **predicate** 함수를(Boolean 타입 함수) 를 인수로 받아서 프리디케이트와 일치하는 모든 요소를 포함하는 스트림을 반환합니다.

*Stream Predicate 예시*

```java
List<Dish> vegetarianMenu = menu.stream()
				.filter(Dish::isVegetarian)
                                .collect(toList());
```

---

### 2) 고유 요소 필터링 (Distinct)

스트림은 중복된 요소를 제거하고 고유 요소로 이루어진 스트림을 반환하는 **distinct** 메소드도 지원합니다.

- 고유 여부는 스트림에서 만든 객체의 **hashCode**, **equals** 로 결정합니다.

Stream *Distinct* 예시
```java
List<Integer> numbers = Arrays.asList(1,2,3,4,5,6,7,8,123,41,5,1,2,3,4,5,5,6,7,8)
        .stream()
        .filter(number->number%2==0)
        .distinct()
        .collect(Collectors.toList());
```
