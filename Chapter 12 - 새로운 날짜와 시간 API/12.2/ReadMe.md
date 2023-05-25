## 12.2 날짜 조정, 파싱, 포매팅

`withAttribute` 메소드로 `LocalDate` 를 변경할 수 있다.

이 때 `Temporal` 객체가 지정된 필드를 지원하지 않으면 `UnsupportedTemporalTypeException` 이 발생한다.

이 때 기존의 `Temporal` 객체를 바꾸는 것이 아니라 필드를 갱신한 복사본을 만든다.

``` java
// 절대적인 방식 LocalDate 속성 바꾸기
LocalDate date1 = LocalDate.of(2017, 9, 21); // 2017-09-21
LocalDate date2 = date1.withYear(2011); // 2011-09-21
LocalDate date3 = date2.withDayOfMonth(25); // 2011-09-25
LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR, 2); // 2011-02-25

// 상대적인 방식 LocalDate 속성 바꾸기
LocalDate date1 = LocalDate.of(2017, 9, 21); // 2017-09-21
LocalDate date2 = date1.plusWeeks(1); // 2017-09-28
LocalDate date3 = date2.minusYears(1); // 2011-09-28
LocalDate date4 = date3.plus(6, ChronoUnit.MONTHS); // 2012-03-28
```

- `withAttribute` 를 사용해서 절대적인 방식으로 속성을 바꿀 수 있다.
- `plus`, `minus` 등 메소드를 사용해서 상대적인 방식으로 속성을 바꿀 수 있다.



### 12.2.1 TemporalAdjusters 사용하기

좀 더 복잡한 방식으로 다양한 동작을 수행하려면 `TemporalAdjuster` 를 전달하는 방법으로 문제를 해결할 수 있다.

```java
import static java.time.temporal.TemporalAdjusters.*;

LocalDate date1 = LocalDate.of(2014, 3, 18); // 2014-03-18 (화)
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY)); // 2014-03-23
LocalDate date3 = date2.with(lastDayOfMonth()); // 2014-03-31
```



필요한 기능이 존재하지 않으면 커스텀 TemporalAdjuster 를 구현하여 사용할 수 있다.

```java
@FunctionalInterface
public interface TemporalAdjuster {
    Temporal adjustInto(Temporal temporal);
}
```



### 12.2.2 날짜와 시간 객체 출력과 파싱

포매팅과 파싱 전용 패키지인 `java.time.format`이 생겼다. 이 패키지에서 가장 중요한 클래스는 `DateTimeFormatter` 이다.

이것을 이용해서 날짜나 시간을 특정 형식의 문자열로 만들 수 있다.

```java
LocalDate date = LocalDate.of(2014, 3, 18);
String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE); // 20140318
String s2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE); // 2014-03-18
```



패턴으로 만들기

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPatterm("dd/MM/yyyy");
LocalDate date = LocalDate.of(2014, 3, 18);
String formattedDate = date1.format(formatter); // 요청 형식의 패턴에 해당하는 문자열 생성
LocalDate date2 = LocalDate.parse(formattedDate, formatter); // 같은 포매터를 적용해서 생성된 문자열을 파싱
```



`ofPattern`으로 `Locale` 로 포매터를 만들 수 있도록 제공한다. 따라서 포매터를 정의해서 좀 더 세부적으로 제어할 수 있다.

```java
DateTimeFormatter italianFormatter =
    DateTimeFormatter.ofPattern("d. MMMM yyyy", Local.ITALIAN);
LocalDate date1 = LocalDate.of(2014, 3, 18);
String formattedDate = date.format(italianFormatter); // 18. marzo 2014
LocalDate date2 = LocalDate.parse(formattedDate, italianFormatter)
```


