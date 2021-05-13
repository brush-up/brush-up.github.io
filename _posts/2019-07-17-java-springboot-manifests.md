---
title:  "[java]springboot에서 manifests에 읽고 쓰기 예제"
excerpt: "springboot에서 manifests에 읽고 쓰기 예제"

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 2019-07-17
last_modified_at: 2019-07-17
---

# MANIFEST.MF에 쓰기
* build.gradle 파일에 아래처럼 하면
```
bootWar {
    manifest {
        attributes 'Implementation-Title': project.name
        attributes 'Implementation-Version': version
        attributes 'Implementation-Timestamp': new Date()
    }
}
```
* MANIFEST.MF에 아래 처럼 정보가 추가된다.
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2019-07-17-java-springboot-manifests-001.png)


# MANIFEST.MF에서 읽기

* before

```java
    public static void main(String[] args) {
        SpringApplication.run(WebuiApplication.class, args);
    }
```
* after
    * build.gradle에 아래내용 추가

    ```
    compile group: 'com.jcabi', name: 'jcabi-manifests', version: '1.1'
    ```
    * source code고치기

    ```java
    import com.xxx.properties.ApplicationProperties;
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.scheduling.annotation.EnableScheduling;

    import org.springframework.context.ApplicationContext;
    import org.springframework.context.ConfigurableApplicationContext;
    import com.jcabi.manifests.Manifests;
    import java.util.Arrays;
    import java.util.Date;

        public static void main(String[] args) {
            runServer(WebuiApplication.class, args);
        }
        
        protected static ApplicationContext runServer(Class<?> source, String[] args) {
            // spring-boot 기동
            ConfigurableApplicationContext context = null;
            try {
                // - 해당 정보는 jar 파일로 실행될 때, manifest.mf 파일을 참고하기 때문에 local 실행시에는 null
                String applicationVersion = source.getPackage().getImplementationVersion();

                // spring boot run
                SpringApplication springApplication = new SpringApplication(source);
                context = springApplication.run(args);

                // applicationProperties 객체 획득
                Object object = context.getBean(ApplicationProperties.class);
                ApplicationProperties applicationProperties = (ApplicationProperties) object;

                // 서버 버전 정보 설정
                applicationProperties.setVersion(applicationVersion);

                // Manifests 파일에서 timestamp 정보를 획득
                // - 해당 정보는 jar 파일로 실행될 때, manifest.mf 파일을 참고하기 때문에 local 실행시에는 null
                if (Manifests.exists("Implementation-Timestamp"))
                    applicationProperties.setPackageCreatedTime(Manifests.read("Implementation-Timestamp"));

                // 서버 기동시 설정한 Active Profile
                String[] activeProfiles = context.getEnvironment().getActiveProfiles();
                if (activeProfiles != null)
                    applicationProperties.setActiveProfiles( Arrays.asList(activeProfiles) );

                // 서버 기동 시간 저장
                applicationProperties.setBootTime(new Date(context.getStartupDate()));

                // 서버 기동 완료 설정
    //            applicationProperties.setStartup(true);

                // 출력해보자.
                log.info("{}", applicationProperties.toString());
            } finally {
                if (context == null) 
					log.error("start fail!");
            }
            return context;
        }        
    ```

    ```java
		package com.xxx.webui.properties;

		import com.fasterxml.jackson.annotation.JsonFormat;
		import com.fasterxml.jackson.annotation.JsonIgnore;
		import lombok.*;
		import lombok.extern.slf4j.Slf4j;
		import org.springframework.boot.context.properties.ConfigurationProperties;
		import org.springframework.stereotype.Component;

		import java.util.Arrays;
		import java.util.Collections;
		import java.util.Date;
		import java.util.List;
		import java.util.concurrent.atomic.AtomicBoolean;

		@Slf4j
		@Getter
		@Setter
		@ConfigurationProperties(value = "app")
		@Component
		public class ApplicationProperties {

			@JsonIgnore
			@Setter(AccessLevel.NONE)
			@Getter(AccessLevel.NONE)
			public final Object shutdownMonitor = new Object();

			// set from application.yaml
			private String name;
			private String description;

			// set from main class
			private String version;

			@Getter(AccessLevel.NONE)
			private List<String> activeProfiles;

			private String packageCreatedTime;

			@Setter(AccessLevel.NONE)
			private final AtomicBoolean startup = new AtomicBoolean(false);

			@JsonIgnore
			@Getter(AccessLevel.NONE)
			@Setter(AccessLevel.NONE)
			private AtomicBoolean shutdown;

			@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "Asia/Seoul")
			private Date bootTime;

			public List<String> getActiveProfiles() {
				if (activeProfiles != null) return activeProfiles;

				String value = System.getProperty("spring.profiles.active");
				log.info("getActiveProfiles : {}" , value);

				return value != null ? Arrays.asList(value.split(",")) : Collections.emptyList();
			}

			public void setStartup(boolean status) {
				this.startup.set(status);
			}

			// 종료 시점에서만 호출 하자.
			public void setShutdown() {
				if (shutdown == null) {
					synchronized (this.shutdownMonitor) {
						log.info("Setting shutdown variable to true");
						shutdown = new AtomicBoolean(true);
						shutdownMonitor.notifyAll();
					}
				}
			}

			@JsonIgnore
			public boolean hasRealProfile() {
				return getActiveProfiles().stream().anyMatch(v -> v.contains("real"));
			}
			...
		}
    ```
# actuate를 통해 rest api 로 정보 확인하기

```java
package com.xxx.actutaor;

import com.xxx.properties.ApplicationProperties;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.actuate.info.InfoContributor;
import org.springframework.boot.actuate.info.InfoEndpoint;
import org.springframework.boot.actuate.info.MapInfoContributor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;


import java.util.ArrayList;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;


//SpringBoot 에서 자동으로 생성되던, infoEndpoint 객체를 직접 생성하기
@Slf4j
@Configuration
public class InfoConfiguration {
    private final Map<String, Object> infoMap = new LinkedHashMap<String, Object>();

    @Autowired
    private ApplicationProperties applicationProperties;

    @Bean
    public InfoEndpoint infoEndpoint() {

        addInfo("app", applicationProperties);
		//addInfo 를 통해 더 필요한 정보 넣자.

        List<InfoContributor> result = new ArrayList<>();
        result.add(new MapInfoContributor(infoMap));

        return new InfoEndpoint(result);
    }

    public void addInfo(String key, Object value) {
        infoMap.put(key, value);
    }
}
```