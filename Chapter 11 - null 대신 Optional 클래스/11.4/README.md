# 11.4 Optional을 사용한 실용 예제

## 11.4.1 잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기

```java
Object value = map.get("key");
```

위 처럼 null이 반환될 수 있는 코드는 Optional로 감싸서 개선할 수 있다.
```java
Optional<Object> value = Optional.ofNullable(map.get("key"));
```

## 11.4.2 예외와 Optional 클래스
null을 확인할 때는 if문을 사용했지만 예외를 발생하시키는 메서드는 try/catch 블록을 사용해야 한다.

```java
public static Optional<Integer> stringToInt(String s) {
  try {
    return Optional.of(Integer.parseInt(s));
  } catch (NumberFormatException e) {
    return Optional.empty();
  }
}
```

위 코드와 같이 정수로 변환할 수 없는 문자열은 빈 Optional로 return하도록 구현할 수 있다.


## 11.4.3 기본형 Optional을 사용하지 말아야 하는 이유
- Optional도 기본형으로 특화된 OptionalInt, OptionalLong, OptionalDouble 등의 클래스를 제공한다.
- 하지만 Optional의 최대 요소 수는 한개이므로 기본형 특화 Optional로 성능을 향상시킬 수 없다.
- 또한 기본형 특화 Optional은 map, flatMap, filter등을 제공하지 않고, 생성한 결과를 다른 일반 Optional과 혼용할 수 없다.

## 11.4.4 응용
```java
Properties props = new Properties();
props.setProperty("a", "5");
props.setProperty("b", "true");
props.setProperty("c", "-3");


// 프로퍼티에서 지속 시간을 읽는 명령형 코드
public int readDuration(Properties props, String name) {
    String value = props.getProperty(name);
    if(value != null) {
        try {
            int i = Integer.parseInt(value);
            if (i>0) {
                return i;
            }
        } catch (NumberFormatException nfe) { }
    }
    return 0;
}


// 지속 시간은 양수여야 하므로 문자열이 양의 정수이면 해당값 반환, 그 외에는 0 반환.
assertEqual(5, readDuration(param, "a"));
assertEqual(0, readDuration(param, "b"));
assertEqual(0, readDuration(param, "c"));
assertEqual(0, readDuration(param, "d"));
```



# 추가 자료
Java Optional 바르게 쓰기 -
https://homoefficio.github.io/2019/10/03/Java-Optional-%EB%B0%94%EB%A5%B4%EA%B2%8C-%EC%93%B0%EA%B8%B0/
