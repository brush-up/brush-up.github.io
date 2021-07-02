---
title:  "[java][작성중]springboot 에서 예외처리하기"
excerpt: "springboot 에서 예외처리하기"

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 2020-03-10
last_modified_at: 2020-03-10

published: true
---



# intro
## springboot 에서 기본적인 예외처리
* 존재하지 않는 페이지로 요청했을때 기본적으로 아래와 같이 응답값을 내려준다

```
Whitelabel Error Page
This application has no explicit mapping for /error, so you are seeing this as a fallback.

Mon May 17 11:28:09 KST 2021
There was an unexpected error (type=Not Found, status=404).
No message available
```

* 또한 Content-Type을 application/json 로 한경우는 아래처럼 에러를 내려준다.

```json
{
"timestamp": 1621218625595,
"status": 404,
"error": "Not Found",
"message": "No message available",
"path": "/err0r_test"
}
```

* 이러한 에러응답은 아래와 같은 설정을 통해 제어가 가능하다
```
# spring boot의 기본 properties
server.error:
  include-exception: false # 오류 응답에 exception의 내용을 포함할지 여부 (TRUE, FALSE) 
  include-stacktrace: never # 오류 응답에 stacktrace 내용을 포함할지 여부 (ALWAYS, NEVER, ON_TRACE_PARAM)
  path: '/error' # 오류 응답을 처리할 핸들러(ErrorController)의 path 
  whitelabel.enabled: true # 브라우저 요청에 대해 서버 오류시 기본으로 노출할 페이지를 사용할지 여부 (TRUE, FALSE)
```
* 기본적으로 어떻게 에러처리되어 응답하게 되는지는 다음에 확인해보자.
    * 우선 당장 사용해야할 부분부터... 


# 개요
* spring 3.2 이전 버전에서 예외를 처리하는 방식은  HandlerExceptionResolver 나 @ExceptionHandler 어노테이션을 사용하는 것이었다. 
* spring 3.2 이후부터는 @ControllerAdvice 어노테이션을 사용하는 방식으로 되었고
* spring 5부터는 rest api에서 기본 오류처리를 위한 빠른 방법인 ResponseStatusException 클래스가 도입되었다. 
## 솔루션1.
* controller 레벨에서 @ExceptionHandler 사용
* @ExceptionHandler 를 사용하는 방법은 특정 컨트롤러에만 활성화 되어 전체적으로 관리하기가 힘들다
```java
public class FooController{  
    //...
    @ExceptionHandler({ CustomException1.class, CustomException2.class })
    public void handleException() {
        //
    }
}
```

## 솔루션2.
* HandlerExceptionResolver 를 정의 하는 방법인데, 지금상황에서 굳이 알필요가 있을까 싶다. 

## 솔루션3.
* spring 3.2 부터 @ControllerAdvice 어노테이션을 사용하면 전역적으로 @ExceptionHandler 를 관리할수 있다. 
```java
@ControllerAdvice
public class RestResponseEntityExceptionHandler 
  extends ResponseEntityExceptionHandler {

    @ExceptionHandler(value 
      = { IllegalArgumentException.class, IllegalStateException.class })
    protected ResponseEntity<Object> handleConflict(
      RuntimeException ex, WebRequest request) {
        String bodyOfResponse = "This should be application specific";
        return handleExceptionInternal(ex, bodyOfResponse, 
          new HttpHeaders(), HttpStatus.CONFLICT, request);
    }
}
```

## 솔루션4.
* spring5 부터 ResponseStatusException  class를 도입하였다. 
* 장단점이있다고 하는데, 아직은 잘 모르겠..
```java
@GetMapping(value = "/{id}")
public Foo findById(@PathVariable("id") Long id, HttpServletResponse response) {
    try {
        Foo resourceById = RestPreconditions.checkFound(service.findOne(id));

        eventPublisher.publishEvent(new SingleResourceRetrievedEvent(this, response));
        return resourceById;
     }
    catch (MyResourceNotFoundException exc) {
         throw new ResponseStatusException(
           HttpStatus.NOT_FOUND, "Foo Not Found", exc);
    }
}
```


# 예제

* @ControllerAdvice 어노테이션을 통해 전역적으로 exception 핸들링이 가능하다
```java
@ControllerAdvice
public class TestExceptionHandler{
    ...
}
```
* 이곳에서 @ExceptionHandler 를 통해 관리가 가능하다
```java

@ControllerAdvice
public class TestExceptionHandler{
    
    @ExceptionHandler({TestException.class})
    public void handleTestException(HttpServletRequest request, TestException exception) {
        ...       
    }
}
```

* 최상위 에러는 아래처럼 Exception 를 넣어서 놓치는 에러가 없도록 하자
```java
@ControllerAdvice
public class TestExceptionHandler{
    
    @ExceptionHandler(Exception.class)
    public void handleException(HttpServletRequest request, Exception ex) {
        ...
    }
}    
```


# 참고
* Spring Web MVC - exception
    * https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-exceptionhandler
* Error Handling for REST with Spring
    * https://www.baeldung.com/exception-handling-for-rest-with-spring
* 이것 더 봐야함
    * https://supawer0728.github.io/2019/04/04/spring-error-handling/
	* 테스트 관련 아래 사항 더 보기
		* https://jaehun2841.github.io/2018/08/30/2018-08-25-spring-mvc-handle-exception/#test-case
		* https://sanghye.tistory.com/29