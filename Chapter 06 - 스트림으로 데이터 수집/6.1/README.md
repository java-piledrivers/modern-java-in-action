컬렉터 

스트림 요소별로 돌아다니면서 변환 함수에 맞게 값을 변환후 모은다


컬렉터의 장점

스트림 요소를 하나의 값으로 리듀스하고 요약 집계,합계,평균 리듀싱기능을 사용 이런걸 요약연산 이라고 한다

int totalCalories = menu.stream().collect(SummingInt(Dish::getCalories));





요소 그룹화

Map<Dish.Type,List<Dish>> dishesByTypes = menu.stream().collect(groupingBy(Dish::getType));

{FISH=[prawns,salmon], OTHER={rice,french fries, season fruit}, Meat=[pork,beef,chicken]}



	
요소분할

거짓, 참 요소를 모두 유지

Map<Boolean,Map<Dish.Type,List<Dish>>> vegitarianDishesByTypes = menu.stream().collect(
partioningBy(Dish::isVegetarian, 분할함수 
groupingBy(Dish::getType));
