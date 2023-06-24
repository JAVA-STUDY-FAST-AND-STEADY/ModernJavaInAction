# 새로운 날짜와 시간 API

- [x] 1. LocalDate, LocalTime, Instant, Duration, Period 클래스
- [x] 2. 날짜 조정, 파싱, 포메팅
- [x] 3. 다양한 시간대와 캘린더 활용 방법
- [x] 4. 마치며
- [x] 5. 배우고 느낀점

<br><br>

## 1. LocalDate, LocalTime, Instant, Duration, Period 클래스

---

<br>

 java.time 패키지에는 LocalData, LocalTime, LocalDateTime, Instant, Duration, Period 등 새로운 클래스를 제공한다.

<br>

### 1.1 LocalDate와 LocalTime 사용

<br>

 새로운 날짜와 시간 API를 사용할 때, 처음 접하게 되는 것이 LocalDate다. LocalDate 인스턴스는 시간을 제외한 날짜를 표현하는 불변 객체다. 특히 LocalDate 객체는 어떤 시간대 정보도 포함하지 않는다. 정적 팩토리 메서드 of로 LocalDate 인스턴스를 만들 수 잇다.

```java
LocalDate date = LocalDate.of(2023, 6, 21); // 2023-06-21
int year = date.getYear(); // 2023
Month month = date.getMonth(); // JUNE
int day = date.getDayOfMonth(); // 21
DayOfWeek dow = date.getDayOfWeek(); // WEDNESDAY
int len = date.lengthOfMonth(); // 30 마지막 일수
boolean leap = date.isLeapYear(); // 윤년인지 아닌지
```

 팩토리 메서드 now는 시스템 시계의 정보를 이용해서 현재 날짜 정보를 얻는다.

```java
LocalDate today = LocalDate.now();
```

 get 메서드에 TemporalField를 전달해서 정보를 얻는 방법도 있다. TemporalField는 시간 관련 객체에서 어떤 필드의 값에 전달하지 정의하는 인터페이스다. 열거자 ChronoField는 TemporalField 인터페이스를 정의하므로 다음 코드에서 보여주는 것처럼 ChronoFiled의 열거자 요소를 이용해서 원하는 정보를 쉽게 얻을 수 있다.

 TemporalField를 이용해서 LocalDate값 읽기

```java
int year = date.get(ChronoField.YEAR);
int month = date.get(ChronoField.MONTH_OF_YEAR);
int day = date.get(ChronoField.DAY_OF_MONTH);

// 내장 매서드를 이용해 가독성을 높일 수 있음
int year = date.getYear();
int month = date.getMonthValue();
int day = date.getDayOfMonth();
```

 LocalTime 클래스 만들고, 값 읽기

```java
LocalTime time = LocalTime.of(13, 45, 20); // 13:45:20
int hour = time.getHour(); // 13
int minute = time.getMinute(); // 45
int second = time.getSecond(); // 20
```

 날짜 문자열과 시간 문자열로 LocalDate와 LocalTime의 인스턴스를 만들 수 있다.

```java
LocalDate date = LocalDate.parse("2017-09-21");
LocalTime time = LocalTime.parse("13:45:20");

// 만일 문자열이 형식에 맞지 않는다면, DateTimeParseException을 발생시킨다.
```

<br>

### 1.2 날짜와 시간 조합

<br>

 LocalDateTime은 LocalDate와 LocalTime을 둘다 가지는 복합 클래스이다. 시간과 날짜를 모두 표현할 수 있으며,직접 만들거나 날짜와 시간을 조합하는 방법도 있다.

```java
// 2017-09-21T13:45:20
LocalDateTime dt1 = LocalDateTime.of(2017, Month.SEPTEMBER, 21, 13, 45, 20);
LocalDateTime dt2 = LocalDateTime.of(date, time);
LocalDateTime dt3 = date.atTime(13, 45, 20);
LocalDateTime dt4 = date.atTime(time);
LocalDateTime dt5 = time.atDate(date);

// LocalDateTime에서 LocalDate와 LocalTime 인스턴스를 toLocalDate, toLocalTime으로 추출할 수 있음
LocalDate ld = dt1.toLocalDate();
LocalTime lt = dt1.toLocalTime();
```

<br>

### 1.3 Instant 클래스 : 기계의 날짜와 시간

<br>

 사람은 보통 주, 날짜, 시간, 분, 초로 날짜와 시간을 계산한다. 하지만 기계는 **연속된 시간에서 특정 지점을 하나의 큰 수로 표현하는 것이 가장 자연스러운 시간 표현 방법**(Epoch, 1970년 1월 1일 00시 00분 00초 부터 현재까지의 누적된 초)이다.

 java.util.instant 클래스에서 유닉스 에포크 시간을 기준으로 특정 지점까지 시가을 초로 표현한다.

 팩토리 메서드 ofEpochSecond에 초를 넘겨줘서 Instant 클래스 인스턴스를 만들 수 있다. 또한 정밀하게 나노초로 계산이 가능하다.

```java
Instant.ofEpochSecond(3);
Instant.ofEpochSecond(3, 0);
Instant.ofEpochSecond(2, 1_000_000_000); // 2초 이후 1억 나노초
Instant.ofEpochSecond(4, -1_000_000_000); // 4초 이전 1억 나노초
```

 또한 기계가 보기 편한 시간을 사람도 볼 수 있게 정적 팩토리 메서드 now를 제공한다. 하지만 기계 전용 유틸리티라는 걸 명심하자.

<br>

### 1.4 Duration과 Period 정의

<br>

 지금까지의 클래스는 Temporal 인터페이스를 구현한다. Temporal 인터페이스는 특정 시간을 모델링하는 객체의 값을 어떻게 읽고 조작할지 정의한다.

 두 시간 객체 사이의 지속시간 duration을 만들 수 있다. Duration 클래스의 정적 팩토리 메서드 between으로 두 시간 객체 사이의 지속시간을 만들 수 있다.

```java
Duration d1 = Duration.between(time1, time2);
Duration d1 = Duration.between(dateTime1, dateTime2);
Duration d2 = Duration.between(instant1, instant2);
```

 LocalDateTime과 Instant는 사람과 기계 사용하는 역할이 다르므로, 혼합할 수 없다. 또한 **Duration은 초와 나노초로 시간 단위를 표현하므로 between 메서드에 LocalDate를 전달할 수 없다. 년, 월, 일로 시간을 표현할 때는 period 클래스를 사용한다.**

```java
Period tenDays = Period.between(LocalDate.of(2017, 9, 11), LocalDate.of(2017,9, 21));
```

 Duration과 Period 만들기

```java
Duration threeMinutes = Duration.ofMinutes(3);
Duration threeMinutes = Duration.of(3, ChronoUnit.MINUTES);

Period tenDays = Period.ofDays(10);
Period threeWeeks = Period.ofWeeks(3);
Period twoYearsSixMonthsOneDay = Period.of(2, 6, 1);
```

#### Duration과 Period가 공통으로 제공하는 메서드

|메서드|정적|설명                                                |
|----|---|---------------------------------------------------|
|between|O|두 시간 사이의 간격을 생성함|
|from|O|시간 단위로 간격을 생성함|
|of|O|주어진 구성 요소에서 간격 인스턴스를 생성함|
|parse|O|문자열을 파싱해서 간격 인스턴스를 생성함|
|addTo|X|현재값의 복사본을 생성한 다음에 지정된 Temporal 객체에 추가함|
|get|X|현재 간격 정보값을 읽음|
|isNegative|X|간격이 음수인지 확인함|
|isZero|X|간격이 0인지 확인함|
|minus|X|현재값에서 주어진 시간을 뺀 복사본을 생성함|
|multipliedBy|X|현재값에 주어진 값을 곱한 복사본을 생성함|
|negated|X|주어진 값을 부호를 반전한 복사본을 생성함|
|plus|X|현재 값에 주어진 시간을 더한 복사본을 생성함|
|subtractFrom|X|지정된 Temporal 객체에서 간격을 뺌|

 지금까지 살펴본 모든 클래스는 **불변**이다. 따라서 함수형 프로그래밍, 스레드 안정성, 도메인 모델의 일관성을 유지하는 데 좋은 특징이다.

<br><br>

## 2. 날짜 조정, 파싱, 포매팅

---

<br>

 withArttribute(with + "") 메서드로 기존의 LocalDate를 바꾼 버전을 직접 간단하게 만들 수 있다. **with 메서드는 Temporal 객체를 바꾸는 것이 아닌, 필드를 갱신한 복사본을 만든다. 이것을 함수형 갱신이라 한다.**

 절대적인 방식으로 LocalDate의 속성 바꾸기

```java
LocalDate date1 = LocalDate.of(2017, 9, 21); // 2017-09-21
LocalDate date2 = date1.withYear(2011); // 2011-09-21
LocalDate date3 = date2.withDayofMonth(25); // 2011-09-25
LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR, 2); // 2011-02-25
```

 선언형으로 LocalDate으로 사용하는 방법도 존재한다. 

 상대적인 방식으로 LocalDate 속성 바꾸기

```java
LocalDate date1 = LocalDate.of(2017, 9, 21); // 2017-09-21
LocalDate date2 = date1.plusWeek(1); // 2017-09-28
LocalDate date3 = date2.minusYears(6); // 2011-09-28
LocalDate date4 = date3.plus(6, ChronoUnit.MONTH); // 2012-03-28
```

#### 특정 시점을 표현하는 날짜 시간 클래스의 공통 메서드

|메서드|정적|설명                                             |
|----|---|------------------------------------------------|
|from|O|주어진 Temporal 객체를 이용해서 클래스의 인스턴스를 생성함|
|now|O|시스템 시계로 Temporal 객체를 생성함|
|of|O|주어진 구성 요소에서 Temporal 객체의 인스턴스를 생성함|
|parse|O|문자열을 파싱해서 Temporal 객체를 생성함|
|atOffset|X|시간대 오프셋과 Temporal 객체를 합침|
|atZone|X|시간대 오프셋과 Temporal 객체를 합침|
|format|X|지정된 포매터를 이용해서 Temporal 객체를 문자열로 변환함(Instant는 지원하지 않음)|
|get|X|Temporal 객체의 상태를 읽음|
|minus|X|특정 시간을 뺀 Temporal 객체의 복사본을 생성|
|plus|X|특정 시간을 더한 Temporal 객체의 복사본을 생성|
|with|X|일부 상태를 바꾼 Temporal 객체의 복사본을 생성|

<br>

### 2.1 TemporalAdjusters 사용하기

<br>

 조금 더 복잡한 날짜 조정 기능을 필요로할 때, 오버로드된 버전의 with 메서드에 좀 더 다양한 동작을 수행할 수 있도록 기능을 제공하는 TemporalAdjuster를 전달하는 방법으로 문제를 해결할 수 있다.

 **TemporalAdjuster는 인터페이스이며, TemporalAdjusters는 정적 팰토리 메서드를 포함하는 클래스이다.** 

 미리 정의된 TemporalAdjusters 사용하기

```java
import static java.time.temporal.TemporalAdjusters.*;
LocalDate date1 = LocalDate.of(2014, 3, 18);
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY));
LocalDate date3 = date2.with(lastDayOfMonth());
```

#### TemPoralAdjusters 클래스의 팩토리 메서드

|메서드              |설명                                     |
|------------------|----------------------------------------|
|dayOfWeekInMonth(int ordinal, DayOfWeek dayOfWeek)|주 단위로 월 내 특정 요일을 지정된 순서로 찾아갑니다.|
|firstDayOfMonth()|현재 월의 첫 번째 날짜를 반환합니다.|
|firstDayOfNextMonth()|다음 달의 첫 번째 날짜를 반환합니다.|
|firstDayOfNextYear()|다음 해의 첫 번째 날짜를 반환합니다.|
|firstDayOfYear()|현재 해의 첫 번째 날짜를 반환합니다.|
|firstInMonth(DayOfWeek dayOfWeek)|현재 월의 첫 번째 지정된 요일의 날짜를 반환합니다.|
|lastDayOfMonth()|현재 월의 마지막 날짜를 반환합니다.|
|lastDayOfNextMonth()|다음 달의 마지막 날짜를 반환합니다.|
|lastDayOfNextYear()|다음 해의 마지막 날짜를 반환합니다.|
|lastDayOfYear()|현재 해의 마지막 날짜를 반환합니다.|
|lastInMonth(DayOfWeek dayOfWeek)|현재 월의 마지막 지정된 요일의 날짜를 반환합니다.|
|next(DayOfWeek dayOfWeek)|다음 지정된 요일의 날짜를 반환합니다.|
|nextOrSame(DayOfWeek dayOfWeek)|다음 지정된 요일의 날짜를 반환하거나 현재 지정된 요일의 날짜를 반환합니다.|
|previous(DayOfWeek dayOfWeek)|이전 지정된 요일의 날짜를 반환합니다.|
|previousOrSame(DayOfWeek dayOfWeek)|이전 지정된 요일의 날짜를 반환하거나 현재 지정된 요일의 날짜를 반환합니다.|

<br>

### 2.2 날짜와 시간 객체 출력과 파싱

<br>

 날짜와 시간 관련 작업에서 포매팅과 파싱은 떨어질 수 없는 관계이다. 포매팅과 파싱 전용 패키지인 java.time.format이 새로추가되었다. 이 패키지에서 가장 중요한 클래스는 DateTimeFormatter로 정적 메서드와 상수를 이용해 쉽게 포매터를 구할 수 있다.

 다음의 예제를 보며 이해해보자

```java
LocalDate date = LocalDate.of(2014, 3, 18);
String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE); // 20140318
String s2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE); // 2014-03-18
```

 반대로 날짜나 시간을 표현하는 문자열을 파싱해서 날짜 객체를 다시 만들 수 있다.

```java
LocalDate date1 = LocalDate.parse("20140318", DateTimeFormatter.BASIC_ISO_DATE);
LocalDate date2 = LocalDate.parse("2014-03-18", DateTimeFormatter.ISO_LOCAL_DATE);
```

 기존의 java.util.DateFormat 클래스와 달리 모든 DateTimeFormatter는 스레드에서 안전하게 사용할 수 있는 클래스이다.

 특정 패턴으로 포매터를 만들 수 있는 정적 팩토리 메서드이다.

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate date1 = LocalDate.of(2014, 3, 18);
String formattedDate = date1.format(formatter);
LocalDate date2 = LocalDate.parse(formattedDate, formatter);
```

 또한 지역에서 사용하는 날짜인 지역화 DateTimeFormatter를 만들 수 있다.

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy", Locale.ITALIAN);
LocalDate date1 = LocalDate.of(2014, 3, 18);
String formattedDate = date1.format(formatter); // 18. marzo 2014
LocalDate date2 = LocalDate.parse(formattedDate, formatter);
```

 DateTimeFormatterBuilder 클래스로 보합적인 포매터를 정의해서 좀 더 세부적으로 포매터를 제어할 수 있다.

 DateTimeFormatter 만들기

```java
DateTimeFormatter italianFormatter = new DateTimeFormatterBuilder()
            .appendText(ChronoField.DAY_OF_MONTH)
            .appendLiteral(". ")
            .appendText(ChronoField.MONTH_OF_YEAR)
            .appendLiteral(" ")
            .appendText(ChronoField.YEAR)
            .parseCaseInsensitive()
            .toFormatter(Locale.ITALIAN);
```

<br><br>

## 3. 다양한 시간대와 캘린더 활용 방법

---

<br>

 지금까지 살펴본 모든 클래스에는 시간대와 관련한 정보가 없었다. 새로운 날짜와 시간 API는 시간대를 간단하게 처리할 수 있다. 기존의 TimeZone을 대체하는 ZoneId 클래스가 새롭게 등장했으며, 새로운 클래스를 이용하면 서머 타입(DST)와 같은 복잡한 사항이 자동으로 처리된다. (불변 클래스다.)

<br>

### 3.1 시간대 사용하기

<br>

 표준 시간이 같은 지여을 묶어서 **시간대** 규칙 집합을 정의한다. ZoneRules 클래스에는 약 40개 정도의 시간대가 있다. ZoneId의 getRules()를 이용해서 해당 시간대의 규정을 획득할 수 있다. 다음의 예시를 보자

```java
ZoneId romeZone = ZoneId.of("Europe/Rome");
```

 지역 ID는 "{지역}/{도시}" 형식으로 이루어지며 IANA Time Zone Database에서 제공하는 지역 집합 정보를 사용한다.

 특정 시점에 시간대 적용

```java
LocalDate date = LocalDate.of(2014, Month.MARCH, 18);
ZonedDateTime zdt1 = date.atStartOfDay(romeZone);
LocalDateTime dateTime = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45);
ZonedDateTime zdt2 = dateTime.atZone(romeZone);
Instant instant = Instant.now();
ZonedDateTime zdt3 = instant.atZone(romeZone);
```

 ZoneDateTime은 LocalDate, LocalTime, ZoneId를 컴포넌트로 가진다.

<br>

### 3.2 UTC/Greenwich 기준의 고정 오프셋

<br>

 때로는 UTC/GMT를 기준으로 시간대를 표현하기도 한다. ZoneId의 서브 클래스인 ZoneOffset 클래스로 런던의 그리니치 0도 자오선과 시간값의 차이를 표현할 수 있다.

```java
ZoneOffset newYorkOffset = ZoneOffset.of("-05:00");
```

 하지만 이 방법은 권장하지 않는다. 서머 타임을 제대로 처리할 수 없기 때문이다. ISO-8601 캘린터 시스템에서 정의하는 UTC/GMT와 오프셋으로 날짜와 시간을 표현하는 OffsetDateTime을 만드는 방법도 있다.

```java
LocalDateTime dateTime = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45);
OffsetDateTime dateTimeInNewYork = OffsetDateTime.of(date, newYorkOffset);
```

 새로운 날짜와 시간 API는 ISO 캘린더 시스템에 기반하지 않은 정보도 처리할 수 있는 기능을 가졌다.

<br>

### 3.3 대안 캘린더 시스템 사용하기

<br>

 ISO-8601 캘린더 시스템은 실질적으로 전 세계에서 통용된다. 하지만 자바 8에서는 추가로 4개의 캘린더 시스템을 제공한다. ThaiBuddhistDate, MinguoDate, JapanesDate, HijrahDate 4개의 클래스가 각각의 캘린더 시스템을 대표한다. 위 4개의 클래스와 LocalDate 클래스는 ChronoLocalDate 인터페이스를 구현하는데, ChronoLocalDate는 임의의 연대기에서 특정 날짜를 표현할 수 있는 기능을 제공하는 인터페이스이다. LocalDate를 이용해 이들 4개의 클래스 중 하나의 인스턴스를 만들 수 있다.

```java
LocalDate date = LocalDate.of(2014, Month.MARCH, 18);
JapaneseDate japaneseDate = JapaneseDate.from(date);
```

 특정 Locale과 Locale에 대한 날짜 인스턴스로 캘린더 시스템을 만드는 방법도 있다. 새로운 날짜와 시간 API에서 Chronology는 캘린더 시스템을 의미하며, 정적 팩토리 메서드 ofLocale을 이용해 Chronology의 인스턴스를 획득할 수 있다.

```java
Chronology japaneseChronology = Chronology.ofLocale(Locale.JAPAN);
ChronoLocalDate now = japaneseChronology.dateNow();
```

 날짜와 시간 API 설계자는 ChronoLocalDate보다 LocalDate를 사용하라고 권고한다.

<br><br>

## 4. 마치며

---

<br>

 - 자바 8 이전 버전에서 제공하는 기존의 java.util.Date 클래스와 관련 클래스에서는 여러 불일치점들과 가변성, 어설픈 오프셋, 기본값, 잘못된 이름 결정 등의 설계 결함이 존재했다.

 - 새로운 날짜와 시간 API에서 날짜와 시간 객체는 모둔 불변이다.

 - 새로운 API는 각각 사람과 기계가 편리하게 날짜와 시간 정보를 관리할 수 있도록 두 가지 표현 방식을 제공한다.

 - 날짜와 시간 객체를 절대적인 방법과 상대적인 방법으로 처리할 수 있으며, 기존 인스턴스를 변환하지 않도록 처리 결과로 새로운 인스턴스가 생성된다.

 - TemporalAdjuster를 이용하면 단순히 값을 바꾸는 것 이상의 복잡한 동작을 수행할 수 있으며, 자신만의 커스텀 날짜 변환 기능을 정의할 수 있다.

 - 날짜와 시간 객체를 특정 포맷으로 출력하고 파싱하는 포매터를 정의할 수 있다. 패턴을 이용하거나 프로그램을 포매터를 만들 수 있으며, 포매터는 스레드 안정성을 보장한다.

 - 특정 지역/장소에 상대적인 시간대 또는 UTC/GMT 기준의 오프셋을  이용해서 시간대를 정의할 수 있으며, 이 시간대를 날짜와 시간 객체에 적용해서 지역화 할 수 있다.

 - ISO-8601 표준 시스템을 준수하지 않는 캘린더 시스템도 사용할 수 있다.


<br><br>

## 5. 배우고 느낀점

---

<br>

<br><br>
