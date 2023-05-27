**12.3 다양한 시간대와 캘린더 활용방법**

기본의 java.util.TimeZone 을 대체할 수 있는 java.time.ZoneId 클래스.

서머타임(DST) 같은 사항이 자동으로 처리.

**1 - 시간대 사용하기**

표준시간이 같은 지역을 묶어 time zone 규칙 집합 정의.

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
