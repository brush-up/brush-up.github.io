---
title:  "java, mybatis 사용하기 샘플"
excerpt: "java, mybatis 사용하기 샘플"

categories:
  - java
tags:
  - [java, Programming]

toc: true
toc_sticky: true
 
date: 2022-10-11
last_modified_at: 2022-10-11

published: false
---

# Intro
* mybatis 에 대한 기본적인 내용은 잘 알겠거니 하고 아래와 같은 설정으로 mybatis 를 사용해 보도록 하겠다.
    * 각 설정에 대한 내용은 아래를 참고해서 보기
        * https://mybatis.org/mybatis-3/ko/configuration.html

```
# https://mybatis.org/mybatis-3/ko/configuration.html 참고하기.
mybatis:
  properties: 
    cacheEnabled: true
    lazyLoadingEnabled: true
    aggressiveLazyLoading: true
    useGeneratedKeys: true
    defaultExecutorType: SIMPLE
    defaultStatementTimeout: 3 # seconds
    mapUnderscoreToCamelCase: true
```

# 구현
* 위의 설정을 사용하기 위한 살을 붙여 보자. 
    * MybatisProperties.java, MybatisConfigBuilder.java 를 아래와 같이 해서 관련 설정을 사용하도록 해보자.
    * 우선 기본적인 설정값들은 아래값을 default 로..

```java
@Data
@Component
@ConfigurationProperties(value="mybatis")
public class MybatisProperties {

    private Map<String, String> properties = new HashMap<>();

```

```java
//...
import org.apache.ibatis.builder.BaseBuilder;
import org.apache.ibatis.executor.loader.ProxyFactory;
import org.apache.ibatis.session.*;
import org.apache.ibatis.type.JdbcType;
import java.util.Properties;

@Slf4j
public class MybatisConfigBuilder extends BaseBuilder {

    public MybatisConfigBuilder(Configuration configuration, Properties props) {
        super(configuration);

        try {
            settingsElement(configuration, props);
        } catch (Exception e) {
            log.error("Exception : {}", e);
            System.exit(-1);
        }
    }

    /**
     * refer to {@link org.apache.ibatis.builder.xml.XMLConfigBuilder#settingsElement(Properties)}
     */
    private void settingsElement(Configuration configuration, Properties props) {
        configuration.setAutoMappingBehavior(AutoMappingBehavior.valueOf(props.getProperty("autoMappingBehavior", "PARTIAL")));
        configuration.setAutoMappingUnknownColumnBehavior(AutoMappingUnknownColumnBehavior.valueOf(props.getProperty("autoMappingUnknownColumnBehavior", "NONE")));
        configuration.setCacheEnabled(booleanValueOf(props.getProperty("cacheEnabled"), true));
        configuration.setProxyFactory((ProxyFactory) createInstance(props.getProperty("proxyFactory")));
        configuration.setLazyLoadingEnabled(booleanValueOf(props.getProperty("lazyLoadingEnabled"), false));
        configuration.setAggressiveLazyLoading(booleanValueOf(props.getProperty("aggressiveLazyLoading"), false));
        configuration.setMultipleResultSetsEnabled(booleanValueOf(props.getProperty("multipleResultSetsEnabled"), true));
        configuration.setUseColumnLabel(booleanValueOf(props.getProperty("useColumnLabel"), true));
        configuration.setUseGeneratedKeys(booleanValueOf(props.getProperty("useGeneratedKeys"), false));
        configuration.setDefaultExecutorType(ExecutorType.valueOf(props.getProperty("defaultExecutorType", "SIMPLE")));
        configuration.setDefaultStatementTimeout(integerValueOf(props.getProperty("defaultStatementTimeout"), null));
        configuration.setDefaultFetchSize(integerValueOf(props.getProperty("defaultFetchSize"), null));
        configuration.setMapUnderscoreToCamelCase(booleanValueOf(props.getProperty("mapUnderscoreToCamelCase"), false));
        configuration.setSafeRowBoundsEnabled(booleanValueOf(props.getProperty("safeRowBoundsEnabled"), false));
        configuration.setLocalCacheScope(LocalCacheScope.valueOf(props.getProperty("localCacheScope", "SESSION")));
        configuration.setJdbcTypeForNull(JdbcType.valueOf(props.getProperty("jdbcTypeForNull", "OTHER")));
        configuration.setLazyLoadTriggerMethods(stringSetValueOf(props.getProperty("lazyLoadTriggerMethods"), "equals,clone,hashCode,toString"));
        configuration.setSafeResultHandlerEnabled(booleanValueOf(props.getProperty("safeResultHandlerEnabled"), true));
        configuration.setDefaultScriptingLanguage(resolveClass(props.getProperty("defaultScriptingLanguage")));
        configuration.setDefaultEnumTypeHandler(resolveClass(props.getProperty("defaultEnumTypeHandler")));
        configuration.setCallSettersOnNulls(booleanValueOf(props.getProperty("callSettersOnNulls"), false));
        configuration.setUseActualParamName(booleanValueOf(props.getProperty("useActualParamName"), true));
        configuration.setReturnInstanceForEmptyRow(booleanValueOf(props.getProperty("returnInstanceForEmptyRow"), false));
        configuration.setLogPrefix(props.getProperty("logPrefix"));
        configuration.setConfigurationFactory(resolveClass(props.getProperty("configurationFactory")));
    }
}
```

* 위의 설정값들을 사용하기 위해서 아래처럼 구현 하였다. 

```java
//...
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import javax.sql.DataSource;
import java.util.Properties;

@Slf4j
public class MybatisUtil {
    public static Properties mybatisProperties(MybatisProperties mybatisProperties) {
        log.info("MyBatis properties:{}", mybatisProperties.getProperties());

        Properties properties = new Properties();
        properties.putAll(mybatisProperties.getProperties());

        return properties;
    }

    public static SqlSessionFactory sqlSessionFactoryBean(DataSource dataSource, MybatisProperties mybatisProperties) throws Exception {
        MybatisConfigBuilder configBuilder = new MybatisConfigBuilder(new org.apache.ibatis.session.Configuration(), mybatisProperties(mybatisProperties));
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        sqlSessionFactoryBean.setConfiguration(configBuilder.getConfiguration());
        sqlSessionFactoryBean.setFailFast(false);
        return sqlSessionFactoryBean.getObject();
    }
}
```

* 이제 * /src/main/resources/com/test 하위에 xml 파일을 위치 (설정에 맞게 잘.)







```java
//OneMybatisConfiguration.java 는 아래처럼
@Slf4j
@Configuration
@MapperScan(basePackages = {
        "com.test.*.persistence.one",
        "com.test.*.*.persistence.one",
        "com.test.*.*.*.persistence.one"
}, sqlSessionFactoryRef="oneSqlSessionFactory")
@EnableTransactionManagement
@ConditionalOnExpression("!T(org.springframework.util.StringUtils).isEmpty('${db.one.master.jdbc-url:}')")
public class OneMybatisConfiguration {
    @Primary
    @Bean(name = "oneSqlSessionFactory")
    public SqlSessionFactory oneSqlSessionFactory(@Qualifier("oneDataSource") DataSource oneDataSource
            , MybatisProperties mybatisProperties) throws Exception {
        log.info("one sql session factory created");
        return MybatisUtils.sqlSessionFactoryBean(oneDataSource, mybatisProperties);
    }
}

//TwoMybatisConfiguration.java 는 아래처럼 하면 될듯.
@Slf4j
@Configuration
// scan for mappers and let them be autowired (mapper interface를 Spring Bean으로 등록)
@MapperScan(basePackages = {
        "com.test.*.persistence.two",
        "com.test.*.*.persistence.two",
        "com.test.*.*.*.persistence.two"
}, sqlSessionFactoryRef="twoSqlSessionFactory")
@EnableTransactionManagement
@ConditionalOnExpression("!T(org.springframework.util.StringUtils).isEmpty('${db.two.master.jdbc-url:}')")
public class LogMybatisConfiguration {

    @Bean(name = "twoSqlSessionFactory")
    public SqlSessionFactory logSqlSessionFactory(
            @Qualifier("twoDataSource") DataSource logDataSource,
            MybatisProperties mybatisProperties) throws Exception {
        log.info("two sql session factory created");
        return MybatisUtils.sqlSessionFactoryBean(logDataSource, mybatisProperties);
    }
}
```




# 사용

* Entity 는 아래처럼 만들어 보자.

```java
import java.util.Date;

@Data
public class TestInfo {

    @JsonInclude(JsonInclude.Include.NON_NULL)
    protected String aa;
    @JsonInclude(JsonInclude.Include.NON_NULL)
    protected String bb;
    protected Date regDate;
}
```

```java
@ToString(callSuper = true)
@NoArgsConstructor
public class TestEntity extends TestInfo {
    public TestEntity(String aa, String bb, Date regDate){
        this.aa = aa;
        this.bb = bb;
        this.regDate = regDate;
    }
}
```


* 이제 마지막으로 TestMapper.java 을 아래처럼 작성해서 사용해 보자.

```java
import com.test.xxx.xxx.db.entity.TestEntity;
import org.apache.ibatis.annotations.Param;

public interface TestMapper {
    AppEntity selectTestByKey(@Param("key")String key);
}
```


```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.test.xxx.xxx.db.persistence.one.TestMapper">

    <select id="selectAppByAppKey" statementType="PREPARED" resultType="com.test.xxx.xxx.db.entity.TestEntity">
        SELECT aa, bb, regDate
        FROM test
        WHERE key = #{key}
    </select>

</mapper>
```

