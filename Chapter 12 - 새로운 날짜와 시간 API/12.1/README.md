### LocalDate, LocalTime, Instant, Duration, Period 클래스


Date 클래스는 직관적이지 못하며 자체적으로 시간대 정보를 알고 있지 않다

Date를 사장 시키고 등장한 Calendar 클래스 또한 쉽게 에러를 일으키는 설계 문제를 갖고 있다


## LocalDate

```java
LocalDate date = LocalDate.of(2023, 05, 26); 
int year = date.getYear(); //2023
Month month = date.getMonth(); //05
int day = date.getDayOfMonth(); //26
LocalDate now = LocalDate.now(); // 현재 날짜 정보
```


## LocalTime

```java
LocalTime time = LocalTime.of(13, 45, 20); // 13:45:20
int hour = time.getHour(); // 13
int minute = time.getMinute(); // 45
int second = time.getSecond(); // 20
```

```java
LocalDate date = LocalDate.parse("2020-12-22");
LocalTime time = LocalTime.parse("13:45:20");
```
parse를 사용해서 날짜와 시간 문자열로 LocalDate와 LocalTime의 인스턴스를 만들 수 있다.


## LocalDateTime

```java
LocalDateTime dateTime = LocalDateTime.of(2023, Month.MAY, 26, 16, 37, 20);
LocalDate date = dateTime.toLocalDate(); // 2023-05-26
LocalTime time = dateTime.toLocalTime(); // 16:37:20
```


# Instant 
```java
Instant.ofEpochSecond(2,1_000_000_000); // 2초후 1억 나노초
Instant.ofEpochSecond(3,-1_000_000_000); // 3초전 1억 나노초
```
기계 전용이라 now() 안됨


# Duration , Period

Duration : 두 객체(시,분,초,나노초)의 지속시간 
Period : 두 객체 (년,월,일)의 지속시간

```java
Duration threeMinutes = Duration.ofMinutes(3);
Period tenDats = Periods.ofDays(10);
```



