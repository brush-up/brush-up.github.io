---
title:  "[java]SpringBoot 의 @ConfigurationProperties  간략 정리"
excerpt: "SpringBoot 의 @ConfigurationProperties 간략 정리를 해보자. "

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 2021-09-17
last_modified_at: 2021-09-17
---



## intro
* Spring Boot는 외부 설정에 정의된 속성에 대해 쉽게 사용할수있도록 @ConfigurationProperties 어노테이션을 제공한다. 

## useage
### 설정
* maven 이나 gradle 설정은 알아서 잘 하자 
* spring-boot-starter-parent 가 필요하고, 속성값에 대한 검증을 하려면 hibernate-validator 도 필요로 하다. 
    * http://hibernate.org/validator/documentation/getting-started/
### 기본설정 및 바인딩 룰
* 우선 샘플로 하나 봐보자.
```java
@Configuration
@ConfigurationProperties(prefix = "mail")
public class ConfigProperties {
    
    private String hostName;
    private int port;
    private String from;

    // standard getters and setters
}
```
* @Configuration 어노테이션을 주어 bean 을 생성하도록 하자
* @ConfigurationProperties 는 동일한 접두사가 있는 경우 계층적으로 사용할수있도록 해준다. 
    * 아래처럼 되어있으면
    ```java
    @ConfigurationProperties(prefix = "mail.a")
    ...
    @ConfigurationProperties(prefix = "mail.b")
    ```
    * 아래처럼 설정 가능하다
    ```
    mail:
      a: xxxxxx 
      b: xxxxxx 
    ```

* Spring framework는 표준 Java bean setter를 사용하므로 각 속성에 대한 setter를 선언해야 한다.
    * POJO에서 @Configuration 을 사용하지 않으면 기본 Spring 애플리케이션 클래스에 @EnableConfigurationProperties(ConfigProperties.class) 를 추가 하여 속성을 POJO에 바인딩해야 합니다.
```java
@SpringBootApplication
@EnableConfigurationProperties(ConfigProperties.class)
public class EnableConfigurationDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(EnableConfigurationDemoApplication.class, args);
    }
}
```
* Spring은 ConfigProperties 클래스 의 필드 중 하나와 동일한 이름과 접두사 mail 이 있는 속성 파일에 정의된 모든 속성을 자동으로 바인딩합니다 . 바인딩 규칙은 상당히 루즈하다. 아래와같은 변형 모두 바인딩이 된다.
```
mail.hostName
mail.hostname
mail.host_name
mail.host-name
mail.HOST_NAME
```

### spring boot 2.2이후
* spring boot 2.2부터는 @ConfigurationProperties 를 사용하면 굳이 @Component나 @EnableConfigurationProperties를 사용할 필요가 없어졌다.  @ConfigurationProperties 만 사용하면 알아서 스캔한다.
* 이제 아래처럼 해도 된다.
    ```java
    @ConfigurationProperties(prefix = "mail") 
    public class ConfigProperties { 

        private String hostName; 
        private int port; 
        private String from; 

        // standard getters and setters 
    }
    ```
    
### 중첩속성
* 아래같은 경우

```java
public class Credentials {
    private String authMethod;
    private String username;
    private String password;
    // standard getters and setters
}

public class ConfigProperties {
    private String from;
    private List<String> defaultRecipients;
    private Map<String, String> additionalHeaders;
    private Credentials credentials;
    // standard getters and setters
}
```
* 아래와 같은 설정이 된다.

```
#Simple properties
mail.hostname=mailer@mail.com
mail.port=9000
mail.from=mailer@mail.com

#List properties
mail.defaultRecipients[0]=admin@mail.com
mail.defaultRecipients[1]=owner@mail.com

#Map Properties
mail.additionalHeaders.redelivery=true
mail.additionalHeaders.secure=true

#Object properties
mail.credentials.username=john
mail.credentials.password=password
mail.credentials.authMethod=SHA1
```

###  속성 검증
* @ConfigurationProperties 는 JSR-303 형식을 사용하여 속성의 유효성 검사를 제공
* 유효성 검사 중 하나라도 실패하면 기본 응용 프로그램이 IllegalStateException 과 함께 시작되지 않음
* Hibernate Validation 프레임워크는 표준 Java Bean getter 및 setter를 사용하므로 각 속성에 대해 getter 및 setter를 선언해야함
* 필수 속성 지정 
```java
@NotBlank
private String hostName;
```

* 속성을 1~4자 길이로 만들어 보기
```java
@Length(max = 4, min = 1)
private String authMethod;
```

* 1025에서 65536까지로 제한하기
```java
@Min(1025)
@Max(65536)
private int port;
```

* 이메일 주소 형식과 일치해야하는 속성 부여
```java
@Pattern(regexp = "^[a-z0-9._%+-]+@[a-z0-9.-]+\\.[a-z]{2,6}$")
private String from;
```
.

### 불변 @ConfigurationProperties 바인딩
* Spring Boot 2.2부터 @ConfigurationProperties 이 달린 클래스가 변경 불가능하도록 @ConstructorBinding 어노테이션을 사용하여 구석 속성을  바인딩할 수 있음.
* @ConstructorBinding을 사용할 때  바인딩하려는 모든 매개변수를 생성자에 제공해야 함.
* ImmutableCredentials의 모든 필드  final 이고 setter 메서드가 없음
* 생성자 바인딩을 사용하려면 @EnableConfigurationProperties  또는  @ConfigurationPropertiesScan 을 사용하여 구성 클래스를 명시적으로 활성화해야함.

```java
@ConfigurationProperties(prefix = "mail.credentials")
@ConstructorBinding
public class ImmutableCredentials {

    private final String authMethod;
    private final String username;
    private final String password;

    public ImmutableCredentials(String authMethod, String username, String password) {
        this.authMethod = authMethod;
        this.username = username;
        this.password = password;
    }

    public String getAuthMethod() {
        return authMethod;
    }

    public String getUsername() {
        return username;
    }

    public String getPassword() {
        return password;
    }
}
```




### Java16 에선
* Java 16 은 JEP 395의 일부로 레코드  유형을 도입했습니다  . 레코드는 변경할 수 없는 데이터의 투명한 전달자 역할을 하는 클래스입니다. 따라서 구성 보유자 및 DTO에 대한 완벽한 후보입니다. 사실 Spring Boot에서 Java 레코드를 구성 속성으로 정의할 수 있습니다 . 예를 들어 이전 예제는 다음과 같이 다시 작성할 수 있습니다.

```java
@ConstructorBinding
@ConfigurationProperties(prefix = "mail.credentials")
public record ImmutableCredentials(String authMethod, String username, String password) {
}
```

* 또한 Spring Boot 2.6 부터 단일 생성자 레코드의 경우 @ConstructorBinding  주석을 삭제할 수 있습니다  . 그러나 레코드에 여러 생성자가 있는 경우 @ConstructorBinding 을 사용하여 속성 바인딩에 사용할 생성자를 식별해야 합니다.

### 참고용 예시들
* 아래와 같은 설정은
```
external-server:
  targets:
    notice:
      domain: https://naver.com
      context: notice
```
* 이렇게 하면 쉽게 이용 가능
```java
@Data
@Component
@ConfigurationProperties(value="external-server")
@Validated
public class TargetProperties {
    private static final String DOMAIN = "domain";
    private static final String CONTEXT = "context";

    @NotNull
    private Map<String, Map<String, String>> targets;

    public String getDomain(final String productId) {
        Map<String, String> productMap = targets.get(productId);

        return productMap != null ? productMap.get(DOMAIN) : null;
    }

    public String getContext(final String productId) {
        Map<String, String> productMap = targets.get(productId);

        return productMap != null ? productMap.get(CONTEXT) : null;
    }

    /**
     * yaml에 선언된 product id 획득
     * notice / ...
     */
    public List<String> getProductIds() {
        Map<String, Map<String, String>> products = this.targets;
        List<String> ids = new ArrayList<>(products.size());

        products.forEach((k,v) -> ids.add(k));

        return ids;
    }
}
```

* 아래처럼도 이용 가능
```
game-info:
  games:
    AA: infoA
    BB: infoB
  adUrl:
    AA: URLA
    BB: URLB
```

```java
@Data
@Component
@ConfigurationProperties(value="game-info")
@Validated
public class GameInfoProperties {

    @NotNull
    private Map<String, String> games;

    @NotNull
    private Map<String, String> adUrl;

    public String getGameString(final String gameId) {
        return games.get(gameId);
    }

    public String getAdUrl(final String gameId) {
        return adUrl.get(gameId);
    }
}

```

## 참고
* https://www.baeldung.com/configuration-properties-in-spring-boot