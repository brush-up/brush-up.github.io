---
title:  "[java] HashMap 관련 정리(java8에서 간결하고 효과적으로 사용하기)"
excerpt: "Java HashMap 관련 정리(java8에서 간결하고 효과적으로 사용하기)"

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 2022-04-11
last_modified_at: 2022-04-11
---


# 1 Overview

* Java에서 HashMap 을 사용하는 방법 과 내부적으로 어떻게 작동하는지 알아볼꺼임
* HashMap 과 매우 유사한 클래스 는 Hashtable 인데, 차이점까지는 굳이 알 필요 없을듯
    * 혹시나 궁금하면 아래를 보자
        * [https://www.baeldung.com/hashmap-hashtable-differences](https://www.baeldung.com/hashmap-hashtable-differences)

# 2 Basic Usage

* map은 key-value 매핑이다.
    * 모든 키가 정확히 하나의 값에 매핑되고 키를 사용하여 맵에서 해당 값을 검색할 수 있다
* 단순히 list 에 값을 추가하지 않는 이유 중 가장 간단한 것은 성능이슈이다.
    * list에서 특정 요소를 찾으려면 시간 복잡도는 O(n) 이고 목록이 정렬된 가령, 이진 검색을 사용하면 O(log n) 이 된다
    * HashMap 의 장점은 값을 삽입하고 검색하는 시간 복잡도가 평균 O(1) 이다.

## 2.1. Setup

* 이 문서 전체에서 사용할 예제는 아래와 같다.

``` java
public class Product {

    private String name;
    private String description;
    private List<String> tags;

    // standard getters/setters/constructors

    public Product addTagsOfOtherProduct(Product product) {
        this.tags.addAll(product.getTags());
        return this;
    }
}
```

## 2.2. Put

* String 타입의 키와 Porduct 타입의 요소들로 HashMap 을 생성할수 있다.

``` java
Map<String, Product> productsByName = new HashMap<>();
```

* HashMap 에 Product 를 put 하자.

``` java
Product eBike = new Product("E-Bike", "A bike with a battery");
Product roadBike = new Product("Road bike", "A bike for competition");
productsByName.put(eBike.getName(), eBike);
productsByName.put(roadBike.getName(), roadBike);
```

## 2.3. Get

* 아래처럼 key 로 map에서 value를 가져올수 있다.

``` java
Product nextPurchase = productsByName.get("E-Bike");
assertEquals("A bike with a battery", nextPurchase.getDescription());
```

* 만약 map에 없는 key로 vlaue를 찾으려고 하면 null 값을 얻게 된다.

``` java
Product nextPurchase = productsByName.get("Car");
assertNull(nextPurchase);
```

* 만약 동일한 key로 2번 insert 하게 되면, map에는 마지막으로 삽입된 value로 대체된다.

``` java
Product newEBike = new Product("E-Bike", "A bike with a better battery");
productsByName.put(newEBike.getName(), newEBike);
assertEquals("A bike with a better battery", productsByName.get("E-Bike").getDescription());
```

## 2.4. Null as the Key

* HashMap은 key값으로 null을 허용한다.

``` java
Product defaultProduct = new Product("Chocolate", "At least buy chocolate");
productsByName.put(null, defaultProduct);

Product nextPurchase = productsByName.get(null);
assertEquals("At least buy chocolate", nextPurchase.getDescription());
```

## 2.5. Values with the Same Key

* 다른 key로 동일한 value를 각각 insert 할수 있다.

``` java
productsByName.put(defaultProduct.getName(), defaultProduct);
assertSame(productsByName.get(null), productsByName.get("Chocolate"));
```

## 2.6. Remove a Value

* HashMap 에서 key-value 매핑을 지울수 있다.

``` java
productsByName.remove("E-Bike");
assertNull(productsByName.get("E-Bike"));
```

## 2.7. Check If a Key or Value Exists in the Map

* map에 key가 있는지 확인하려면 containsKey() 메소드를 사용하면 된다.

``` java
productsByName.containsKey("E-Bike");
```

* 또는 map에 값이 있는지 확인하기 위해 containsValue() 메소드를 사용할수 있다.


``` java
productsByName.containsValue(eBike);
```

* 위의 두 메소드 모두 true를 반환한다. 키가 있는지 확인하는 시간복잡도는 O(1) 이지만 value를 확인하는 시간 복잡도는 O(n)이다. 성능차이가 중요하다.


## 2.8. Iterating Over a HashMap

* HashpMap의 모든 key-value쌍을 순회하는 방법은 기본적으로 3가지가 있다.

* 1번째 방법
``` java
for(String key : productsByName.keySet()) {
    Product product = productsByName.get(key);
}
```

* 2번째 방법
``` java
for(Map.Entry<String, Product> entry : productsByName.entrySet()) {
    Product product =  entry.getValue();
    String key = entry.getKey();
    //do something with the key and value
}
```

* 3번째 방법
``` java
List<Product> products = new ArrayList<>(productsByName.values());
```

# 3 The Key
* HashMap의 key로 어떤 class도 사용할수 있다. 
    * 다만, map이 잘 동작하려면 equals() 와 hashCode() 를 구현해야 한다.
* Product 를 key로 사용하고 price를 value로 하는 map을 예제로 들어본다.

``` java
HashMap<Product, Integer> priceByProduct = new HashMap<>();
priceByProduct.put(eBike, 900);
```

* equals() 와 hashCode() 메소드를 구현해보자.

``` java
@Override
public boolean equals(Object o) {
    if (this == o) {
        return true;
    }
    if (o == null || getClass() != o.getClass()) {
        return false;
    }

    Product product = (Product) o;
    return Objects.equals(name, product.name) &&
      Objects.equals(description, product.description);
}

@Override
public int hashCode() {
    return Objects.hash(name, description);
}
```

* Note that hashCode() and equals() need to be overridden only for classes that we want to use as map keys, not for classes that are only used as values in a map. 

# 4 Additional Methods as of Java 8

* java 8에서 몇가지  functional 스타일의 메소드가  HashMap에 추가가 되었다.
* 이에 대해 몇가지 알아 보겠다.

## 4.1. forEach()

* forEach 메소드는  map의 모든 요소들을 순회하는 방법이다.

``` java
productsByName.forEach( (key, product) -> {
    System.out.println("Key: " + key + " Product:" + product.getDescription());
    //do something with the key and value
});
```

Prior to Java 8:

``` java
for(Map.Entry<String, Product> entry : productsByName.entrySet()) {
    Product product =  entry.getValue();
    String key = entry.getKey();
    //do something with the key and value
}
```

* 조금더 자세한 내용은 아래를 확인하자
    * https://www.baeldung.com/foreach-java

## 4.2. getOrDefault()

* getOrDefault() 를 이용하여 map에서 값을 가져오거나 값이 없는 경우 기본 값을 반환할 수 있다.

``` java
Product chocolate = new Product("chocolate", "something sweet");
Product defaultProduct = productsByName.getOrDefault("horse carriage", chocolate); 
Product bike = productsByName.getOrDefault("E-Bike", chocolate);
```

Prior to Java 8:

``` java
Product bike2 = productsByName.containsKey("E-Bike") 
    ? productsByName.get("E-Bike") 
    : chocolate;
Product defaultProduct2 = productsByName.containsKey("horse carriage") 
    ? productsByName.get("horse carriage") 
    : chocolate;
```

## 4.3. putIfAbsent()

* 지정된 key에 value가 매핑 되지 않았으면 새 매핑을 추가할수 있는 기능이다.

``` java
productsByName.putIfAbsent("E-Bike", chocolate);
```

Prior to Java 8:

``` java
if(productsByName.containsKey("E-Bike")) {
    productsByName.put("E-Bike", chocolate);
}
```

* 조금더 자세한 내용은 아래를 확인하자
    * https://www.baeldung.com/java-merge-maps

## 4.4. merge()

* merge() 를 사용ㅇ하면 매핑이 존재하는 경우 키값을 수정하거나, 그렇지 않은 경우 새 값을 추가할수 있다.

``` java
Product eBike2 = new Product("E-Bike", "A bike with a battery");
eBike2.getTags().add("sport");
productsByName.merge("E-Bike", eBike2, Product::addTagsOfOtherProduct);
```

Prior to Java 8:

``` java
if(productsByName.containsKey("E-Bike")) {
    productsByName.get("E-Bike").addTagsOfOtherProduct(eBike2);
} else {
    productsByName.put("E-Bike", eBike2);
}
```

## 4.5. compute()

* compute() 는 키에 대한 값을 계산 할 수 있다.

``` java
productsByName.compute("E-Bike", (k,v) -> {
    if(v != null) {
        return v.addTagsOfOtherProduct(eBike2);
    } else {
        return eBike2;
    }
});
```

Prior to Java 8:

``` java
if(productsByName.containsKey("E-Bike")) {
    productsByName.get("E-Bike").addTagsOfOtherProduct(eBike2); 
} else {
    productsByName.put("E-Bike", eBike2); 
}
```


# 5 HashMap Internals

* HashMap 내부적으로 어떻게 동작하는지 알아보자
    * 간단한 list대신 HashMap 의 이점도 함께 알아본다.
    * HashMap 을 사용하면 put 과 get 작업에 대해 시간 복잡도  O(1)의 평균 시간 복잡도와 O(n) 의 공간 복잡도(메모리 양)를 얻을 수 있다.

## 5.1. The Hash Code and Equals

* 모든 값을 반복하는 대신 HashMap은 key를 기반으로 위치를 계산하려 시도한다.
* HashMap은 buckets 이라고 불리는 곳에  value를 저장하고 , 이 버킷의 수를 용량(capacity)이라고 부른다. 
* map에 value를 넣을때 hashCode() 메서드는 값이 저장될 버킷을 결정하는데 사용된다. 
* 값을 검색하기 위해 HashMap은 hashCode() 를 사용하여 동일한 방식으로 버킷을 계산한다. 그런 다음 해당 버킷에서 찾은 객체를 iterates 하고 equals() 메소드를 사용하여 정확히 일치하는 항목을 찾는다.

## 5.2. Keys' Immutability
* 대부분의 경우 변경할수 엇ㅂ는 key를 사용해야한다. 
* 맵에 값을 저장하는 키를 사용한후 키 값을 변경하면 어떻게 될까?
    * 아래 예시에선 MutableKey를 생성한다.

``` java
public class MutableKey {
    private String name;

    // standard constructor, getter and setter

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        MutableKey that = (MutableKey) o;
        return Objects.equals(name, that.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name);
    }
}
```

* 자 , 테스트를 진행해보자.

``` java
MutableKey key = new MutableKey("initial");

Map<MutableKey, String> items = new HashMap<>();
items.put(key, "success");

key.setName("changed");

assertNull(items.get(key));
```

* 보시다시피 key가 변경되면 더 이상 key에 해당하는 값을 가져올수 없으며 대산 null이 반환된다. 

## 5.3. Collisions

* 2개의 다른 키에 동일한 해시가 있는 경우는 어떻게 될까? 이 경우 해당 키에 속한 2개의 값은 동일한 버킷에 저장된다.
    * 버킷 내부의 값은 list에 저장되고 모든 요소를 반복하여 검색이 된다 이것의 비용은 O(n)이 된다.
        *  java8 부터 한 버킷 내부의 값이 저장되는 데이터 구조는 버킷에 8개 이상의 값이 포함된 경우 list에서 balanced tree 로 변경이 되고 반대의 경우도 마찬가지이다.


## 5.4. Capacity and Load Factor

* value가 여러개인 버킷이 많은 것을 방지하기 위해 버킷의 75%가 비어있지 않는 경우 용량은 2배로 늘어남
* 기본값은 75%이고 초기 용량은 16 (둘다 생성자에서 설정할 수 있음)

## 5.5. Summary of put and get Operations

* map에 put 할  경우 HashMap 이 버킷을 계산함.
    * 버킷에 이미 값이 포함되어이있으면 해당 버킷에 속한 list에 값을 추가함
* map에 get 할 경우 
    * HashMap 이 버킷을 계산하고 list나 tree에서 동일한 키 값을 가진 값을 가지고 옮

# 6 Conclusion

* 내부를 더 알고 싶으면 아래 자료를 더 보자
    * https://www.baeldung.com/java-hashmap-advanced

# 7. 조금 더 알아보자

## putIfAbsent(), computeIfAbsent()
* 2가지 메서드의 공통점은 key의 존재 여부에 따라서 새로운 key와 value 값을 추가하는 메소드.

### putIfAbsent
```java
default V putIfAbsent(K key, V value) 
```
* key : Map의 key 값
* value : value 값
* 반환 값
    * key 값이 존재하는 경우
    * Map의 value 값을 반환한다
    * key 값이 존재하지 않는 경우
    * key와 value를 Map에 저장하고 null을 반환하다

```java
@Test
public void putIfAbsent() {
  Map<String, Integer> map = new HashMap<>();
  map.put("John", 5);

  assertThat(map.putIfAbsent("John", 10)).isEqualTo(5); //존재하는 경우, value값을 반환한다
  assertThat(map.size()).isEqualTo(1);

  assertThat(map.putIfAbsent("John2", 10)).isNull(); //없는 경우, null로 반환하고 map에 저장함
  assertThat(map.size()).isEqualTo(2);
}
```

### computeIfAbsent

```java
default V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction)
```
* key : Map의 키 값
* mappingFunction
    * mappingFunction 람다 함수는 key 값이 존재하지 않을 때만 실행된다
* 반환값
    * key 값이 존재하는 경우
    * map안에 있는 value을 반환한다
    * key 값이 존재하지 않는 경우
    * Map에 새로운 key와 value(mappingFunction 람다 함수를 실행한 결과) 값을 저장한다

```java
@Test
public void computeIfAbsent() {
  Map<String, Integer> map = new HashMap<>();
  map.put("John", 5);

  assertThat(map.computeIfAbsent("John", key -> key.length())).isEqualTo(5); //존재하면 value값을 반환함
  assertThat(map.size()).isEqualTo(1);

  //없으면 2번째 인자 함수를 실행한 결과를 반환하고 map에도 추가가 된다
  assertThat(map.computeIfAbsent("John2", key -> key.length())).isEqualTo("John2".length());
  assertThat(map.get("John2")).isNotNull();
  assertThat(map.size()).isEqualTo(2);

  assertThat(map.computeIfAbsent("John3", key -> null)).isNull();
  assertThat(map.size()).isEqualTo(2);
}
```


## compute(), computeIfPresent(), merge()
* 3개의 메서드들은 모두 Map의 value 값을 업데이트할 때 사용

### compute
    * compute는 key와 remappingFunction을 인자로 받고 key가 존재해야, value값을 인자로 넘겨준 remappingFunction 람다 함수의 결과로 업데이트가 됩니다. key 값이 존재하지 않는 경우에는 NullPointerException이 발생합니다.
```java
default V compute(K key,
        BiFunction<? super K, ? super V, ? extends V> remappingFunction)
```

```java
@Test
public void compute() {
    Map<String, Integer> map = new HashMap<>();
    map.put("john", 20);
    map.put("paul", 30);
    map.put("peter", 40);

    map.compute("peter", (k, v) -> v + 50);
    assertThat(map.get("peter")).isEqualTo(40 + 50);
}
```

### computeIfPresent

```java
default V compute(K key,
                  BiFunction<? super K, ? super V, ? extends V> remappingFunction)
```

* 결과 값
    * key 값이 존재하는 경우
        * remappingFunction 람다 함수 실행 결과로 value 값이 업데이트가 된다
    * key가 존재하지 않는 경우
        * null을 반환한다

```java
@Test
public void computeIfPresent() {
  Map<String, Integer> map = new HashMap<>();
  map.put("john", 20);
  map.put("paul", 30);
  map.put("peter", 40);

  map.computeIfPresent("kelly", (k, v) -> v + 10);
  assertThat(map.get("kelly")).isNull();

  map.computeIfPresent("peter", (k, v) -> v + 10);
  assertThat(map.get("peter")).isEqualTo(40 + 10);
}
```

### merge

```java
default V merge(K key, V value, BiFunction<? super V, ? super V, ? extends V> remappingFunction)
```
* 결과 값
* key 값이 존재하는 경우
    * Case 1 : remappingFunction 람다 함수의 결과가 null 아니면
        * remappingFunction 람다 함수 실행 결과로 value 값이 업데이트가 된다
    * Case 2 : remappingFunction 람다 함수의 결과가 null 이면
        * map에서 해당 key를 삭제한다
* key가 존재하지 않는 경우
    * Map에 key, value값이 추가된다

```java
@Test
public void merge() {
  Map<String, Integer> map = new HashMap<>();
  map.put("john", 20);
  map.put("paul", 30);
  map.put("peter", 40);

  //key값이 존재를 하면, 해당 key의 값을 remapping 함수의 결과 값으로 바꾼다
  map.merge("peter", 50, (k, v) -> map.get("john") + 10);
  assertThat(map.get("peter")).isEqualTo(30);

  //key가 존재하고 remapping 함수의 결과가 null이면 map에서 해당 key를 삭제한다
  map.merge("peter", 30, (k, v) -> map.get("nancy"));
  assertThat(map.get("peter")).isNull();
  assertThat(map.size()).isEqualTo(3);

  //key가 존재하지 않으면 key, value값을 추가함
  map.merge("kelly", 50, (k, v) -> map.get("john") + 10);
  assertThat(map.get("kelly")).isEqualTo(50);
  assertThat(map.size()).isEqualTo(4);

}
```

## getOrDefault()
* getOrDefault 가 반환하는 값은 아래와 같습니다.

```java
default V getOrDefault(Object key, V defaultValue)
```
* key 값이 존재하는 경우
    * Map의 value값을 반환한다
* key 값이 존재하지 않는 경우
    * defaultValue을 반환한다

```java
 @Test
public void getOrDefault() {
  String str = "aagbssdf";
  Map<Character, Integer> map1 = new HashMap<>();
  Map<Character, Integer> map2 = new HashMap<>();

  //getOrDefault 사용하지 않는 경우
  for (char c : str.toCharArray()) {
    if (map2.containsKey(c)) {
      map2.put(c, map2.get(c) + 1);
    } else {
      map2.put(c, 1);
    }
  }

  //getOrDefault 사용하는 경우
  for (char c : str.toCharArray()) {
    map1.put(c, map1.getOrDefault(c, 0) + 1);
  }

  assertThat(map1).isEqualTo(map2);
}
```

## 참고

* [https://www.baeldung.com/java-hashmap](https://www.baeldung.com/java-hashmap)
* [https://blog.advenoh.pe.kr/java/자바8-HashMap-보다-간결하고-효과적으로-작성하기/](https://blog.advenoh.pe.kr/java/%EC%9E%90%EB%B0%948-HashMap-%EB%B3%B4%EB%8B%A4-%EA%B0%84%EA%B2%B0%ED%95%98%EA%B3%A0-%ED%9A%A8%EA%B3%BC%EC%A0%81%EC%9C%BC%EB%A1%9C-%EC%9E%91%EC%84%B1%ED%95%98%EA%B8%B0/)

