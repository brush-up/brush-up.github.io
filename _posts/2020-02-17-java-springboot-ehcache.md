---
title:  "[java][작성중]Spring Boot + Ehcache 예제 정리중"
excerpt: "Spring Boot + Ehcache 예제 정리중 "

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 2020-02-17
last_modified_at: 2020-02-17

published: true
---


# spring boot 캐시 사용 예시

## intro

* ehcache 3.x버전부터 JSR-107([https://www.jcp.org/en/jsr/detail?id=107](https://www.jcp.org/en/jsr/detail?id=107)과의 호환성을 제공한다
* JSR : (Java Specification Requests) 사양 및 기술적 변경에 대한 정식 제안 문서. 개인 및 조직은 JCP (Java Community Process)의 회원이 될 수 있으며 JSR에 언급 된 스펙에 따라 코드를 개발할 수 있다. 개발 된 기술적 변화는 JCP 회원들의 검토를 거쳐 승인된다.
* JSR-107 : (JCACHE – Java Temporary Caching API) 객체 생성, 공유 액세스, 스풀링, 무효화 및 JVM 전반에 걸친 일관성을 포함하여 Java 객체의 메모리 캐싱에서 사용할 API 에 대한 기준으로 볼 수 있다. 해당 Spec 으로 구현된 cache로는 EhCache가 유명하며, Hazelcast, Infinispan, Couchbase, Redis, Caffeine 등도 해당 기준을 따르는 것으로 알려져 있다.

## Dependencies

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.4.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
    <version>2.4.0</version></dependency>
<dependency>
    <groupId>javax.cache</groupId>
    <artifactId>cache-api</artifactId>
    <version>1.1.1</version>
</dependency>
<dependency>
    <groupId>org.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>3.8.1</version>
</dependency>
```

## 예제

### 서비스를 호출하여 숫자를 제곱한 결과를 json으로 반환하는 간단한 rest api 예제

``` java
@RestController
@RequestMapping("/number", MediaType.APPLICATION_JSON_UTF8_VALUE)
public class NumberController {
    // ...
    @Autowired
    private NumberService numberService;
    @GetMapping(path = "/square/{number}")
    public String getSquare(@PathVariable Long number) {
        log.info("call numberService to square {}", number);
        return String.format("{\"square\": %s}", numberService.square(number));
    }
}
```

* spring 이 캐싱을 처리할수있도록 @Cacheable 어노테이션을 달자. 이 어노테이션의 결과로 스프링은 square 메서드에 대한 호출을 가로채고 Ehcache를 호출하기 위한 NumberService의 프록시를 생성한다.
* 캐시 이름 및 추가 제한 조건을 달자.

``` java
@Service
public class NumberService {
    // ...
    @Cacheable(
      value = "squareCache", 
      key = "#number", 
      condition = "#number>10")
    public BigDecimal square(Long number) {
        BigDecimal square = BigDecimal.valueOf(number)
          .multiply(BigDecimal.valueOf(number));
        log.info("square of {} is {}", number, square);
        return square;
    }
}
```

## 캐시 설정

* @EnableCaching 어노테이션을 달면 캐시를 사용할수 있도록 활성화된다.

``` java
@Configuration
@EnableCaching
public class CacheConfig {
}
```

* 스프링의 auto-configuration은 JSR-107의 구현을 찾는다. 하지만 캐시가 생성되지 않는 것이 기본 설정값이다.
ehcache.xml file 설정이 있어야 하기에 다음 처럼 설정 위치를 알려줘야한다.

```
spring.cache.jcache.config=classpath:ehcache.xml
```

* squareCache 캐시 생성을 위해 아래처럼 ehcache.xml 파일을 생성해보자

``` xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://www.ehcache.org/v3"
    xmlns:jsr107="http://www.ehcache.org/v3/jsr107"
    xsi:schemaLocation="
            http://www.ehcache.org/v3 http://www.ehcache.org/schema/ehcache-core-3.0.xsd
            http://www.ehcache.org/v3/jsr107 http://www.ehcache.org/schema/ehcache-107-ext-3.0.xsd">
    <cache alias="squareCache">
        <key-type>java.lang.Long</key-type>
        <value-type>java.math.BigDecimal</value-type>
        <expiry>
            <ttl unit="seconds">30</ttl>
        </expiry>
        <listeners>
            <listener>
                <class>com.baeldung.cachetest.config.CacheEventLogger</class>
                <event-firing-mode>ASYNCHRONOUS</event-firing-mode>
                <event-ordering-mode>UNORDERED</event-ordering-mode>
                <events-to-fire-on>CREATED</events-to-fire-on>
                <events-to-fire-on>EXPIRED</events-to-fire-on>
            </listener>
        </listeners>
        <resources>
            <heap unit="entries">2</heap>
            <offheap unit="MB">10</offheap>
        </resources>
    </cache>
</config>
```

* CREATED 와 EXPIRED 캐시 이벤트 로그를 남길 리스너도 추가해보자

``` java
public class CacheEventLogger 
  implements CacheEventListener<Object, Object> {
    // ...
    @Override
    public void onEvent(
      CacheEvent<? extends Object, ? extends Object> cacheEvent) {
        log.info(/* message */,
          cacheEvent.getKey(), cacheEvent.getOldValue(), cacheEvent.getNewValue());
    }
}
```

* 이제 실행 및 테스트는 알아서...
rest api로 호출되면 아래처럼 로그가 남을 테지

```
INFO [nio-8080-exec-1] c.b.cachetest.rest.NumberController : call numberService to square 12
INFO [nio-8080-exec-1] c.b.cachetest.service.NumberService : square of 12 is 144
INFO [e [_default_]-0] c.b.cachetest.config.CacheEventLogger : Cache event CREATED for item with key 12. Old value = null, New value = 144
```

CREATED 이벶트 로그도 이렇게 남을것이고

```
INFO [nio-8080-exec-2] c.b.cachetest.rest.NumberController : call numberService to square 12
```

NumberService 의 square 메소드 호출 로그가 남지 않을것이지.
30초 이후에 다시 호출하면 아래처럼 EXPIRED 이벤트가 남고 다시 캐시에 추가가 되겠지

```
INFO [nio-8080-exec-1] (...) NumberController : call numberService to square 12
INFO [e [_default_]-1] (...) CacheEventLogger : Cache event EXPIRED for item with key 12. Old value = 144,New value = null
INFO [nio-8080-exec-1] (... )NumberService : square of 12 is 144
INFO [e [_default_]-1] (...) CacheEventLogger : Cache event CREATED for item with key 12. Old value = null, New value = 144
```

마지막으로 @Cacheable 어노테이션에 10보다 큰값을 캐시하도록 해서 9이하의 값은 캐쉬가 되지 않을것이야.

# 추가로 더 알아 보자.

## 캐시설정을 조금더

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://www.ehcache.org/ehcache.xsd"
         updateCheck="true"
         monitoring="autodetect"
         dynamicConfig="true">
 
    <cache name="사용할CacheName"
           maxElementsInMemory="1000"
           eternal="true"
           overflowToDisk="false"
           timeToLiveSeconds="300"
           timeToIdleSeconds="0"
           memoryStoreEvictionPolicy="LFU"
           transactionalMode="off">
    </cache>
 
</ehcache>
```

* name : 캐시 이름 지정
* maxEntriesLocalHeap: 메모리에 생성될 Entry Max값 (0=제한없음)
* maxEntriesLocalDisk: 디스크(DiskStore)에 저장될 Entry Max값 (0=제한없음)
* eternal: 영구 Cache 사용 여부 (true 인경우 timeToIdleSeconds, timeToLiveSeconds 설정은 무시된다.)
* timeToIdleSeconds: 해당 시간 동안 캐쉬가 사용되지 않으면 삭제. (0=삭제되지 않는다)
* timeToLiveSeconds: 해당 시간이 지나면 캐쉬는 삭제된다. (0=삭제되지 않는다)
* diskExpiryThreadIntervalSeconds: DiskStore 캐시 정리 작업 실행 간격 (Default=120초)
* diskSpoolBufferSizeMB: 스풀버퍼에 대한 DiskStore 크기 설정
* clearOnFlush: flush() 메서드 호출 시점에 메모리(MemoryStore) 삭제 여부. (Default=true)
* memoryStoreEvictionPolicy : maxEntriesLocalHeap 설정 값에 도달했을때 설정된 정책에 따라객체가 제거되고 새로 추가된다.
* logging: 로깅 사용 여부를 설정한다.
* maxEntriesInCache: Terracotta의 분산캐시에만 사용가능하며, 클러스터에 저장 할 수 있는 최대 엔트리 수를 설정한다. 0은 제한이 없다. 캐시가 작동하는 동안에 속성을 수정할 수 있다.
* overflowToOffHeap: 이 설정은 Ehcache 엔터프라이즈 버전에서 사용할 수 있다. true 로 설정하며 성능을 향상시킬 수 있는 Off-heap 메모리 스토리지를 활용하여 캐시를 사용할 수 있다. Off-heap 메모리 자바의 GC에 영향을 주지않는 다. (Default=false)
(참고사이트 : [http://www.ehcache.org/ehcache.xml](http://www.ehcache.org/ehcache.xml)

## ehcache 캐시 어노테이션

@CachePut : 캐시 생성

@Cacheable : 조회해서 없으면 캐시 생성하고 결과 반환, 캐시된 값이 있는 경우 해당 값 반환. key 정의 및 조건 정의 가능.

@CacheEvict : 캐시 삭제, allEntries를 이용해서 전체 삭제 가능.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
    updateCheck="false">
    <diskStore path="java.io.tmpdir" />

    <cache name="findMemberCache"
           maxEntriesLocalHeap="10000"
           maxEntriesLocalDisk="1000"
           eternal="false"
           diskSpoolBufferSizeMB="20"
           timeToIdleSeconds="300" timeToLiveSeconds="600"
           memoryStoreEvictionPolicy="LFU"
           transactionalMode="off">
        <persistence strategy="localTempSwap" />
    </cache>

</ehcache>
```

@Cacheable(value="findMemberCache", key="#name") 은 ehcache.xml에서 지정한 findMemberCache 캐시를 사용하겠다는 의미이며,
여기서 key는 메소드 argument인 name을 사용하겠다는 의미이다.
즉, name에 따라 별도로 캐시한다는 의미이다.
findByNameCache 메소드의 argument에 따라 캐시되기 때문에 name이 jojoldu인지, test1인지 등 name에 캐시 여부를 체크하여 캐시 안되어 있을 경우 캐시를 하고, 있으면 캐시된걸 전달하게 된다.

@CacheEvict(value = "findMemberCache", key="#name") 은 해당 캐시 내용을 지우겠다는 의미이다.
캐시 데이터가 갱신되어야 한다면 @CacheEvict가 선언된 메소드를 실행시키면 캐시 데이터는 삭제되고 새로운 데이터를 받아 캐시하게 된다.
@Cacheable과 마찬가지로 key에 따라 캐시를 선택해서 제거가 가능하다.

캐시와 비캐시 메소드들 간의 성능비교를 하기 위해 slowQuery라는 메소드를 추가하였다.
엄청나게 많은 양의 데이터가 존재하여 한번 조회 할때마다 2초 이상의 시간이 필요하다고 가정 했다.
slowQuery가 2초간 thread를 sleep 시키기 때문에 findByNameNoCache와 findByNameCache 메소드는 최소 2초 이상의 시간이 수행 된다.

## 참고

* [https://www.baeldung.com/spring-boot-ehcache](https://www.baeldung.com/spring-boot-ehcache)
* [http://blog.breakingthat.com/2018/03/19/springboot-ehcache-적용/](http://blog.breakingthat.com/2018/03/19/springboot-ehcache-적용/)
* [https://jojoldu.tistory.com/57](https://jojoldu.tistory.com/57)

