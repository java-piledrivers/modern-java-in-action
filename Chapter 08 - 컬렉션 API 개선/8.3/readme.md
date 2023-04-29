# 맵 처리

자바 8부터 Map 인터페이스에 몇가지 디폴트 메서드가 추가됐다.

## foreach()

키와 값을 인수로 하는 Biconsumer를 인수로 받는 forEach메서드를 활용해서 확인할 수 있다.

```java
Map<String, String> user = Map.of("hodong123", "김호동", "gildong443", "홍길동", "tomsday", "tom");
user.forEach((id, name) -> System.out.println(id + " / " + name));
```

## 정렬 메서드

Entry.comparingByValue, Entry.comparingByKey를 넘겨 맵을 키와 값을 기준으로 정렬 할 수 있다.

- comparingByValue: value 기준으로 정렬
- comparingByKey: key 기준으로 정렬실행 결과
    
    ```java
    Map<String,String> favoriteFruit = Map.ofEntries(
          entry("홍길동", "사과"),
          entry("김계란", "바나나"),
          entry("강호동", "파인애플")
    );
    favoriteFruit.entrySet()
          .stream()
          .sorted(Map.Entry.comparingByKey())
          .forEachOrdered(System.out::println);
    ```
    
    ```
    강호동=파인애플
    김계란=바나나
    홍길동=사과
    ```
    

## getOrDefault()

기존에는 찾으려는 키가 없는 경우 null이 반환되는데 getOrDefault메서드는 키가 없으면 기본값을 반환할 수 있다.

```java
System.out.println(favoriteFruit.getOrDefault("이수근","키위"));// 키위 출력
```

## 계산 패턴

Map에 key가 존재하는지에 따라 어떤 동작을 실행하고 결과를 저장해야 하는지 상황일 때 사용하는 패턴

- computeIfAbsent: 제공된 키에 해당하는 값이 없거나 null이면 , key를 이용해 새값을 계산하고 맵에 추가(정보를 캐시할 때 사용)
- computeIfPresent: 제공된 키가 존재하면 새값을 계산하고 맵에 추가
- compute: 제공된 키로 새값을 계산하고 맵에 추가

## 삭제 패턴

key가 특정한 값과 연관되었을 때만 제거하는 오버로드 버전 메서드

```java
Map<String,String> like =new HashMap<>();
like.put("강호동", "사과");
like.put("이수근", "파인애플");
like.put("김계란", "바나나");
System.out.println(like);
like.remove("김계란", "다른과일");
System.out.println(like);
like.remove("김계란", "바나나");
System.out.println(like);
```

실행 결과

```
{강호동=사과, 이수근=파인애플, 김계란=바나나}
{강호동=사과, 이수근=파인애플, 김계란=바나나}
{강호동=사과, 이수근=파인애플}
```

## 교체 패턴

map의 항목을 바꾸는 데 사용할 수 있는 메서드

- replaceAll: BiFuction을 적용한 결과로 각 항목의 값을 교체
- replace: 키가 존재하면 값을 바꿈, 키가 특정 맵으로 매핑 되었을때만 교체하는 메서드도 존재putAll()

    
## 합침
    
- 특정 맵의 데이터를 모두 put
- 중복된 키는 덮어씀
    
    ```java
    // base data
    Map<String,String> favoriteMovie =new HashMap<>();
    favoriteMovie.put("아빠","스타워즈");
    favoriteMovie.put("엄마","해리포터");
    favoriteMovie.put("애기","반지의제왕");
    Map<String,String> dislikedMovie =new HashMap<>();
    dislikedMovie.put("아빠","스파이더맨");
    dislikedMovie.put("엄마","배트맨");
    dislikedMovie.put("할아버지","앤트맨");
    
    ```
   

# 개선된 ConcurrentHashMap

- 동시성 친화적이며 최신 기술을 반영한 HashMap
- 특정 부분만 잠궈 동시에 추가, 갱신 작업이 가능
- 동기화된 HashTable 버전에 비해 읽기, 쓰기 연산이 월등함
