## 8.1 컬렉션 팩토리

자바 9에서는 컬렉션 객체를 쉽게 만들수 있는 몇 가지 방법을 제공한다.

```java
List<String> friends = new ArrayList<>();
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");
```

```java
List<String> friends = Arrays.asList("LEE", "SANG");
friends.set(0, "LEES");
friends.add("WON");
```

Unsupported OperationException 예외 발생

```java
Set<String> friends = Stream.of("LEE", "SANG","WON")
  .collect(Collector.toSet());
```


## 리스트 팩토리

```java
List<String> friends = List.of("LEE", "SANG", "WON");
```




## 집합 팩토리

```java
Set<String> friends = Set.of("LEE", "SANG", "WON");
```




## 맵 팩토리


```java
Map<String, Integer> friendsMap = Map.of("LEE", 1, "SANG", 2, "WON", 3);
```

```java
Map<String, Integer> ageOfFrieds = Map.ofEntries(
  entry("LEE", 1),
  entry("SANG", 2),
  entry("WON", 3));
```
