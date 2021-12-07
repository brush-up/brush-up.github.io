---
title:  "[java]Java Spring @Transactional 어노테이션 관해"
excerpt: "Java Spring @Transactional 어노테이션 관해"

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 2021-12-05
last_modified_at: 2021-12-05
---



## intro
* DB 등의 트랜잭션 처리를 위해 사용되는 어노테이션 이다.
* 일일이 프로그래밍으로 작업하던 것을 좀 쉽게 해준다. 
    * db connection을 얻은이후 일일이 transaction begin, commit 해주고 특정 예외 발생시 rollback해주던 부분을 자동으로 손쉽게 해주니 편하긴 하다.

## 사용법
* @Transactional은  class 나  method 에 선언해주면 된다.
    * class에 선언해주면 해당 class에 속하는 method에 자동 적용되고, method에 적용하면 해당 mthod에만 적용된다.

* class 단위 설정(test1, test2에 적용된다.)

```java
@Service
@Transactional
public class TestClass {
	public void test1() {
        ...
	}
	public void test2() {
		...
	}
}
```
* method 단위 설정 (test1에만 적용된다.)

```java
@Service
public class TestClass {
	@Transactional
	public void test1() {
        ...
	}
	public void test2() {
        ...
	}
}
```


### 우선순위
* @Transactional는 아래와 같이 우선순위를 가진다
    * (높음 )메소드 > class > 인터페이스 메소드 > 인터페이스 (낮음)
        * 공통적인 트랜잭션 규칙은 클래스에, 특별한 규칙은 메서드에 선언하면 좋다.
* 자바 어노테이션은 인터페이스로부터 상속되지 않기 때문에  클래스 기반 프록시 나 AspectJ 기반에서 트랜잭션 설정을 인식 할 수 없다. 인터페이스 보다는 클래스에 적용하자


## AOP
* AOP는 일반적으로 아래와 같은 방식이 있다고함.
    * JDK Dynamic Proxy 방식
        * 다이나믹 프록시 객체는 타깃이 상속하고있는 인터페이스를 상속 후 추상메서드를 구현하며 내부적으로 타깃 메서드 호출 전 후로 특정 동작을 하게함
        * JDK Dynamic Proxy 방식은 Java의 리플렉션 패키지에 존재하는 Proxy 클래스 통해 동적으로 다이나믹 프록시 객체를 생성한다.
    * CGLib 방식
        * 타깃오브젝트가 인터페이스를 상속하고 있지 않는 다면 CGLib를 사용하여 인터페이스 대신 타깃오브젝트를 상속하는 프록시 객체를 만든다.
        * JDK Proxy와는 달리 리플렉션을 사용하지 않고 바이트코드 조작을 통해 런타임 시점에 프록시 객체를 생성하는 방식
    * AspectJ 방식
        * 프록시처럼 간접적인방법이 아닌 컴파일 시점에 컴파일된 타깃 클래스파일 자체를 수정하거나 클래스가 JVM에 로딩되는 시점을 가로채서 바이트코드를 수정하여 부가기능이 비즈니스 로직과 같이 있는것처럼 만들어버린다.

* Transactional는 Spring AOP를 통해 구현
    * 클래스, 메소드에 @Transactional이 선언되면 해당 클래스에 트랜잭션이 적용된 프록시 객체 생성
    * 프록시 객체는 @Transactional이 포함된 메서드가 호출될 경우, 트랜잭션을 시작하고 Commit or Rollback을 수행
    * CheckedException or 예외가 없을 때는 Commit 수행
    * UncheckedException이 발생하면 Rollback 수행

## 기타

### 트랜잭션의 모드
* @Transactional은 Proxy Mode와 AspectJ Mode가 있는데 Proxy Mode가 Default로 설정되어있다.
* Proxy Mode는 다음과 같은 경우 동작하지 않는다.
    * 반드시 public 메서드에 적용되어야한다.
        * Protected, Private Method에서는 선언되어도 에러가 발생하지는 않지만, 동작하지도 않는다.
        * Non-Public 메서드에 적용하고 싶으면 AspectJ Mode를 고려해야한다.
    * @Transactional이 적용되지 않은 Public Method에서 @Transactional이 적용된 Public Method를 호출할 경우, 트랜잭션이 동작하지 않는다.

## 속성 옵션
### readOnly
* readOnly 설정을 주면 transactionManager 에 읽기전용 hint 가 적용이 된다고한다
* hibernate 에서 readOnly 설정이 들어갈 경우 트랜잭션 commit 시, flush 를 하지 않습니다.
    * 정확히는 session 의 flush_mode 가 FLUSH_MANUAL 로 변경
    * flush 하는 resource + flush 를 안함으로써 dirty checking 을 안하는 resource 가 제거되니 성능 향상이 있습니다.
* mysql에선 이렇게 동작하려나?

```
set session transaction isolation level read uncommitted ;
select * from table ;
set session transaction isolation level repeatable read ;
```
* 밴더마다 다르게 동작할수 있다는것 유의해야한다. mysql에서 @Transactional(readOnly = true)가 적용된 메서드에서 @Transactional 혹은 @Transactional(readOnly = false)가 적용된 메서드를 호출 할 경우 무조건 read-only Transaction이 적용된지만, R을 제외한 CUD를 호출 할 경우 에러가 발생할수도있고 아닐수도 있다.
* 사용예

```java
@Transactional(readOnly = true)
``` 

### 예제
* 아래 예제처럼 여러가지 옵션들이 있다. 정리하려다 보니.. 귀찮 일단,,, 나중에 하자. 

```java
    @Transactional(propagation = Propagation.REQUIRES_NEW, noRollbackFor = Custom001Exception.class)
    public void test_mtehod() {
        ...
    }
    
    @Transactional(readOnly = false, propagation = Propagation.REQUIRES_NEW)
    public void method() {
        ...
    }
    
```

### 이부분 더 봐야함.
* @transactional (readonly= true) 메서드가 @transactional (ReadOnly= false) 메소드를 호출하면 어떻게 될까?
    * 이전 트랜잭션이 지속되기 때문에  (readonly= true) 지속 될 것이고, 이 작업을 수행하려면 아래처럼 신규 트랜잭션으로 실행해야 한다 @Transactional(propagation= Propagation.REQUIRES_NEW). 
        * 하지만. 이거 잘 동작하지 않는거 같다. 먼자 잘못설정했나? 다시 한번 확인이 필요하다.

## 다중 Transaction Manager
### 정리필요
* 필요에 따라서 다수의 독립된 트랜잭션 매니져를 사용할 수 있다.
* 이는 @Transactional의 value 속성에 사용할 PlatformTransactionlManager를 지정하면 된다.
* 참고로 PlatformTransactionManager는 Spring에서 제공하는  트랜잭션 관리 인터페이스이다.
* 인터페이스의 구현체에는 JDBC, 하이버네이트 등이 있는데 이는 트랜잭션이 실제 작동할 환경이다.
* 따라서 반드시 알맞은 PlatformTransactionlManager 구현체가 정의되어  있어야한다.
* 다시 본론으로 돌아와, 다중 트랜잭션 매니저는 다음과 같이 지정한다.
value 속성에 Bean의 id or qualifier 값을 지정한다. 
아래는 qualifier를 이용한 Spring Docs의 예제이다.

```java
public class TransactionalService {
    @Transactional("order")
    public void setSomething(String name) { ... }
    @Transactional("account")
    public void doSomething() { ... }
}
```

```xml
<bean id="transactionManager1" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	...
	<qualifier value="order"/>
</bean>

<bean id="transactionManager2" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	...
	<qualifier value="account"/>
</bean>
```


## 정말 기타
### cache
* seleclt -> update -> select 테스트를 하다보니 계속 동일한 값이 나오길래 확인해보니
    * Mybatis local session cache 문제로 동일한 쿼리문을 반복적으로 호출했기 때문에 계속 같은값을 부르고 있었다.

* 아래처럼 flushCache를 true로 useCache를 false로 해주면 캐싱문제는 해결된다.
    * 왜 둘다 바꿔야하지? 차이가 머지?
    * 확인해보자.
* before

```xml
    <select id="select" parameterType="HashMap" statementType="PREPARED" resultType="com.xxx.entity.XXXEntity">
        SELECT user_id...
        WHERE app_id = #{appId}
    </select>
```
* after

```xml
    <select id="select" parameterType="HashMap" statementType="PREPARED" resultType="com.xxx.entity.XXXEntity" flushCache="true" useCache="false">
        SELECT user_id...
        WHERE app_id = #{appId}
    </select>
```

* 일단 여기까지만.. 정리할것들이 많다.

## 참고
* https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html
* https://docs.spring.io/spring-framework/docs/4.2.x/spring-framework-reference/html/transaction.html
* https://hwannny.tistory.com/98
* https://imiyoungman.tistory.com/9
* https://goddaehee.tistory.com/167
* https://techblog.woowahan.com/2606/
* 옵션등은 이 자료를.
    * https://www.baeldung.com/spring-transactional-propagation-isolation