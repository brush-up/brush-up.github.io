---
title:  "[java]spring boot 2.x에서 actuator 관련(micrometer )"
excerpt: "spring boot 2.x에서 actuator 관련(micrometer ) 참고용 정리"

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 2019-07-17
last_modified_at: 2019-07-17

published: true
---

## 기본 변경사항등
* /health 와 /info를 제외한 대분분의 endpoints 는 기본적으로 막혀있다. 
* 포토설정방법은 아래처럼 변경됨
    ```
    management.port=8081
    처럼 하던것에서 아래처럼 변경
    management.server.port=8081
    ```
* 추가하거나 막기 위해선 아래처럼 설정하면 될듯
    ```
    management.endpoints.web.exposure.include=health,info,env,metrics
    management.endpoints.web.exposure.exclude=health    
    ```

* 기본경로가 /application 에서 /actuator로 변경되었다. 
    * 바꾸려면 아래와 같이 설정하면 될듯
        ```
        management.endpoints.web.base-path=/application
        ```
        * 이렇게 하면 http://127.0.0.1:12001/actuator/health 에서 http://127.0.0.1:12001/application/health 로 접근 가능.

* @WebEndpointExtension 어노테이션이 @EndpointWebExtension 으로 변경되었고
* @WebEndpoint, @JmxEndpoint 추가되었는데 아래처럼 할 경우 web에만 노출하거나 jmx로 노출하려고 하면 이렇게 쓰면 된다.
* `@Endpoint` 어노테이션을 쓰면 web, jmx둘다 노출된다.
* 아래처럼 하는경우 는.. 
```java
@WebEndpoint(id = "hello")
@Component
public class HelloWebEndpoint {

  @ReadOperation
  public String hello() {
    return "hello world";
  }
}
```
## micrometer 관련 
* 이제 spring 에서 micrometer Metrics를 지원하기 시작함
* ../actuator/metrics 로 요청하면 아래처럼 나옴 (예전에는 모든 데이터가 다 나왔었던것 같은데..)
    ```
    {
      "names": [
        "jvm.memory.max",
        "jvm.threads.states",
        "process.files.max",
        "jvm.gc.memory.promoted",
        ...
      ]
    }
    ```
* 개별 실제 값을 보려면 아래처럼 항목 지정해서 요청해야해
    * .../actuator/metrics/`jvm.threads.states`
    ```
    {
      "name": "jvm.threads.states",
      "description": "The current number of threads having TERMINATED state",
      "baseUnit": "threads",
      "measurements": [
        {
        "statistic": "VALUE",
        "value": 24
        }
      ],
      "availableTags": [
        {
          "tag": "state",
          "values": [
            "timed-waiting",
            "new",
            "runnable",
            "blocked",
            "waiting",
            "terminated"
          ]
        }
      ]
    }
    ```
* Micrometer 조금 더
    * management.metrics.web.server.auto-time-requests 특성이 true로 설정되면 Spring MVC, WebFlux 및 Jersey 서버는 각각 모든 요청에 대한 타이밍 정보를 자동으로 수집하는 Spring의 자동 구성을 제공합니다. 기본 설정은 true
    * 사용자 정의 만들기
        * Micrometer는 몇 가지 기본 Meter 유형을 정의합니다
            * `Counter`는 증가할 수 있는 값을 추적합니다.
            * ` Gauge`는 미터가 공개(또는 조회)될 때 관찰된 값을 측정하고 리턴합니다.
            * ` Timer`는 이벤트 발행 횟수 및 이 이벤트의 누적 경과 시간을 추적합니다.
            * `DistributionSummary`는 Timer와 유사하며, 이벤트 분포를 추적하지만 시간을 측정하지는 않습니다.
        * MeterRegistry가 삽입된 후 @PostConstruct 메소드에서 또는 생성자(MeterRegistry를 매개변수로서 전달)에서 Meter를 작성하십시오.


    * Counter
        * 가령 RestController 내에서 Counter를 사용하기 위해선 아래처럼 할수 있어
            ```java
            @RestController
            public class MyController {

                private final Counter myCounter;

                public MyController(MeterRegistry meterRegistry) {

                    // Create the counter using the helper method on the builder
                    myCounter = meterRegistry.counter("my.counter", "mytagname", "mytagvalue");

                }
            ```
        * 그런 다음 서비스 메소드 내에서 미터 값을 업데이트할 수 있어
            ```java
            @GetMapping("/")
            public String getSomething() {
                myCounter.increment();
                ...
            }
            ```
        * 다른 예제
            ```java
            private Counter counter;
            private final MeterRegistry meterRegistry;

            ...
            
            this.counter = Counter
                    .builder("yuik.yuik")
                    .description("yuik description")
                    .tags("dev", "performance")
                    .register(meterRegistry);
                    
            ...
            this.counter.increment();
            ```
    * Timer
        * 짧은기간동안의 이벤트 빈도를 측정하기 위한것이야.
        * 아래처럼 간단하게 사용도 가능
            ```java
            private Timer timer;
            private final MeterRegistry meterRegistry;
            ...

            this.timer = Timer
                    .builder("yuik.timer")
                    .description("yuik timer description")
                    .register(meterRegistry);
            ...
            this.timer.record(2000, TimeUnit.MILLISECONDS);
            ```
            * 이런경우 아래처럼 요청하면 결과값은 아래처럼 누적치로 COUNT, TOTAL_TIME 나 나타남.
            `...actuator/metrics/yuik.timer`
                ```
                {
                    "name": "yuik.timer",
                    "description": "yuik timer description",
                    "baseUnit": "seconds",
                    "measurements": [{
                            "statistic": "COUNT",
                            "value": 1.0
                        }, {
                            "statistic": "TOTAL_TIME",
                            "value": 8.277E-6
                        }, {
                            "statistic": "MAX",
                            "value": 8.277E-6
                        }
                    ],
                    "availableTags": []
                }

                ```
        * 아니면 아래처럼 시간측정해서 할수도있어
            ```java
                private Timer timer;
                private final MeterRegistry meterRegistry;
                
                ...
                
                this.timer = Timer
                        .builder("yuik.timer")
                        .description("yuik timer description")
                        .register(meterRegistry);
                ...
                
                Timer.Sample sample = Timer.start(meterRegistry);
                
                ...
                
                sample.stop(this.timer);   //내부적으로 record 호출
            ```


* 추가적으로 아래자료 더 보기
    * http://micrometer.io/docs/concepts
    * https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html
    * https://www.baeldung.com/micrometer
    * `Spring Boot Actuator provides dependency management and auto-configuration for Micrometer. Now it’s supported in Spring Boot 2.0/1.x and Spring Framework 5.0/4.x.`
    * https://www.slideshare.net/makingx/spring-boot-actuator-20-micrometer

