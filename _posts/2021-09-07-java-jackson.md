---
title:  "[java]Jackson 간략 정리"
excerpt: "Jackson 간략 정리를 해보자. "

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 2021-09-07
last_modified_at: 2021-09-07
---

## Jackson은?
* https://github.com/FasterXML/jackson
*  Java Object <-> JSON으로 상호 변환 해줌
*   XML/YAML/CSV 등 다양한 형식의 데이타도 지원

## 사용법은?
* gradle 설정은 알아서 하시고 간단 샘플을 한번 보자

### json 형태로 변환등을 할때 
* 아래와 같이 있다고 할때
    * 참고로 JSON 으로 되기 위해선, 멤버변수의 유무가 아니라 Getter, Setter를 기준으로 동작함. 걍 롬복쓰자
    * 본문 아래의 @JsonProperty 설명 추가로 보기.

```java
//lombok 사용.
@Data
@Builder
public class Sample {
  public String data1;
  public int data2;  
}
```

* data.json
```json
{
    "data1": "test",
    "data2": 17
}
```

* json 데이타를 java object 로 변환
    * File, URL, String 방식으로 데이타를 읽어올 수 있음.
```java
ObjectMapper mapper = new ObjectMapper();
.
.
Sample value = mapper.readValue(new File("data.json"), Sample.class);
//URL 에서 읽기
value = mapper.readValue(new URL("http://sample.com/data.json"), Sample.class);
//String 에서 읽기
value = mapper.readValue("{\"data1\":\"ex1\", \"data2\":100}", Sample.class);
```

* java object 를 json 으로 변환
```java
Sample sample = new Sample();
sample.data1 = "ex1";
sample.data2 = 100;

ObjectMapper mapper = new ObjectMapper();
 
// result.json 파일로 저장
mapper.writeValue(new File("result.json"), sample);
// byte[] 로 저장
byte[] jsonBytes = mapper.writeValueAsBytes(sample);
// string 으로 저장
String jsonString = mapper.writeValueAsString(sample);
```

### Spring에서 client로 JSON 형태로 응답을 줄때
* MessageConvertor
    * MessageConvertor는 스프링에서 자바 객체와 HTTP 요청 / 응답 바디를 변환하는 역할을 한다. 
* Jackson 은 json 데이터 출력을 위해 MappingJacksonHttpMessageConverter를 제공하는데, 스프링의 MessageConvertor를 MappingJacksonHttpMessageConverter 으로 등록하면 컨트롤러가 리턴객체를 ObjectMapper로 json 객체로 만들어 출력하는듯 하다? spring 3.1이후에는 Jackson 라이브러리가 존재하면 자동으로 MessageConverter가 등록이 된다. 

* 방법1
    * HttpServletResponse 객체에 문자열 쌩으로 만들어 리턴하기
        * 위에 설명한 ObjectMapper로 json 을 만들어서 넣어도 되겠지.
* 방법2
    * @ResponseBody 어노테이션을 사용하기
    * 메소드의 return 형 앞에 @ResponseBody를 붙이게 되면 해당 객체가 자동으로 json객체로 변환되어 반환됨
    * ex
    ```java
    //Controller
    @REquestMapping(value = "/test", method=requestMEthod.Get)
    public @ResponseBody TestBody get(@RequestParam("input") String input){
        .
        .
        .
    }
    ```
* 방법3
    * @RestController 어노테이션 사용하기
    * RestController는 Spring 4 에서 Rest API 등을 위해 등장한 애노테이션. 이전버전의 @Controller와 @ResponseBody를 포함한다
    * ex
    ```java
    @RestController
    @RequestMapping({"/test/v1.0/"})
    public class TestController {
        .
        .
    }
    ```



## 주요 Annotation은?
* @JsonIgnoreProperties
    *  Serializer/Deserialize 시 제외시킬 프로퍼티를 지정한다.
    ```java
    @JsonIgnoreProperties({ "foo", "bar" })
    public class MyBean
    {
       //아래 두 개는 제외됨.
       public String foo;
       public String bar;

       // will not be written as JSON; nor assigned from JSON:
       @JsonIgnore
       public String internal;

       // no annotation, public field is read/written normally
       public String external;

       @JsonIgnore
       public void setCode(int c) { _code = c; }

       // note: will also be ignored because setter has annotation!
       public int getCode() { return _code; }
    }
    ```
* @JsonProperty
    * getter/setter 의 이름을 property 와 다른 이름을 사용할 수 있도록 설정한다. Database 를 자바 클래스로 매핑하는데 DB의 컬럼명이 알기 어려울 경우등에 유용하게 사용할 수 있다.
    * 다음과 같은 테이블이 있을 경우
    ```sql
    CREATE TABLE Users (
  u INT NOT NULL,
  a INT NOT NULL,
  e VARCHAR(80) NOT NULL
    );
    ```
    * 다음과 같이 JsonProperty 를 사용하면 DB 의 컬럼명을 변경하지 않아도 가독성을 높일 수 있다.
    ```java
    public class User
    {
        @JsonProperty("userId");
        public Integer u;

        @JsonProperty("age");
        public Integer a;

        @JsonProperty("email");
        public String e;
    }
    ```
* @JsonInclude
    * Serialize 시 동작을 지정한다. 기본적으로 Jackson은 값의 유무와 상관없이 무조건 serialize 하게 되지만 다음과 같이 설정할 경우 not null 이거나 none empty 일 경우에만 serialize 된다.
    ```java
    public class User
    {
        public Integer u;

        @JsonInclude(JsonInclude.Include.NON_NULL)
        public Integer age;


        @JsonInclude(JsonInclude.Include.NON_EMPTY)
        public String email;
    }    
    ```
* JsonPropertyOrder
    * Json 직렬화 순서를 제어
    ```java
    @JsonPropertyOrder({"name", "id"})
    @Builder
    public static class PropertyOrder {
        private long id;
        private String name;
    }
    ```
    * 적용전
    ```json
    {
      "id": 1,
      "name": "name"
    }
    ```
    * 적용후
     ```json
    {
      "name": "name",
      "id": 1
    }
    ```


