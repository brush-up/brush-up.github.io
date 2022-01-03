---
title:  "[java] 시간/날짜 관련 정리"
excerpt: "Java 시간/날짜 관련 정리"

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 20202-01-03
last_modified_at: 2022-01-03
---



## 기본 용어
*  에포크 (Epoch) : 시간 기원
    * 중요한 사건,변화가 일어난 정적인 시간 기원
        * Unix Epoch  : 1970년 1월 1일 새벽 0시 (유닉스 출현 기원)
        * Prime Epoch : 1900년 1월 1일 새벽 0시 (NTP 프로토콜에서 사용하는 기원)
        * GPS Epoch   : 1980년 1월 6일 일요일 0시 0분 0초 (GPS 시간 기원)
* GMT
    * GMT는 Greenwich Mean Time의 약자로서 경도 0도에 위치한 영국 그리니치 천문대를 기준으로 하는 태양 시간을 의미한다. GMT 시간은 1925년 2월 5일부터  1972년 1월 1일까지 세계 표준시로 사용되었다. 한국의 타임존은 GMT+09:00 이다. 
* UTC
    * UTC는 지구 자전주기의 흐름이 늦어지고 있는 문제를 해결하기 위해 1972년에 세슘 원자의 진동수에 기반한 국제 원자시를 기준으로 다시 지정된 시간대이다. 즉, UTC는 좀더 정확한 시간측정을 위해서 GMT를 대체하기 위해 제정된 새로운 표준이며, 시간적으로는 둘 사이에 아주 미세한 차이가 있긴하다.
* OFFSET
    * UTC+09:00 에서 +09:00 의 의미는 UTC의 기준시간보다 9시간이 빠르다는 의미이다. 즉 UTC 기준으로 현재 낮 12시라면 한국시간으로는 오후 9시가 될 것이다. 이렇게 UTC와의 차이를 나타낸 것을 오프셋이라고 하며, +09:00 혹은 -03:00 등과 같이 표현된다.


## 차이는?
* LocalDateTime 
    * ISO-8601 타임존 개념이 없음.
    * 설치된 os에서 타임존에 맞춘 시간을 의미 (우리나라의 현재 시각이 리턴됨)
    * ex
        * 2012-12-05T10:10:10
* ZonedDateTime
    * 타임존이 추가됨
    * ex
        * 2012-12-05T10:10:10+09:00 Asia/Seoul
* OffsetDateTime 
    * UTC의 offset만 표현
    * ex
        * 2012-12-05T10:10:10+09:00



* Date와 OffsetDateTime의 차이는
    * OffsetDateTime는  java.util.Date에 대한 최신 대안으로 JDK 8에 도입
    * OffsetDateTime는 thread-safe하고 nanoseconds 단위로 저장하지만 Date는 thread-safe하지 않으며 시간을 millisecond 단위로 저장
    * OffsetDateTime 는 값 기반 클래스이므로 비교할때 == 대신 equals 를 사용해야 함.
    * 각각의 toString 결과는 아래와 같음
        * Date: Sat Oct 19 17:12:30 2019 (비표준 형식)
        * OffsetDateTime: 2019-10-19T17:12:30.174Z   (ISO-8601 형식)
    * Date 를 OffsetDateTime 로 변환하기
    ```java
    //UTC 시간대로 변환하기
    Date date = new Date();
    OffsetDateTime offsetDateTime = date.toInstant().atOffset(ZoneOffset.UTC);
    // +3:30(테헤란 시간)으로 변환하기.
    int hour = 3;
    int minute = 30;
    offsetDateTime = date.toInstant().atOffset(ZoneOffset.ofHoursMinutes(hour, minute));
    ```

* 아래처러 정리가됨
```
|---------------------ZonedDateTime-----------------------|
|------------OffsetDateTime----------------|
|--------LocalDateTime------|
|--LocalDate--|--LocalTime--|--ZoneOffSet--|--ZoneRegion--|
  2021-12-24      10:10:12       +09:00       Asiz/Seoul 
```
.


## 변환은?
* LocalDateTime
    * LocalDateTime객체에 atOffset(ZoneOffset offset) 메소드를 사용하면 OffsetDateTime으로 변환할 수 있다. ZoneOffset은 of(int hours) 메소드를 사용해서 만들수 있고.
    * LocalDateTime객체에 atZone(ZoneId zone) 메소드를 사용하면 ZonedDateTime으로 변환할 수 있다. ZoneId는 of(String zoneId) 가 되는데, 지정된 String들이 있다.
* ZonedDatetime
    * toLocalDateTime()으로 LocalDateTime을 얻을 수 있다.
    * toOffsetDateTime()으로 OffsetDateTime을 얻을 수 있다.
* OffsetDateTime
    * toLocalDateTime()으로 LocalDateTime을 얻을 수 있다
    * atZoneSameInstant(ZoneId zone)으로 zone정보를 붙여 ZonedDateTime을 줄 수 있다.
*  Java에서 시간을 Unix-epoch 밀리초로 변환하는 여러 방법
    *  Core java
        *  1. Using Date
        ```java
        long millis = 1556175797428L; // April 25, 2019 7:03:17.428 UTC
        Date date = // implementation details
        Assert.assertEquals(millis, date.getTime());
        //getTime() 메서드 를 호출하여 날짜 를 밀리초로 변환
        ```
        * 2. Using Calendar
        ```java
        Calendar calendar = // implementation details
        Assert.assertEquals(millis, calendar.getTimeInMillis());
        //Calendar 객체 가 있으면 getTimeInMillis() 메서드를 사용
        ```
    * Java 8 Date Time API
        * 1. Using Instant
            * 간단히 말해서 Instant 는 Java의 epoch 타임라인의 한 지점, Instant 에서 현재 시간을 밀리초 단위로 가져올 수 있습니다 .
        ```java
        java.time.Instant instant = // implementation details
        Assert.assertEquals(millis, instant.toEpochMilli());
        //toEpochMilli() 메서드는 앞에서 정의한 것과 동일한 수의 밀리초를 반환합니다
        ```
        * 2. Using LocalDateTime
            *  Java 8's Date and Time API to convert a LocalDateTime into milliseconds:
        ```java
        LocalDateTime localDateTime = // implementation details
        ZonedDateTime zdt = ZonedDateTime.of(localDateTime, ZoneId.systemDefault());
        Assert.assertEquals(millis, zdt.toInstant().toEpochMilli());
        //먼저 현재 날짜의 인스턴스를 만들었습니다. 그런 다음 toEpochMilli() 메서드를 사용하여 ZonedDateTime 을 밀리초로 변환했습니다 .아시다시피 LocalDateTime 에는 시간대에 대한 정보가 포함되어 있지 않습니다. 즉, LocalDateTime 인스턴스 에서 직접 밀리초를 가져올 수 없습니다 .
        ```
        
        
## 참고
* 시간 변환은 아래사이트에서 해보자.
    * https://www.epochconverter.com/

    
## 예시
* Epoch time to LocalDate
```java
LocalDate ld = Instant.ofEpochMilli(epoch).atZone(ZoneId.systemDefault()).toLocalDate();
```
* Epoch time to LocalDateTime

```java
LocalDateTime ldt = Instant.ofEpochMilli(epoch).atZone(ZoneId.systemDefault()).toLocalDateTime();
```
* 예시
```java
long epoch = Instant.now().toEpochMilli();
System.out.println(epoch);

LocalDate ld = Instant.ofEpochMilli(epoch)
        .atZone(ZoneId.systemDefault()).toLocalDate();
System.out.println(ld);

LocalDateTime ldt = Instant.ofEpochMilli(epoch)
        .atZone(ZoneId.systemDefault()).toLocalDateTime();
System.out.println(ldt);
```
* Output
```
1581420629955
2020-02-11
2020-02-11T19:30:29.955
```

## 심화 예시

* 2019-01-07T23:59:59+09:00 를 입력받아서..

```java
@GetMapping(value = "/sample")
public SampleResponse getTest(
	@RequestParam(required = false, defaultValue = "#{T(java.time.OffsetDateTime).now().minusMonths(3)}") @DateTimeFormat(iso = DateTimeFormat.ISO.DATE_TIME) OffsetDateTime afterDate) 
{
	.
	sampleService.getSampleList(afterDate);
	.
}
.
.
void getSampleList(OffsetDateTime afterDate) {
	LocalDateTime afterDateTime = DateUtils.getLocalDateTime(afterDate.toEpochSecond() * 1000);
	List<Sample> sampleList = sampleMapper.selectSampleAfterDate(DateUtils.getDate(afterDateTime));
}
.
.
public static Date getDate(LocalDate localDate) {
	return Date.from(localDate.atStartOfDay(ZoneId.systemDefault()).toInstant());
}
public static Date getDate(LocalDateTime localDateTime) {
	return Date.from(localDateTime.atZone(ZoneId.systemDefault()).toInstant());
}
public static Date getDate(LocalDateTime localDateTime, ZoneOffset offset) {
	return Date.from(localDateTime.toInstant(offset));
}
public static Date getEndDate(LocalDate localDate) {
	LocalDateTime localDateTime = LocalDateTime.of(localDate, LocalTime.MAX).withNano(999999000);
	
	return getDate(localDateTime);
}
.
.
List<Sample> selectSampleAfterDate(@Param("afterDate") Date afterDate);
```

## 참고
* https://www.baeldung.com/java-zoneddatetime-offsetdatetime
* https://www.baeldung.com/java-convert-date-to-offsetdatetime
* http://www.ktword.co.kr/word/abbr_view.php?nav=2&id=525&m_temp1=2706
* https://meetup.toast.com/posts/125
* https://www.baeldung.com/java-time-milliseconds

