---
title:  "[java] 어노테이션 정리2 (의존성 주입 관련 및 기타)"
excerpt: "Java 어노테이션 정리2 (의존성 주입 관련 및 기타)"

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 2022-03-29
last_modified_at: 2022-03-29
---



## Bean 객체 등록
* 빈(Bean)
    * 스프링 컨테이너에 의해 자바 객체가 만들어지게 되면 이 객체를 스프링은 스프링빈이라 부름
        * 스프링 빈과 자바 일반 객체와 차이는 없고 스프링 컨테이너에 의해 만들어진 객체를 스프링 빈이라고만 부를뿐
* 스프링 빈의 어노테이션
    * @Component, @Service, @contorller @Repository @Bean, @Configuration 등으로 필요한 빈들을 등록하고 필요한 곳에서 @Autowired를 통해 주입받아 사용하는 것이 일반적
	![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-03-29-java-annotation-02-01.PNG)


* Bean 을 주입하는 방식 3가지
    * @Autowired
    * setter injection
    * 생성자(@AllArgsConstructor 사용)

* @Component , @Autowired
    * 스프링빈 으로 등록하기 가장 쉬운 방법은 클래스 선언부 위에 Component 어노테이션을 사용하는것
    * component-scan을 선언에 의해 특정 패키지 안의 클래스들을 스캔
    * ex
    ```java
    public interface AdminService {
        void activeService();
    }
    ```
    ```java
    @Component
    public class AdminServiceImpl implements AdminService {

        @Override
        public void activeService(String void){

        }

    }
    ```
    ```java
    public class AdminController {

        @Autowired
        private AdminService adminService;

        @PostMapping(value = "/active")
        public Response activeService(@RequestHeader(value = "test", required = false) String test,
                                         @RequestBody(required = true) @Valid RequestSetData requestSetData){

            adminService.activeService(transactionId);
            ...
        }

    }
    ```
* @Primary, @Qualifier
    * @Primary : @Bean 혹은 @Component와 함께 사용하며 객체 생성의 우선권을 부여
    * @Quallifier : @Autowired와 함께 사용하며 Bean 의 이름이 같은 객체를 찾음
        * 같은 타입의 빈이 2개 이상 존재할 경우 특정 의존성을 주입할수있도록 하기위해 사용
        * @Bean("name") 으로 주고 @Autowired("name")로 사용.
* @Bean 과 @Component의 차이
    * @Bean은 메소드 레벨에서 선언하며, 반환되는 객체(인스턴스)를 개발자가 수동으로 빈으로 등록하는 애노테이션이다.
        * (예를 들면 Redis와 연동하기위해 RedisConfig 클래스에 @Component를 선언할수는 없으니 RedisTemplate의 인스턴스를 생성하는 redisTemplate메소드를 만들고 해당 메소드에 @Bean을 선언하여 Bean으로 등록한다.)
    * @Component는 클래스 레벨에서 선언함으로써 스프링이 런타임시에 컴포넌트스캔을 하여 자동으로 빈을 찾고(detect) 등록하는 애노테이션이다.      
        
## 스프링 프레임 워크에서 의존성 주입 방법
* 등록된 빈을 사용하기 위한 스프링 프레임워크의 DI(Dependency Injection) 방법은 3가지
    * 생성자 주입(Constructor Injection)
    * 필드 주입(Field Injection)
    * 수정자 주입(Setter Injection)

### 생성자 주입
```java
@Component
public class TestCode {
    private final Test test;
    public TestCode(Test test){
        this.test = test;
    }
}
```
* 단일 생성자의 경우 @Autowired 어노테이션 붙이지 않아도 되지만, 생성자가 2개이상이면 생성자에 붙여줘야함
* 롬복을 쓰면 쉽게 처리된다
```java
@Component
@RequiredArgsConstructor
public class TestCode {
    private final Test test;
}
```
* 만약 같은 타입의 빈이 있을 경우는  @Qualifer 를 통해 구현체의 클래스명이나 지정한 빈의 이름으로 가져올수 있다.
```java
@Component
@RequiredArgsConstructor
public class TestCode {

    @Qualifier("test")
    private final Test test;
}
```


### 필드 주입
* 필드에 @Autoried 어노테이션을 붙이면 된다.
    * 생성자 주입과 달리 필드를 final로 정의하지 못한다
    * 같은 타입의 빈이 있을 경우 @Qaualifer이나 @Resource 를 통해 빈의 이름으로 가져와서 주입할수 있다.
```java
@Component
public class TestCode {

    @Autowired
    private Test test;
}
```


### 수정자 주입
* 이 방식도 final 선언이 안된다. 
* 꼭 setter 메서드일 필요없이 메서드 이름이 수정자 네이밍 패텅이 아니어도 동일 기능을 하면 된다.
```java
@Component
public class TestCode {

    private Test test;
    
    @Autowired
    public void setTest(Test test){
        this.test = test;
    }
}
```

### 어떤 방식이 좋을까?
* 아래의 이유로 생성자 주입을 권장한다
    * 순환참조 방지
        * 생성자 주입 방식은 필드 주입이나 수정자 주입과는 빈을 주입하는 순서가 다르다
            * 필드 주입, 수정자 주입은 먼저 빈을 생성한 뒤 주입하려는 빈을 찾아 주입한다
            * 생성자 주입은 먼저 생성자안의 안자에서 사용되는 빈을 찾거나 빈 팩토리에서 만든다.
                * 먼저 빈을 생성하지 않고 주입하려는 빈을 먼저 찾는다.
    * final 선언이 가능
        * 필드 주입과 수정자 주입은 필드를 final로 선언할수없다 즉, 나중에 변경될수 있다.
        * 럴타임에 객체 불변성을 보장하려면 생성자 주입으로 하자
    * 테스트 코드 작성 용이


## 기타
* @RequestParam
* @PathVariable
* @RequestBody
* @ModelAttribute
* 아래 샘플을 보면 그냥 이해가 될듯하여 설명은 패스.

```java
    @PutMapping(value = "/keys/{key}/nodes/{node}")
    public ResponseCommon updateNode(@RequestHeader(value = "TRANSACTION_ID", required = false) String transactionId,
                                        @RequestHeader(value = "SECRET_KEY", required = false, defaultValue = "") String secretKey,
                                        @PathVariable String key,
                                        @PathVariable String node,
                                        @RequestBody(required = true) @Valid RequestUpdateNode requestUpdateNode){

        //...
    }
```



## 기타?

* @PostConstruct, @PreDestroy
    * https://docs.oracle.com/javaee/7/api/javax/annotation/PostConstruct.html
    * PostConstruct 는 의존성 주입이 이루어진 후 초기화를 수행
    * PreDestroy 는 그 반대겠지?

* @RestController
    * @Controller와 @ResponseBody의 조합
    * 다음 두 코드는 Spring MVC에서 동일한 동작
    ```java
    @Controller
    @ResponseBody
    public class MVCController{
        logic...
    }

    @RestController
    public class ReftFulController{
        logic...
    }
    ```
    * json 으로 응답보내기 위한 어노테이션
    * 하위의 메소드들은 전부 @ResponseBody 가 적용됨
* @ResponseBody
    * @Controller  어노테이션을 가지고 있지만 텍스트를 반환하고 싶을때 사용.