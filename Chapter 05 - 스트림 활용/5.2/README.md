## 스트림 슬라이싱

### 프레디케이트를 이용한 슬라이싱
- takewhile은 주어진 조건을 만족하는 동안 요소를 가져올 수 있게 해주는 함수. 조건이 거짓이 되는 순간, 반복이 종료되고 조건에 맞는 요소들이 반환된다.

- dropwhile은 주어진 조건을 만족하지 않는 요소를 가져오는 함수. 처음으로 조건이 거짓이 되는 순간부터 남은 모든 요소를 반환한다.

 *지난장에서 다룬것처럼 프레디케이트 함수란 참(True) 또는 거짓(False)을 반환하는 함수를 말한다. takewhile과 dropwhile는 각각 프레디케이트 함수로 요소에 대해 조건을 평가하고 그 결과에 따라 요소를 처리하는 데 사용됩니다.


### 스트림 축소

- limit(n)를 사용한다면 최대 요소 n개만 반환한다.
- skip(n)를 사용한다면 처음 n개를 제외하고 스트림 값을 반환한다. // 스트림의 크기가 n개 이하라면 빈 스트림을 반환한다.

정렬되지 않은 상태에서도 사용할 수 있지만 당연히 결과도 정렬되지 않은 상태로 반환됨 

### 사용법
```java

//리스트가 정렬되어있다고 가정
List<Dish> specialMenu = Arrays.asList(
    new Dish("seasonal fruit", true, 120, Dish.Type.OTHER),
    new Dish("prawns", false, 300, Dish.Type.FISH),
    new Dish("rice", true, 350, Dish.Type.OTHER),
    new Dish("chicken", false, 400, Dish.Type.MEAT),
    new Dish("french fries", true, 530, Dish.Type.OTHER));

//takewhile
List<Dish> slicedMenu1 = specialMenu.stream().takeWhile(dish -> dish.getCalories() < 320 ).collect(toList()); // seasonal fruit, prawns

//dropwhile
List<Dish> slicedMenu2 = specialMenu.stream().dropWhile(dish -> dish.getCalories() < 320 ).collect(toList()); // rice, chicken, french fries


//limit
List<Dish> dishs = specialMenu.stream().dropWhile(dish -> dish.getCalories() > 300 ).limit(2).collect(toList()); // prawns, rice

//skip
List<Dish> dishs = specialMenu.stream().dropWhile(dish -> dish.getCalories() > 300 ).skip(2).collect(toList()); // chicken, french fries
```



