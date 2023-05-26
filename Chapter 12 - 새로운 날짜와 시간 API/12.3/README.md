# 새로운 날짜와 시간 API

java.util.Date 클래스 문제점

- 특정 시점을 날짜가 아닌 밀리초 단위로 표현한다
- 1900년을 기준으로 하는 오프셋, 0에서 시작하는 달 인덱스등 모호한 설계
- toString 으로 반환되는 문자열을 활용하기 어렵다.
- JVM 기본시간대 CET 중앙유럽시간대를 사용하므로 자체적 시간대 정보가 없음.
- 가변클래스

java.util.Calendar 클래스 문제점

- 달의 인덱스는 0에서 시작
- DateFormat 기능이 없음.
- 가변 클래스.
- DateFormat은 스레드에 안전하지 않으므로 두 스레드가 동시 하나의 포매터로 날짜 파싱할때 결과가 예기치 못함.

---

**12.1. LocalDate, LocalTime, Instant, Duration, Period 클래스**

**1 - LocalDate와 LocalTime 사용**

LocalDate 인스턴스는 시간을 제외한 날짜를 표현하는 불변 객체이고 어떤 시간대 정보도 포함하지 않는다.

정적 팩토리 메서드 of 로 LocalDate 인스턴스 생성한다.

```java
LocalDate date = LocalDate.of(2020,6,11);// 2020-06-11int year = date.getYear();// 2020
Month month = date.getMonth();// JUNEint day = date.getDayOfMonth();// 11
DayOfWeek dow = date.getDayOfWeek();// THURSDAYint len = date.lengthOfMonth();// 30boolean leap = date.isLeapYear();// false
```

현재날짜정보

LocalDate today = LocalDate.now();

get 메서드에 TemporalField를 전달해서 정보를 얻는 방법

-> TemporalField는 시간관련 객체에서 어떤 필드의 값에 접근할지 정의하는 인터페이스.

ChronoField의 열거자 요소를 이용해 정보를 얻음.

```java
int year = data.get(ChronoField.YEAR);
int month = data.get(ChronoField.MONTH_OF_YEAR);
int day = data.get(ChronoField.DAY_OF_MONTH);

// 내장 메서드 이용int year = date.getYear();
int month = date.getMonthValue();
int day = date.getDayOfMonth();

// LocalTime 만들고 값 읽기
LocalTime time = Localtime.of(13,45,20);
int hour = time.getHour();
int minute = time.getMinute();
int second = time.getSecond();

//날짜와 시간 문자열로 날짜, 시간 인스턴스 만들기
LocalDate date = LocalDate.parse("2017-09-21");
LocalTime time = LocalTime.parse(13:45:20);
```

**2 - 날짜와 시간 조합**

LocalDateTime은 LocalDate와 LocalTime을 쌍으로 갖는 복합 클래스이다.

```java
// LocalDateTime을 직접만들거나 조합하는 방법
LocalDateTime dt1 = LocalDateTime.of(2017, Month.SEPTEMBER, 21,13,45,20);
LocalDateTime dt2 = LocalDateTime.of(date, time);
LocalDateTime dt3 = date.atTime(13,45,20);
LocalDateTime dt4 = date.atTime(time);
LocalDateTime dt5 = time.atDate(date);

// 추출
LocalDateTime date1 = dt1.toLocalDate();
LocalDateTime time1 = dt1.toLocalTime();
```

**3 - Instant 클래스 : 기계의 날짜와 시간**

java.time.Instant 클래스에서는 기계적인 관점에서 시간을 표현한다.

Unix Epoch Time을 기준으로 시간을 초로 표현. 나노초(10억분의 1초)의 정밀도를 제공.

```java
Instant.ofEpochSecond(2, 1_000_000_000);// 2초 이후의 1억 나노초
Instant.ofEpochSecond(4, -1_000_000_000);// 2초 이전의 1억 나노초// 사람이 읽을수 있는 시간정보를 제공하지 않으므로// UnsupportedTemporalTypeException 예외 발생가능.
```

**4 - Duration과 Period 정의**

Temporal 인터페이스는 특정시간을 모델링하는 객체의 값을 어떻게 읽고 조작할지 정의한다.

Duration 클래스의 정적팩토리 메서드 between으로 두 시간객체의 지속시간을 만들 수 있다.

```java
Duration d1 = Duration.between(time1, time2);
Duration d1 = Duration.between(dateTime1, dateTime2);
Duration d2 = Duration.between(instant1, instant2);
// instant객체와 dateTime 객체는 혼용 불가능.// Period 클래스의 팩토리 메서드 between을 이용해 두 LocalDate의 차이 확인.
Period tenDays = Period.between(LocalDate.of(2017,9,11), LocalDate(2017,9,21));

// Duration과 Period 클래스가 제공하는 다양한 팩토리 메서드.
Duration threeMinutes = Duration.ofMinutes(3);
Duration threeMinutes = Duration.of(3, ChronoUnit.MINUTES);

Period tenDays = Period.ofDays(10);
Period threeWeeks = Period.ofWeeks(3);
Period twoYearsSixMonthsOneDay = Period.of(2,6,1);
```

---

**12.2 날짜 조정, 파싱, 포매팅**

바뀐속성을 포함하는 새로운 객체를 반환하는 메서드.

-> 객체를 바꾸는 것이 아니라 필드를 갱신한 복사본을 만든다. 함수형 갱신.

```java
LocalDate date1 = LocalDate.of(2017,9,21);// 2017-09-21
LocalDate date2 = date1.withYear(2011);// 2011-09-21
LocalDate date3 = date2.withDayOfMonth(25);// 2011-09-25
LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR,2);// 2011-02-25

LocalDate date2_1 = date1.plusWeeks(1);// 2017-09-28
LocalDate date3_1 = date2_1.minusYears(6);// 2011-09-28
LocalDate date4_1 = date3_1.plus(6, ChronoUnit.MONTHS);// 2012-03-28
```

plus, minus 메서드도 Temporal 인터페이스에 정의되어 있다.

ChronoUnit 열거형은 TemporalUnit 인터페이스를 쉽게 활용할 수 있는 구현을 제공한다.

LocalDate, LocalTime, LocalDateTime, Instant 등 날짜와 시간을 표현하는 클래스의 공통메서드

| 메서드 | 정적 | 설명 |
| --- | --- | --- |
| from | o | 주어진 Temporal 객체를 이용해 클래스의 인스턴스 생성 |
| now | o | 시스템 시계로 Temporal 객체를 생성 |
| of | o | 주어진 구성 요소에서 Temporal 객체의 인스턴스 생성 |
| parse | o | 문자열을 파싱해서 Temporal 객체를 생성 |
| atOffset | x | 시간대 오프셋과 Temporal 객체를 합침 |
| atZone | x | 시간대 오프셋과 Temporal 객체를 합침 |
| format | x | 지정된 포매터를 이용해 Temporal 객체를 문자열로 변환(Instant 지원 x) |
| get | x | Temporal 객체의 상태를 읽음 |
| minus | x | 특정시간을 뺀 Temporal 객체의 복사본 생성 |
| plus | x | 특정시간을 더한 Temporal 객체의 복사본 생성 |
| with | x | 일부 상태를 바꾼 Temporal 객체의 복사본 생성 |
1. **TemporalAdjusters 사용하기**

다음주 돌아오는 일요일, 어떤 달의 마지말 날을 구하려면??

```java
importstatic java.time.temporal.TemporalAdjusters.*;
LocalDate date1 = LocalDate.of(2014,3,18);
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY));
LocalDate date3 = date2.with(lastDayOfMonth());
```

TemporalAdjuster 커스텀 구현을 간단하게 만들수도 있다.

-> 하나의 메서드만 정의하므로 함수형 인터페이스 이다.

TemporalAdjuster 인터페이스를 UnaryOperator<Temporal>과 같은 형식으로 간주.

1. **날짜와 시간 객체 출력과 파싱**

포매팅과 파싱 전용 패키지 java.time.format

DateTimeFormatter의 정적팩토리 메서드 이용해 쉽게 포매팅 할수 있고

parse메서드를 이용해 문자열을 날짜 객체로 만들 수 있다.

```java
LocalDate date = LocalDate.of(2020,6,28);
String d1 = date.format(DateTimeFormatter.BASIC_ISO_DATE);// 20200628
String d2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE);// 2020-06-28

LocalDate date1 = LocalDate.parse("20200628", DateTimeFormatter.BASIC_ISO_DATE);
LocalDate date2 = LocalDate.parse("20200628", DateTimeFormatter.ISO_LOCAL_DATE);
```

특정 패턴으로 포매터를 만들어 쓸수도 있다.

Locale로 지역화된 포매터를 만들 수 있게 오버로드 메서드도 제공한다.

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate date1 = LocalDate.of(2020,6,28);
String formattedDate = date1.format(formatter);
LocalDate date2 = LocalDate.parse(formattedDate, formatter);

DateTimeFormatter italianFormatter = DateTimeFormatter.ofPattern("d. MMMM yyyy", Locale.ITALIAN);
LocalDate date3 = LocalDate.of(2020,3,18);
String formattedDate1 = date3.format(italianFormatter);// 28. marzo 2020
LocalDate date4 = LocalDate.parse(formattedDate1, italianFormatter);
```

DateTimeFormatterBuilder클래스로 복합적인 포매터를 정의해 제어 가능하다

프로그램 적으로 포매터를 만들수 있다.

```java
DateTimeFormatter italianFormatter =new DateTimeFormatterBuilder()
                                           .appendText(ChronoField.DAY_OF_MONTH)
                                           .appendLiteral(". ")
                                           .appendText(ChronoField.MONTH_OF_YEAR)
                                           .appendLiteral("")
                                           .appendText(ChronoField.YEAR)
                                           .parseCaseInsensitive()
                                           .toFormatter(Locale.ITALIAN);
```

---

**12.3 다양한 시간대와 캘린더 활용방법**

기본의 java.util.TimeZone 을 대체할 수 있는 java.time.ZoneId 클래스.

서머타임(DST) 같은 사항이 자동으로 처리.

**1 - 시간대 사용하기**

표준시간이 같은 지역을 묶어 time zone 규칙 집합 정의.

https://www.iana.org/time-zones 참고

getRules()이용해 규정을 획득한다.

```java
ZoneId romeZone = ZoneId.of("Europe/Rome");// {지역}/{도시}

LocalDate date = LocalDate.of(2020, Month.JUNE, 28);
ZonedDateTime zdt1 = date.atStartOfDay(romeZone);
LocalDateTime datetime = LocalDateTime.of(2020, Month.JUNE, 28, 13, 45);
ZoneDateTime zdt2 = dateTime.atZone(romeZone);
Instant instant = Instant.now();
ZoneDateTime zdt3 = instant.atZone(romeZone);

//ZoneId를 이용해 LocalDateTime을 Instant 로
Instant instant1 = Instant.now();
LocalDateTime timeFromInstant = LocalDateTime.ofInstant(instant, romeZone);
```

ZonedDateTime 개념

| LocalDate | LocalTime | ZoneId |
| --- | --- | --- |
| LocateDateTime |  |  |
| ZonedDateTime |  |  |

**2 - UTC/Greenwich 기준의 고정 오프셋**

UTC 협정 세계시 / GMT 그리니치 표준시 를 기준으로 시간대를 표현하기도 한다.

ZoneId의 서브클래스인 ZoneOffset 클래스로 기간값 차이 표현 가능하다.

```java
ZoneOffset newYorkOffset = ZoneOffset.of("-05:00");

LocalDateTime date = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45);
OffsetDateTime dateTimeInNewYork = OffsetDateTime.of(date, newYorkOffset);

```

**3 - 대안 캘린더 시스템 사용하기**

자바8에서 추가된 4개의 캘린터 시스템

ThaiBuddhistDate, MinguoDate, JapaneseDate, HijrahDate

프로그램 입출력을 지역화하는 상황을 제외하고는

모든 데이터 저장, 조작, 비지니스 규칙해석 등의 작업에서 LocalDate를 사용해야 한다.

---

- 자바 8이전에 util.Date 클래스 관련해서 불일치점 가변성, 어설픈 오프셋, 기본값, 잘못된 이름 등 설계결함 존재.
- 새로운 날짜 시간 API 에서 날짜와 시간 객체는 불변.
- 새로운 API는 사람 (LocalDateTime) 기계(Instant) 가 사용하도록 두가지 표현방식을 제공함.
- 날짜와 시간객체를 절대적인 방법과 상대적인 방법으로 처리할수 있고,기존 인스턴스를 변환하지 않게 처리 결과로 새로운 인스턴스가 생성.
- TemporalAdjuster를 이용해 복잡한 동작을 수행, 자신만의 커스텀 날짜 변환 기능을 정의 가능.
- 날짜와 시간 객체를 특정 포맷으로 출력하고 파싱하는 포매터 정의 가능. 패턴을 이용해 프로그램식으로 포매터 만들 수 있음.
- 지역 상대적인 시간대 또는 UTC/GMT 기분의 오프셋을 이용해 시간대를 정의 가능하고,이 시간대를 날짜와 시간객체에 적용해 지역화 가능
