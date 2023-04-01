## 스트림 슬라이싱

### 프레디케이트를 이용한 슬라이싱
- takewhile은 주어진 조건을 만족하는 동안 요소를 가져올 수 있게 해주는 함수. 조건이 거짓이 되는 순간, 반복이 종료되고 조건에 맞는 요소들이 반환된다.

- dropwhile은 주어진 조건을 만족하지 않는 요소를 가져오는 함수. 처음으로 조건이 거짓이 되는 순간부터 남은 모든 요소를 반환한다.

 *지난장에서 다룬것처럼 프레디케이트 함수란 참(True) 또는 거짓(False)을 반환하는 함수를 말한다. takewhile과 dropwhile는 각각 프레디케이트 함수로 요소에 대해 조건을 평가하고 그 결과에 따라 요소를 처리하는 데 사용됩니다.


### 스트림 축소

- limit(n)를 사용한다면 최대 요소 n개만 반환한다.
- skip(n)를 사용한다면 처음 n개를 제외하고 스트림 값을 반환한다. // 스트림의 크기가 n개 이하라면 빈 스트림을 반환한다.

#### 정렬되지 않은 상태에서도 사용할 수는 있지만 의도와 다르게 값이 반환됨

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
List<Dish> dishs = specialMenu.stream().dropWhile(dish -> dish.getCalories() > 299 ).limit(2).collect(toList()); // prawns, rice

//skip
List<Dish> dishs = specialMenu.stream().dropWhile(dish -> dish.getCalories() > 299 ).skip(2).collect(toList()); // chicken, french fries
```

#### 정렬되지 않을 경우
```java

//리스트가 정렬되어있지 않을때
	List<Dish> specialMenu = Arrays.asList(
	    new Dish("seasonal fruit", true, 120, Dish.Type.OTHER),
	    new Dish("prawns", false, 300, Dish.Type.FISH),
		new Dish("rice", true, 350, Dish.Type.OTHER),
	    new Dish("chip", true, 100, Dish.Type.OTHER),
		new Dish("chicken", false, 400, Dish.Type.MEAT),
		new Dish("french fries", true, 530, Dish.Type.OTHER),
		new Dish("pizzea", true, 380, Dish.Type.OTHER),
		new Dish("fork", true, 450, Dish.Type.OTHER));
    

//takewhile
List<Dish> slicedMenu1 = specialMenu.stream().takeWhile(dish -> dish.getCalories() < 320 ).collect(toList()); 
// 사용자의 의도대로라면 320 미만인 seasonal fruit, prawns, chip 이 반환되어야하나, 실제로는 seasonal fruit, prawns만 반환된다. rice가 나오는순간 연산이 종료되기 때문

//dropwhile
List<Dish> slicedMenu2 = specialMenu.stream().dropWhile(dish -> dish.getCalories() < 320 ).collect(toList()); 
// 사용자의 의도대로라면 320를 초과하는 음식만 나와야하나 정렬되지 않은 chip이 포함되어 반환된다.


```


### 장점과 단점
장점: 
문자열이나 리스트의 경우에는 전체 데이터를 메모리에 로드해야만 슬라이싱을 적용할 수 있다.
하지만 스트림 슬라이싱은 데이터를 동적으로 처리하므로, 전체 데이터를 메모리에 로드하지 않고도 원하는 부분을 추출할 수 있다. 이러한 특성으로 큰 데이터셋인 경우에도 메모리에 전체 데이터를 로드할 필요없어 메모리 효율성이나 속도면에서 유리할 수 있다. 

단점:

스트림 슬라이싱의 takeWhile과 dropWhile은 조건에 따라 연산을 종료하므로, 정렬되지 않은 데이터에서는 사용자가 기대하는 결과를 얻지 못할 수 있다. 정렬되지 않은 상태에서 사용하면 예상하지 못한 결과를 얻을 수 있으므로, 사용자가 기대하는 결과를 얻으려면 데이터를 먼저 정렬한 후 takeWhile과 dropWhile을 사용해야한다.

