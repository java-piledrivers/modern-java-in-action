## 4.3 스트림과 컬렉션

  스트림과 컬렉션은 순차적으로 값에 접근하여 값을 저장하는 자료구조의 인터페이스를 제공한다.



## 스트림과 컬렉션의 차이점

  언제 계산하느냐가 가장 큰 차이점 이다.

  컬렉션 : 모든 요소가 컬렉션에 들어가기 전에 계산이 되어야 한다.

  스트림 : 요청 할때만 요소를 계산한다.(게으르게 만들어진 컬렉션)



## 딱 한번만 탐색할수 있는 스트림


  한번쓰면 끝!! 
  
  ```
  List<String> title = Arrays.asList("JAVA8","In","Action");
  Stream<String> s = title.stream();
  s.forEach(System.out::println);  //출력됨
  s.forEach(System.out::println);  //안됨
  ```
  
  
## 외부 반복과 내부 반복
  
  외부반복 : 사용자가 직접 요소를 반복한다. (For-each)
  
  ```
  List<String> names = new ArrayList<>();
  for(Dish dish : menu){
    names.add(dish.getName());
  }
  ```
  
  내부반복 : 반복을 알아서 처리하고 결과 스트림 값을 어디에 저장한다.
  
  ```
  List<String> names = menu.steram().Map(Dish::getName).collect(toList());
  ```
  
 ## 내부반복을 사용하면 좋은점
  
  병렬성을 관리해주어야 하는 for-each 과 같은 외부 반복과 달리 
  
  내부 반복은 병렬성 구현을 자동으로 선택한다 
