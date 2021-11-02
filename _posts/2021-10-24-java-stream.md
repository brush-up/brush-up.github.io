---
title:  "[java][작성중]stream 간략 정리"
excerpt: "stream 간략 정리를 해보자. "

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 2021-10-24
last_modified_at: 2021-10-24
---

## intro
 Java 8 기능 중 하나인 Java stream(스트림)에 대해 간략하게 알아 보자
 

## Stream API
* Java 8의 주요 새 기능 중 하나는 요소들의 시퀀스 처리하기 위한 클래스를 포함 하는 스트림 기능인 java.util.stream 의 도입입니다 .
* central API 클래스는 Stream< T > 입니다..
 

### Stream 생성
* 아래처럼 스트림은 stream() 및 of() 메서드, builder를 사용하여 컬렉션 또는 배열과 같은 다양하게 생성할 수 있습니다 .
```java
String[] arr = new String[]{"a", "b", "c"};
Stream<String> stream = Arrays.stream(arr);
stream = Stream.of("a", "b", "c");
.
.
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();
Stream<String> parallelStream = list.parallelStream(); // 병렬 처리 스트림
.
.
Stream<String> builderStream = 
  Stream.<String>builder()
    .add("Eric").add("Elena").add("Java")
    .build(); // [Eric, Elena, Java]
```

* stream() 기본 메서드가 Collection 인터페이스에 추가되고 모든 컬렉션을 요소 소스로 사용하여 Stream< T >를 만들 수 있습니다.
```java
Stream<String> stream = list.stream();
```


### Streams을 사용한 멀티스레딩
* Stream API는 또한 병렬 모드에서 스트림의 요소에 대한 작업을 실행 하는 parallelStream() 메서드를 제공하여 멀티스레딩을 단순화 합니다.
* 아래 코드 는 스트림의 모든 요소에 대해 doWork() 메서드 를 병렬 로 실행할 수 있습니다 .
```java
list.parallelStream().forEach(element -> doWork(element));
```

### 기타
* 파일 스트림
    * 자바 NIO 의 Files 클래스의 lines 메소드는 해당 파일의 각 라인을 스트링 타입의 스트림으로 만들어줍니다.
    ```java
    Stream<String> lineStream = 
  Files.lines(Paths.get("file.txt"), 
              Charset.forName("UTF-8"));
    ```
* 문자열 스트링
    * 다음은 정규표현식(RegEx)을 이용해서 문자열을 자르고, 각 요소들로 스트림을 만든 예제입니다.
    ```java
    Stream<String> stringStream = 
  Pattern.compile(", ").splitAsStream("Eric, Elena, Java");
  // [Eric, Elena, Java]
    ```
* 병렬 스트림 Parallel Stream
    * 스트림 생성 시 사용하는 stream 대신 parallelStream 메소드를 사용해서 병렬 스트림을 쉽게 생성할 수 있습니다. 내부적으로는 쓰레드를 처리하기 위해 자바 7부터 도입된 Fork/Join framework 를 사용합니다.
        * https://docs.oracle.com/javase/tutorial/essential/concurrency/forkjoin.html
    ```java
    // 병렬 스트림 생성
    Stream<Product> parallelStream = productList.parallelStream();
    // 병렬 여부 확인
    boolean isParallel = parallelStream.isParallel();
    // 따라서 다음 코드는 각 작업을 쓰레드를 이용해 병렬 처리됩
    boolean isMany = parallelStream
  .map(product -> product.getAmount() * 10)
  .anyMatch(amount -> amount > 200);
    ```
* 다시 시퀀셜(sequential) 모드로 돌리고 싶다면 다음처럼 sequential 메소드를 사용합니다. 뒤에서 한번 더 다루겠지만 반드시 병렬 스트림이 좋은 것은 아닙니다.
```java
IntStream intStream = intStream.sequential();
boolean isParallel = intStream.isParallel();
```

## 스트림 작업
* 스트림에서 수행할 수 있는 유용한 작업이 많이 있습니다.
* 중간 작업 (return Stream<T> )과 터미널 작업 (확정 유형의 결과 반환 ) 으로 나뉩니다 . 중간 작업은 연결을 허용합니다.
* 스트림에 대한 작업이 소스를 변경하지 않는다는 점도 주목할 가치가 있습니다.

* 다음은 간단한 예입니다.
```java
long count = list.stream().distinct().count();
```

* 따라서 distinct() 메서드는 이전 스트림의 고유 요소로 구성된 새 스트림을 생성하는 중간 작업을 나타냅니다. 그리고 count() 메서드는 스트림의 크기를 반환하는 터미널 작업입니다.


### Iterating
* 스트림 API는 for-each 및 while 루프를 대체하는 데 도움이 됩니다. 작업 논리에 집중할 수 있지만 요소 시퀀스에 대한 반복에는 집중할 수 없습니다. 예를 들면:
```java
for (String string : list) {
    if (string.contains("a")) {
        return true;
    }
}
```

이 코드는 Java 8 코드 한 줄로 변경할 수 있습니다.

```java
boolean isExist = list.stream().anyMatch(element -> element.contains("a"));
```


### Filtering
* filter() 메서드를 사용하면 술어를 충족하는 요소 스트림을 선택할 수 있습니다.
(The filter() method allows us to pick a stream of elements that satisfy a predicate.)

예를 들어 다음 목록을 고려하십시오.

```java
ArrayList<String> list = new ArrayList<>();
list.add("One");
list.add("OneAndOnly");
list.add("Derek");
list.add("Change");
list.add("factory");
list.add("justBefore");
list.add("Italy");
list.add("Italy");
list.add("Thursday");
list.add("");
list.add("");
```

* 다음 코드는 List<String>의 Stream<String>을 만들고 char "d"를 포함하는 이 스트림의 모든 요소를 찾고 필터링된 요소만 포함하는 새 스트림을 만듭니다.

```java
Stream<String> stream = list.stream().filter(element -> element.contains("d"));
```

### Sorting
* 정렬의 방법은 다른 정렬과 마찬가지로 Comparator 를 이용
* 인자 없이 그냥 호출할 경우 오름차순으로 정렬합니다.
* boxed 는 형변환 비슷한것인듯. (int, long, double 요소를 Integer, Long, Double로 박싱한다고..)
```java
IntStream.of(14, 11, 20, 39, 23)
  .sorted()
  .boxed()
  .collect(Collectors.toList());
// [11, 14, 20, 23, 39]
```
* 인자를 넘기는 경우와 비교해보겠습니다. 스트링 리스트에서 알파벳 순으로 정렬한 코드와 Comparator 를 넘겨서 역순으로 정렬한 코드입니다.

```java
List<String> lang = 
  Arrays.asList("Java", "Scala", "Groovy", "Python", "Go", "Swift");

lang.stream()
  .sorted()
  .collect(Collectors.toList());
// [Go, Groovy, Java, Python, Scala, Swift]

lang.stream()
  .sorted(Comparator.reverseOrder())
  .collect(Collectors.toList());
// [Swift, Scala, Python, Java, Groovy, Go]
```

* 문자열 길이를 기준으로 정렬
```java
lang.stream()
  .sorted(Comparator.comparingInt(String::length))
  .collect(Collectors.toList());
// [Go, Java, Scala, Swift, Groovy, Python]

lang.stream()
  .sorted((s1, s2) -> s2.length() - s1.length())
  .collect(Collectors.toList());
// [Groovy, Python, Scala, Swift, Java, Go]
```


### Mapping
* 특수 기능을 적용 하여 Stream 요소를 변환 하고 이러한 새 요소를 Stream 으로 수집 하려면 map() 메서드를 사용할 수 있습니다 .
    * 맵(map)은 스트림 내 요소들을 하나씩 특정 값으로 변환해줍니다. 이 때 값을 변환하기 위한 람다를 인자로 받습니다.

```java
List<String> uris = new ArrayList<>();
uris.add("C:\\My.txt");
Stream<Path> stream = uris.stream().map(uri -> Paths.get(uri));
```

* 따라서 위의 코드 는 초기 Stream 의 모든 요소에 특정 람다 식을 적용하여 Stream<String> 을 Stream<Path> 로 변환 합니다.
* 모든 요소에 고유한 요소 시퀀스가 ​​포함된 스트림이 있고 이러한 내부 요소의 스트림을 생성하려면 flatMap() 메서드를 사용해야 합니다 .

```java
List<Detail> details = new ArrayList<>();
details.add(new Detail());
Stream<String> stream
  = details.stream().flatMap(detail -> detail.getParts().stream());
```

* 이 예에는 Detail 유형의 요소 목록이 있습니다. Detail 클래스에는 List<String>인 PARTS 필드가 있습니다. flatMap() 메서드의 도움으로 PARTS 필드의 모든 요소가 추출되어 새 결과 스트림에 추가됩니다. 그 후에는 초기 Stream<Detail>이 손실됩니다.


### Matching

* Stream API는 일부 술어에 따라 시퀀스의 요소를 검증하는 편리한 도구 세트를 제공합니다. 이를 위해 anyMatch(), allMatch(), noneMatch() 메서드 중 하나를 사용할 수 있습니다. 그들의 이름은 자명합니다. 다음은 부울을 반환하는 터미널 작업입니다.
(Stream API gives a handy set of instruments to validate elements of a sequence according to some predicate. To do this, one of the following methods can be used: anyMatch(), allMatch(), noneMatch(). Their names are self-explanatory. Those are terminal operations that return a boolean:)

```java
boolean isValid = list.stream().anyMatch(element -> element.contains("h")); // true
boolean isValidOne = list.stream().allMatch(element -> element.contains("h")); // false
boolean isValidTwo = list.stream().noneMatch(element -> element.contains("h")); // false
```

* 빈 스트림의 경우 지정된 술어가 있는 allMatch() 메서드는 true를 반환합니다.

```java
Stream.empty().allMatch(Objects::nonNull); // true
```

* 술어를 만족하지 않는 요소를 찾을 수 없기 때문에 이것은 합리적인 기본값입니다.
(This is a sensible default, as we can't find any element that doesn't satisfy the predicate.)

* 마찬가지로 anyMatch() 메서드는 빈 스트림에 대해 항상 false를 반환합니다.
```java
Stream.empty().anyMatch(Objects::nonNull); // false
```
* 다시 말하지만, 이 조건을 만족하는 요소를 찾을 수 없기 때문에 이것은 합리적입니다.


### Reduction

* Stream API를 사용하면 Stream 유형의 reduce() 메서드를 사용하여 지정된 함수에 따라 요소 시퀀스를 일부 값으로 줄일 수 있습니다. 이 메서드는 두 개의 매개변수를 사용합니다. 첫 번째 – 시작 값, 두 번째 – 누산기 기능.
(Stream API allows reducing a sequence of elements to some value according to a specified function with the help of the reduce() method of the type Stream. This method takes two parameters: first – start value, second – an accumulator function.)


* List<Integer>가 있고 이러한 모든 요소와 일부 초기 Integer(이 예에서는 23)의 합계를 원한다고 상상해 보십시오. 따라서 다음 코드를 실행할 수 있으며 결과는 26(23 + 1 + 1 + 1)이 됩니다.

```java
List<Integer> integers = Arrays.asList(1, 1, 1);
Integer reduced = integers.stream().reduce(23, (a, b) -> a + b);
```


### Collecting
* reduction는 Stream 유형 의 collect() 메서드로 도 제공할 수 있습니다 . 이 작업은 스트림을 Collection 또는 Map 으로 변환 하고 단일 문자열 형태로 스트림을 나타내는 경우에 매우 편리 합니다 . 거의 모든 일반적인 수집 작업에 대한 솔루션을 제공 하는 유틸리티 클래스 수집기 가 있습니다. 사소한 작업이 아닌 일부 작업의 경우 사용자 지정 수집기 를 만들 수 있습니다.


```java
List<String> resultList 
  = list.stream().map(element -> element.toUpperCase()).collect(Collectors.toList());
```

* 이 코드는 터미널 collect() 작업을 사용하여 Stream<String>을 List<String>으로 줄입니다.


## 참고
* https://futurecreator.github.io/2018/08/26/java-8-streams/
* https://www.baeldung.com/java-streams
    * https://www.baeldung.com/java-8-streams-introduction