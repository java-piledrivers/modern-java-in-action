## 10.2 최신 자바 API의 작은 DSL

- 자바 8 이전에는 불필요한 코드가 추가되어야 했다. 람다와 메소드 참조가 등장하면서 **DSL 관점에서** 바뀌게 되었다.

- Comparator로 네이티브 자바API의 재사용성과 메서드 결합도 높힌 사례를 확인하자.

- 이것들은 컬렉션 정렬 도메인의 최소 DSL이다.

  ```java
  Collections.sort(persons, new Comparator<Person>(){
    public int compare(Person p1, Person p2){
      return p1.getAge() - p2.getAge();
    }
  })
    
  // lambda
  Collections.sort(people, (p1, p2) -> p1.getAge() - p2.getAge());
  
  // comparing
  Collections.sort(persons, comparing(p -> p.getAge()));
  
  // comparing + 메서드 참조
  Collections.sort(persons, comparing(Person::getAge));
  
  // 정렬
  Collections.sort(persons, comparing(Person::getAge).thenComparing(Person::getName));
  ```

### 10.2.1 스트림 API는 컬렉션을 조작하는 DSL

Stream 인터페이스는 네이티브 자바 API에 작은 내부 DSL을 적용한 좋은 예다.

아래를 참고하면 가독성과 유지보수성 모두가 저하되었다.

```java
List<String> errors = new ArrayList<>();
int errorCount = 0;
String fileName = "someFile.txt";
BufferedReader bufferedReader = new BufferedReader(new FileReader(fileName));

String line = bufferedReader.readLine();
while (errorCount < 40 && line != null) {
    if (line.startsWith("ERROR")) {
        errors.add(line);
        errorCount++;
    }

    line = bufferedReader.readLine();
}
```

함수형으로 개선된 예제이다.

Stream API의 플루언트 형식은 잘 설계된 DSL의 또 다른 특징이다.

```java
Files.lines(Paths.get(fileName)) // 파일을 열어서 문자열 스트림을 만듦
        .filter(l -> l.startsWith("ERROR")) // ERROR로 필터링
        .limit(40) // 40행으로 제한
        .collect(Collectors.toList()); // 결과를 리스트로 수집
```

### 10.2.2 데이터를 수집하는 DSL인 Collectors

전 장에서 Stream 인터페이스를 살펴보았다면, Collectors 인터페이스 또한 데이터 수집을 수행하는 DSL로 간주할 수 있다.

차를 브랜드와 색상으로 그룹화하는 로직이 있다고 했을 때

```
// 중첩형식
Map<String, Map<Color, List<Car>>> carsByBrandAndColor = cars
        .stream()
        .collect(groupingBy(Car::getBrand,
                groupingBy(Car::getColor)));
                
// 플루언트 방식
Comparator<Person> comparator = comparing(Person::getAge).thenComparing(Person::getName);
```

조합할 대상이 많은 경우에는 **보통 플루언트 형식이 중첩 형식에 비해 가독성이 좋다.**

다음과 같이 GroupingBuilder를 만들면 groupingBy 팩터리 메서드에 작업을 위임하여 유연하게 문제를 대처할 수 있다.

```
public class GroupingBuilder<T, D, K> {

    private final Collector<? super T, ?, Map<K, D>> collector;

    private GroupingBuilder(Collector<? super T, ?, Map<K, D>> collector) {
        this.collector = collector;
    }

    public Collector<? super T, ?, Map<K, D>> get() {
        return collector;
    }
    
    public <J> GroupingBuilder<T, Map<K, D>, J> after(Function<? super T, ? extends J> classifier) {
        return new GroupingBuilder<>(groupingBy(classifier, collector));
    }
    
    public static <T, D, K> GroupingBuilder<T, List<T>, K> groupOn(Function<? super T, ? extends K> classifier) {
        return new GroupingBuilder<>(groupingBy(classifier));
    }
}
```

하지만 이를 사용하게 될 때 **그룹화 함수를 반대로 써야 하므로 직관적이지 않은 문제가 있다.**

```
Collector<? super Car, ?, Map<Brand, Map<Color, List<Car>>>> carGroupingCollector = GroupingBuilder
    .groupOn(Car::getColor)
    .after(Car::getBrand).get();
```

 


