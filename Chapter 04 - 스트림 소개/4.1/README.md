## 스트림

자바 8 API에 새로 추가된 기능으로, 질의로 표현하는 방식인 선언형으로 컬렉션 데이터를 처리

## 스트림의 특징

-   **선언형**: 제어 블록(루프, 조건문)을 이용해서 어떻게 동작을 구현할지 지정할 필요 없이 동작의 수행 지정을 지정할 수 있다.
-   **조립 가능**: filter, sorted, map, collect와 같은 여러 빌딩 블록 연산을 연결해 복잡한 데이터를 처리하는 파이프라인을 만들 수 있다.
-   **병렬화**: filter와 같은 연산은 특정 스레딩 모델에 제한되지 않고 자유롭게 사용할 수 있다. 띠리서 데이터 처리 과정을 병렬화할 수 있다.

```java
//자바 7 코드
List<Dish> lowCaloricDishes = new ArrayList<>(); //중간 변수
for (Dish dish : menu) {
    if (dish.getCalories() < 400) {
        lowCaloricDishes.add(dish);
    }
}
Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
    public int compare(Dish dish1, Dish dish2) {
        return Integer.compare(dish1.getCalories(), dish2.getCalories());
    }
});

List<String> lowCaloricDishesName = new ArrayList<>();
for (Dish dish : lowCaloricDishes) {
    lowCaloricDishesName.add(dish.getName());
}
```

```java
//자바 8 코드
List<String> lowCaloricDishesName = menu.stream()
                                        .filter(d -> d.getCalories() < 400)
                                        .sorted(comparing(Dishes::getCalories))
                                        .map(Dish::getName)
                                        .collect(toList());
```
