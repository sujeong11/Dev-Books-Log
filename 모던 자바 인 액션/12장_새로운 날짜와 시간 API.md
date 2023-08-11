## 12.1 LocalDate, LocalTime, Instant, Duration, Period 클래스

java.time 패키지에서 해당 클래스들을 제공한다. 해당 클래스는 모두 `불변`이다.

> 위 클래스들은 `Tmporal 인터페이스`를 구현하는데, Temporal 인터페이스는 **특정 시간을 모델링하는 객체의 값을 어떻게 읽고 조작할지 정의**한다.
> 

### LocalDate와 LocalTime 사용

[1] `LocalDate`: 시간을 제외한 날짜를 표현하는 불변 객체 (어떤 시간대 정보도 포함X)

정적 팩토리 메서드 `of`로 인스턴스 생성

```java
LocalDate date = LocalDate.of(2017, 9, 21); // 2017-09-21
// 내장 메서드 사용
int year = date.getYear(); // 2017
Month month = date.getMonth(); // SEPTEMBER
int day = date.getDayOfMonth(); // 21
DayOfWeek dow = date.getDayOfWeek(); // THURSDAY
int len = date.lengthOfMonth(); // 31
boolean leap = date.isLeapYear(); // false (윤년)
```

펙토리 메서드 `now`는 시스템의 정보를 이용해 **현재 날짜 정보**를 얻는다.

```java
LocalDate today = LocalDate.now();
```

[2] `LocalTime`

1. 시간과 분을 인수로 받는 `of` 메서드
2. 시간과 분, 초를 인수로 받는 `of` 메서드

```java
LocalTime time = LocalTime.of(13, 45, 20); // 13:45:20
// 내장 메서드 사용
int hour = time.getHour();
int minute time.getMinute();
int second = time.getSecond();
```

- 문자열로 LocalDate와 LocalTime의 인스턴스를 생성 가능 ⇒ `parse` 정적 메서드 사용

```java
LocalDate date = LocalDate.parse("2017-09-21");
LocalTime time = LocalTime.parse("13:45:20");
```

### 날짜와 시간 조합

`LocalDateTime`: LocalDate와 LocalTime을 쌍으로 갖는 **복합 클래스.**

```java
LocalDateTime dt1 = LocalDateTime.of(2017, Month.SEPTEMBER, 21, 13, 45, 20);
```

```java
LocalDateTime dt2 = LocalDateTime.of(date, time); // 날짜와 시간을 조합
```

```java
// LocalDate의 atTime()에 시간을 제공하거나 LocalTime의 atDate()에 날짜를 제공해 생성
LocalDateTime dt3 = date.atTime(13, 45, 20);
LocalDateTime dt4 = date.atTime(time);
LocalDateTime dt5 = time.atDate(date);
```

LocalDate나 LocalTime 인스턴스를 추출할 수 있다.

```java
LocalDatedate1 date1 = dt1.toLocalDate(); // 2017-09-21
LocalTime time1 = dt1.toLocalTime(); // 13:45:20
```

### Instant 클래스 : 기계의 날짜와 시간

기계에서는 주, 날짜, 시간, 분의 단위로 시간을 표현하기가 어렵다. 기계에서는 특정 지점을 하나의 큰 수로 표현하는 것이 가장 자연스러운 표현 방법이다. 
=> `Instant 클래스`: 유닉스 에포크 시간을 기준으로 특정 지점까지의 시간을 초로 표현

```java
Instant.ofEpochSecond(3); // 초를 인수에 넘겨줘서 인스턴스 생성
// 두 번쨰 인수로 나노초 단위로 시간 보정 가능
Instant.ofEpochSecond(2, 1_000_000_000); // 2초 이후의 1억 나노초(1초)
```

해당 클래스는 기계 전용의 유틸리티이므로 사람이 읽을 수 있는 시간 정보를 제공하지 않는다.

### Duration과 Period 정의

[1] `Duration`

정적 팩토리 메서드 `between`으로 **두 시간 객체 사이의 지속 시간**을 만들 수 있다. **초와 나노초로 시간 단위를 표현**한다.

```java
Duration d1 = Duration.between(time1, time2);
Duration d1 = Duration.between(dateTime1, dateTime2);
Duration d2 - Duration.between(instant1, instant2);
```

[2] `Period`

정적 팩토리 메서드 `between`으로 **두 LocalDate의 차이를 확인**할 수 있다.

```java
Period tenDays = Period.between(LocalDate.of(2017, 9, 11); LocalDate.of(2017, 9, 21));
```

<br>

## 12.2 날짜 조정, 파싱, 포매팅

`write / plus / minus 메서드들`은 **Temporal 인터페이스의 정의**되어 있고, **Temporal 객체의 필드 값을 읽거나 고칠 수 있어** 바꾼 버전의 `새로운` 객체를 만들 수 있다. (모든 메서드는 기존 객체를 바꾸지 않는다.)

- 절대적인 방식

```java
LocalDate date1 = LocalDate.of(2017, 9, 21); // 2017-09-21
LocalDate date2 = date1.withYear(2011); // 2011-09-21
LocalDate date3 = date2.withDayOfMonth(25); // 2011-09-25
LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR, 2); // 2011-02-25
```

- 상대적인 방식

```java
LocalDate date1 = LocalDate.of(2017, 9, 21); // 2017-09-21
LocalDate date2 = date1.plusWeeks(1); // 2017-09-28
LocalDate date3 = date2.minusYears(6); // 2011-09-28
LocalDate date4 = date3.plus(6, ChronoUnit.MONTHS); // 2012-03-28
// ChronoUnit 열거형은 TemporalUnit인터페이스를 쉽게 활용할 수 있는 구현 제공
```

### TemporalAdjusters 사용하기

돌아오는 평일, 어떤 달의 마지막 날 등 좀 더 복잡한 날짜 조정 기능이 필요할 때는 `with 메서드`에 **더 다양한 기능을 제공**하는 `TemporalAdjusters`를 전달하면 된다.

- 다양한 정적 팩토리 메서드를 제공

```java
import static java.time.temporal.TemporalAdjusters.*;
LocalDate date1 = LocalDate.of(2014, 3, 18); // 2014-03-18
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY)); // 2014-03-23
LocalDate date3 = date2.with(lastDatOfMonth()); // 2014-03-31
```

필요한 기능이 정의되어 있지 않을 때는 비교적 쉽게 커스텀 TemporalAdjusters 구현을 만들 수 있다.

### 날짜와 시간 객체 출력과 파싱

java.time.format이 새로 추가되었다. 이 패키지에서 가장 중요한 클래스는 DateTimeFormatter이다.

`DateTimeFormatter 클래스`: BASIC_ISO_DATE와 ISO_LOCAL_DATE 등의 상수를 미리 정의하고 있다. 날짜나 시간을 특정 형식의 문자열로 만들 수 있다.

```java
LocalDate date = LocalDate.of(2014, 3, 18);
String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE); // 20140318
String s2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE); // 2014-03-18

// 모든 클래스의 팩터리 메서드 parse를 이용 가능
LocalDate date1 = LocalDate.parse("20140318", DateTimeFormatter.BASIC_ISO_DATE);
LocalDate date1 = LLocalDate.parse("2014-03-18", DateTimeFormatter.ISO_LOCAL_DATE);
```

기존의 java.util.DateFormat 클래스와 달리 위 클래스는 **스레드에서 안전하게 사용할 수 있는 클래스**이다.

- 특정 패턴으로 포매터를 만들 수 있는 정적 팩터리 메서드 `ofPattern`를 제공한다.

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate date1 = LocalDate.of(2014, 3, 18);
String formattedDate = date1.format(formatter);
LocalDate date2 = LocalDate.parse(formattedDate, formatter);
```

- `DateTimeFormatterBuilder` 클래스로 복합적인 포매터를 정의해서 좀 더 세부적으로 포매터를 제어할 수 있다.

```java
DateTimeFormatter italianFormatter = new DateTimeFormatterBuilder()
        .appendText(ChronoField.MONTH_OF_MONTH)
        .appendLiteral(". ")
        .appendText(ChronoField.MONTH_OF_YEAR)
        .appendLiteral(". ")
        .appendText(ChronoField.YEAR)
        .parseCaseInsensitive()
        .toFormatter(Locale.ITALIAN);
```

<br>

## 12.3 다양한 시간대와 캘린더 활용 방법

기존의 java.util.TimeZone을 대체할 수 있는 java.time.ZoneId 클래스(`불변 클래스`)가 새롭게 등장했다. 새로운 클래스를 이용하면 서머타임과 같은 사항이 자동으로 처리된다.

### 시간대 사용하기

표준 시간이 같은 지역을 묶어서 시간대 규칙 집합을 정의한다. 약 40개 정도의 시간대가 있다. 

- 지역 ID로 특정 ZoneId를 구분한다.

| 지역 ID는 ‘**`{지역}/{도시}`**’ 형식으로 이루어진다.

```java
ZoneId romeZone = ZoneId.of("Europe/Rome");
```

- ZoneId 객체를 얻은 다음에는 LocalDate, LocalDateTime, Instant를 이용해서 ZonedDateTime 인스턴스로 변환할 수 있다.

```java
LocalDate date = LocalDate.of(2014, Month.MARCH, 18);
ZonedDateTime zdt1 = date.atStartOfDay(romeZone);
LocalDateTime dateTime = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45);
ZonedDateTime zdt2 = dateTime.atZone(romeZone);
Instant instant = Instant.now();
ZonedDateTime zdt3 = Instant.atZone(romeZone);
```

<br>

## 12.4 마치며

- 새로운 날짜와 시간 API에서 날짜와 시간 객체는 모두 불변
- 새로운 API는 각각 사람과 기께가 편리하게 날짜와 시간 정보를 관리할 수 있도록 두 가지 표현 방식을 제공
- 날짜와 시간 객체를 절대적인 / 상대적인 방법으로 처리 가능하며 기존 인스턴스를 변환하지 않도록 새로운 인스턴스가 생성된다.
- TemporalAdjuster를 이용하면 단순히 값을 바꾸는 것 이상의 복잡한 동작을 수행할 수 있으며 자신만의 커스텀 날짜 변환 기능을 정의할 수 있다.
- 날짜와 시간 객체를 특정 포맷으로 출력하고 파싱하는 포매터를 정의 가능, 포매터는 스레드 안전성을 보장
- 특정 지역/장소 상대적인 시간대 또는 UTC/GMT 기준의 오프셋을 이용해서 시간대를 정의 가능
- ISO-8601 표준 시스템을 준수하지 않는 캘리더 시스템도 사용할 수 있다.
