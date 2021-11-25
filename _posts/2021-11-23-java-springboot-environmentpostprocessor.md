---
title:  "[java]JSpringboot 의 EnvironmentPostProcessor 를 이용해보자"
excerpt: "Springboot 의 EnvironmentPostProcessor 를 이용해 보자"

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 2021-11-23
last_modified_at: 2021-11-23
---

## 시작하기
* 특정 내용별로 설정파일을 분리하고 싶거나 프로그램 시작전에 설정파일에 대해 무언가 처리를 하려고 할때 사용하면 좋다.
* custom 하게 설정을 관리하고 싶으면 사용하자


## 사용법
* META-INF/spring.factories 파일에 다음 내용 추가.
```
org.springframework.boot.env.EnvironmentPostProcessor=com.example.YourEnvironmentPostProcessor
```

* 코드 샘플 

```java
public class EnvironmentPostProcessorExample implements EnvironmentPostProcessor {

 private final YamlPropertySourceLoader loader = new YamlPropertySourceLoader();

    @Override
    public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
        addPropertySource(environment, "test0.yaml");    
        addPropertySource(environment, "sample/test1.yaml");
        addPropertySource(environment, "sample/test2.yaml");
    }

    private void addPropertySource(ConfigurableEnvironment environment, String path) {
        Resource resource = new ClassPathResource(path);
        List<PropertySource<?>> propertySource = loadYaml(resource);
        for (PropertySource<?> source : propertySource) {
            environment.getPropertySources().addLast(source);
        }
    }

    private List<PropertySource<?>> loadYaml(Resource resource) {
        log.info("LoadYaml : {}", resource);
        if (!resource.exists()) {
            throw new IllegalArgumentException("Resource " + resource + " does not exist");
        }

        try {
            return this.loader.load(resource.getFilename(), resource);
        } catch (IOException ex) {
            throw new IllegalStateException("Failed to load yaml configuration from " + resource, ex);
        }
    }

}
```

* 각 Properties class와 properties를 만들어 보자. 
    * sample/test1.yaml, sample/test2.yaml 샘플은 포함안함.

```java
@Data
@Component
@ConfigurationProperties(value="sample-test1")
@Validated
public class Test0Properties {

    @NotNull
    private Map<String, List<PushMessage>> messages;

    public List<JsonMessage> getMessage(String key) {
        return messages.get(key);
    }
}

```
* test0.yaml 내용 
```
sample-test1:
  messages:
    jsonMessage:
      - {a: "Y", b: "bb", c: "", d: ""}
      - {a: "N", b: "bb", c: "", d: ""}
      - {a: "N", b: "bb", c: "", d: ""}
```
* JsonMessage class 샘플
```java
@NoArgsConstructor
@AllArgsConstructor
@Builder
@Data
public class JsonMessage {

    @NotEmpty
    private String b;
    @NotEmpty
    private String c;
    @NotEmpty
    private String d;
    private BooleanFlag a;
}
```


## 참고
* https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.application.customize-the-environment-or-application-context
* https://www.baeldung.com/spring-boot-environmentpostprocessor
* https://www.baeldung.com/spring-boot-custom-auto-configuration