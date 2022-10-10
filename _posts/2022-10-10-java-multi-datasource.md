---
title:  "Java 에서 동적으로 multi dataSource 사용하기"
excerpt: "Java 에서 동적으로 multi dataSource 사용하기"

categories:
  - java
tags:
  - [java, Programming]

toc: true
toc_sticky: true
 
date: 2022-10-10
last_modified_at: 2022-10-10

published: false
---


# Intro
* 으례 그러하듯 하나의 서버에선 여러가지의 DB에 사용성에 따라 Master, Slave 에 붙어서 사용할 것이다. 
* DB각 2개(one, two)에 각각 master, slave 가 있다고 하면 설정은 아래와 비슷하게 될 것이다.

```
db:
  one:
    master:
      jdbc-url: jdbc:mysql://url:port/DB_NAME?connectTimeout=5000&useSSL=false&serverTimezone=Asia/Seoul
      user-name: id
      password: pwd
      pool-name: pool-name
      maximum-pool-size: 10
      minimum-idle: 2
      connection-timeout: 30000
      idle-timeout: 30000
    slave:
      jdbc-url: jdbc:mysql://url:port/DB_NAME?connectTimeout=5000&useSSL=false&serverTimezone=Asia/Seoul
      user-name: id
      password: pwd
      pool-name: pool-name
      maximum-pool-size: 10
      minimum-idle: 2
      connection-timeout: 30000
      idle-timeout: 30000
  two:
    master:
      jdbc-url: jdbc:mysql://url:port/DB_NAME?connectTimeout=5000&useSSL=false&serverTimezone=Asia/Seoul
      user-name: id
      password: pwd
      pool-name: pool-name
      maximum-pool-size: 10
      minimum-idle: 2
      connection-timeout: 30000
      idle-timeout: 30000
    slave:
      jdbc-url: jdbc:mysql://url:port/DB_NAME?connectTimeout=5000&useSSL=false&serverTimezone=Asia/Seoul
      user-name: id
      password: pwd
      pool-name: pool-name
      maximum-pool-size: 10
      minimum-idle: 2
      connection-timeout: 30000
      idle-timeout: 30000
```

# 구현
* 위의 설정을 사용하기 위한 살을 붙여 보자. DB 관련 공통 설정 정보를 담기 위해 아래처럼 만들었다.

```java
import lombok.Data;

@Data
public class DataSourceProperties {
    protected String jdbcUrl;
    protected String userName;
    protected String passWord;
    protected String poolName;
    protected int maximumPoolSize;
    protected int minimumIdle;
    protected int connectionTimeout;
    protected int idleTimeout;
    protected String driverClassName;
}

```
* 위의 설정을 가지고 각각의 DB에서 사용할 설정 정보를 사용하기 위해 아래처럼 조금더 만들어 보자.

```java
//OneMasterDataSourceProperties.java 파일은 아래처럼 되겠지.
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;
import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.ToString;

@Data
@ToString(callSuper=true)
@EqualsAndHashCode(callSuper=true)
@Configuration
@ConfigurationProperties(value="db.one.master")
public class OneMasterDataSourceProperties extends DataSourceProperties {
}

//OneSlaveDataSourceProperties.java 파일은 아래처럼 되겠지.
@Data
@ToString(callSuper=true)
@EqualsAndHashCode(callSuper=true)
@Configuration
@ConfigurationProperties(value="db.one.slave")
public class OneSlaveDataSourceProperties extends DataSourceProperties {
}

//TwoMasterDataSourceProperties.java 파일은 아래처럼 되겠지.
@Data
@ToString(callSuper=true)
@EqualsAndHashCode(callSuper=true)
@Configuration
@ConfigurationProperties(value="db.two.master")
public class TwoMasterDataSourceProperties extends DataSourceProperties {
}

//TwoSlaveDataSourceProperties.java 파일은 아래처럼 되겠지.
@Data
@ToString(callSuper=true)
@EqualsAndHashCode(callSuper=true)
@Configuration
@ConfigurationProperties(value="db.two.slave")
public class TwoSlaveDataSourceProperties extends DataSourceProperties {
}

```


* DB connection pool 을 위해선 잘 알려져있고 괜찮은 성능으로 많이 쓰이는 hikaricp를 사용 할 것이다. 
  * 위의 설정으로 생성하기 위해서 아래처럼 DataSourceUtil 을 만들어 보았다.

```java
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
import org.apache.commons.lang3.StringUtils;

public class DataSourceUtil {
    public static HikariDataSource createHikariDataSource(DataSourceProperties properties) {
        HikariConfig hikariConfig = new HikariConfig();
        hikariConfig.setJdbcUrl(properties.getJdbcUrl());
        hikariConfig.setUsername(properties.getUserName());
        hikariConfig.setPassword(properties.getPassWord());
        hikariConfig.setPoolName(properties.getPoolName());
        hikariConfig.setMaximumPoolSize(properties.getMaximumPoolSize());
        hikariConfig.setMinimumIdle(properties.getMinimumIdle());
        hikariConfig.setConnectionTimeout(properties.getConnectionTimeout());
        hikariConfig.setIdleTimeout(properties.getIdleTimeout());
        if (StringUtils.isNotBlank(properties.getDriverClassName())) {
            hikariConfig.setDriverClassName(properties.getDriverClassName());
        }
        return new HikariDataSource(hikariConfig);
    }
}
```

* 자 이제 준비가 되었으니 각각의 설정에 맞게 HikariDataSource 를 생성하는 로직을 아래처럼 만들어 보자.

```java
@Slf4j
@Component
public class DataSourceConfiguration {

    @Bean(name="oneMasterDataSource", destroyMethod = "close")
    public HikariDataSource oneMasterDataSource(OneMasterDataSourceProperties properties) {
        //이건 debug 보단 info 가 맞을듯.
        if (log.isDebugEnabled())
            log.debug("connect one master datasource: {}", properties);
        return DataSourceUtils.createHikariDataSource( properties );
    }

    @Bean(name="oneSlaveDataSource", destroyMethod = "close")
    public HikariDataSource oneSlaveDataSource(OneSlaveDataSourceProperties properties) {
        if (log.isDebugEnabled())
            log.debug("connect one slave datasource: {}", properties);
        return DataSourceUtils.createHikariDataSource( properties );
    }

    @Bean(name="twoMasterDataSource", destroyMethod = "close")
    @ConditionalOnExpression("!T(org.springframework.util.StringUtils).isEmpty('${db.two.master.jdbc-url:}')")
    public HikariDataSource twoMasterDataSource(twoMasterDataSourceProperties properties) {
        if (log.isDebugEnabled())
            log.debug("connect two master datasource: {}", properties);
        return DataSourceUtils.createHikariDataSource( properties );
    }

    @Bean(name="twoSlaveDataSource", destroyMethod = "close")
    @ConditionalOnExpression("!T(org.springframework.util.StringUtils).isEmpty('${db.two.slave.jdbc-url:}')")
    public HikariDataSource twoSlaveDataSource(twoSlaveDataSourceProperties properties) {
        if (log.isDebugEnabled())
            log.debug("connect two slave datasource: {}", properties);
        return DataSourceUtils.createHikariDataSource( properties );
    }
}
```
* 자자 이제 기본적인 부분은 거의 끝났다. const 성 정보를 관리하기위해 우선 아래처럼 사용해 보자.

```java
public class JdbcConstants {
    public static String MASTER = "Master";
    public static String SLAVE = "Slave";
}
```

* 이제 동적으로 필요한 부분에 알맞게 잘 사용하기 위해서 아래처럼 구현 할 것이다.
* 동적으로 쿼리 요청시마다 datasource 를 변경해야 할 때 AbstractRoutingDataSource 를 사용하면 된다. 
* AbstractRoutingDataSource는 이름 기반으로 다중 DataSource 정보를 등록하는 방법인데, AbstractRoutingDataSource 의 determineCurrentLooupKey()를 오버라이드해서 상황에 맞게 동작하도록 구현하면 된다.


```java
@Slf4j
public class ReplicationRoutingDataSource extends AbstractRoutingDataSource {

    @Override
    protected Object determineCurrentLookupKey() {
        String dataSourceType = TransactionSynchronizationManager.isCurrentTransactionReadOnly() ? JdbcConstants.SLAVE : JdbcConstants.MASTER;
        if(log.isDebugEnabled())
            log.debug("determineCurrentLookupKey datasourceType: {}", dataSourceType);
        return dataSourceType;
        
        //아래처럼 하는것보단.위가 더 괜찮을듯.
        // boolean isReadOnly = TransactionSynchronizationManager.isCurrentTransactionReadOnly();
        // if (isReadOnly) {
        //     return "Slave";
        // } else {
        //     return "Master";
        // }

    }
}

```

* 위에서처럼 readonly 에서 slave db에 붙게 하기 위해선, transactionManager 에 LazyConnectionDataSourceProxy 를 설정해 주어야 한다.
* 아래처럼 구현해 보자.

```java
@Slf4j
@Configuration
public class RoutingDataSourceConfiguration {

    @Bean(name="oneRoutingDataSource")
    @ConditionalOnBean(name={"oneMasterDataSource", "oneSlaveDataSource"})
    public DataSource oneRoutingDataSource(@Qualifier("oneMasterDataSource")DataSource masterDataSource,
                                              @Qualifier("oneSlaveDataSource")DataSource slaveDataSource) {
        return routingDataSource(masterDataSource, slaveDataSource);
    }

    @Primary
    @Bean(name="oneDataSource")
    @ConditionalOnBean(name={"oneMasterDataSource", "oneSlaveDataSource"})
    @DependsOn({ "oneMasterDataSource", "oneSlaveDataSource"})
    public DataSource oneDataSource(@Qualifier("oneRoutingDataSource") DataSource oneRoutingDataSource) {
        return new LazyConnectionDataSourceProxy(oneRoutingDataSource);
    }


    @Bean(name="twoRoutingDataSource")
    @ConditionalOnBean(name={"twoMasterDataSource", "twoSlaveDataSource"})
    public DataSource twoRoutingDataSource(@Qualifier("twoMasterDataSource") DataSource masterDataSource,
                                           @Qualifier("twoSlaveDataSource") DataSource slaveDataSource) {
        return routingDataSource(masterDataSource, slaveDataSource);
    }


    @Bean(name="twoDataSource")
    @DependsOn({ "twoMasterDataSource", "twoSlaveDataSource"})
    @ConditionalOnBean(name={"twoMasterDataSource", "twoSlaveDataSource"})
    public DataSource logDataSource(@Qualifier("twoRoutingDataSource") DataSource logRoutingDataSource) {
        return new LazyConnectionDataSourceProxy(logRoutingDataSource);
    }

    private ReplicationRoutingDataSource routingDataSource(DataSource masterDataSource, DataSource slaveDataSource) {
        ReplicationRoutingDataSource routingDataSource = new ReplicationRoutingDataSource();

        Map<Object, Object> dataSourceMap = new HashMap<>(2);
        dataSourceMap.put(JdbcConstants.MASTER, masterDataSource);
        dataSourceMap.put(JdbcConstants.SLAVE, slaveDataSource);

        routingDataSource.setTargetDataSources(dataSourceMap);
        routingDataSource.setDefaultTargetDataSource(masterDataSource);

        return routingDataSource;
    }
}
```

# 사용
* 이제 대략 아래처럼 Transactional 어노테이션에 readOnly 를 true로 한경우 slave DB 에서 동작할 것이다.

```java
@Transactional(readOnly = true)
public List<String> findStringList() {
	return testMapper.selectStringList();
}

```

* 이제 mybatis 에서 동작하도록 세세하게 알아 보겠다. 
    * 아니다 이건 다른 문서로 정리하기로.




