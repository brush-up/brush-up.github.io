---
title:  "[java] Spring Task Scheduler 가이드"
excerpt: "Spring Task Scheduler 가이드"

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 2022-01-14
last_modified_at: 2022-01-14
---


## 1. intro
* Spring task scheduling 메커니즘을 살짝 알아보자
* Spring의 스케줄링에 대해 조금더 알고 싶으면 아래를 더 보자
    * https://www.baeldung.com/spring-async
    * https://www.baeldung.com/spring-scheduled-tasks

## 2. ThreadPoolTaskScheduler
* ThreadPoolTaskScheduler 는 작업을 ScheduledExecutorService에 위임 하고 TaskExecutor 인터페이스를 구현하므로 내부 스레드 관리에 적합하다고하는데..
 * 아래의 예시를 보자 (ThreadPoolTaskSchedulerConfig 에서 ThreadPoolTaskScheduler 빈을 정의 한다.)

```java
@Configuration
@ComponentScan(
  basePackages="com.baeldung.taskscheduler",
  basePackageClasses={ThreadPoolTaskSchedulerExamples.class})
public class ThreadPoolTaskSchedulerConfig {

    @Bean
    public ThreadPoolTaskScheduler threadPoolTaskScheduler(){
        ThreadPoolTaskScheduler threadPoolTaskScheduler
          = new ThreadPoolTaskScheduler();
        threadPoolTaskScheduler.setPoolSize(5);
        threadPoolTaskScheduler.setThreadNamePrefix(
          "ThreadPoolTaskScheduler");
        return threadPoolTaskScheduler;
    }
}
```
* pool size 5를 기반으로 구성된 bean threadPoolTaskScheduler는 비동기 작업을 실행할수있다
* 모든 ThreadPoolTaskScheduler 관련 스레드 이름에는 ThreadPoolTaskScheduler 접두사가 붙도록 했다.
* 이제, 예약할수 있는 간단한 작업을 구현해 보자

```java
class RunnableTask implements Runnable{
    private String message;
    
    public RunnableTask(String message){
        this.message = message;
    }
    
    @Override
    public void run() {
        System.out.println(new Date()+" Runnable Task with "+message
          +" on thread "+Thread.currentThread().getName());
    }
}
```
* 이제 이 작업이 스케줄러에 의해 실행되도록 아래처럼 간단하게 예약할 수 있다.

```java
taskScheduler.schedule(
  new Runnabletask("Specific time, 3 Seconds from now"),
  new Date(System.currentTimeMillis + 3000)
);
```
* taskScheduler는 정확히 현재 시간으로부터 3초후에 작업을 시작하도록 스케줄이 돌도록 하였다.

---- 

이제 ThreadPoolTaskScheduler 스케줄링 메커니즘 에 대해 좀 더 자세히 알아보자
## 3. Schedule Runnable Task With Fixed Delay
* 고정 지연 일정은 두 가지 간단한 메커니즘으로 수행할 수 있다.

### 3.1. Scheduling After a Fixed Delay of the Last Scheduled Execution
* 1000밀리초의 고정 지연 후에 실행되도록 작업을 구성해 보자

```java
taskScheduler.scheduleWithFixedDelay(
  new RunnableTask("Fixed 1 second Delay"), 1000);
```
* RunnableTask는 이전 작업을 완료하고  1000 milliseconds 이후에 실행된다. 

### 3.2. Scheduling After a Fixed Delay of a Specific Date
* 지정된 시작 시간의 고정 지연 후에 실행되도록 작업을 구성해 보겠습니다.


```java
taskScheduler.scheduleWithFixedDelay(
  new RunnableTask("Current Date Fixed 1 second Delay"),
  new Date(),
  1000);
```

*  @PostConstruct 가 시작하고 이어서 1000 밀리 초 지연 후 RunnableTask가 실행된다.
  

## 4. Scheduling at a Fixed Rate
* 고정된 주기로 실행 가능한 작업을 예약하기 위한 두 가지 간단한 메커니즘이 있습니다

### 4.1. Scheduling the RunnableTask at a Fixed Rate
* 고정된 밀리초 속도 로 실행되도록 작업을 예약해 보겠습니다 .

```java
taskScheduler.scheduleAtFixedRate(
  new RunnableTask("Fixed Rate of 2 seconds") , 2000);
```
* 이전 RunnableTask 가 실행중이던지 말던지 관계없이 항상 2000밀리초 후에 RunnableTask가  실행됩니다.


### 4.2. Scheduling the RunnableTask at a Fixed Rate From a Given Date

```java
taskScheduler.scheduleAtFixedRate(new RunnableTask(
  "Fixed Rate of 2 seconds"), new Date(), 3000);
```
* RunnableTask는 현재 시간 이후에 3000 밀리 초 뒤에 를 실행 작업을 실행한다


## 5. Scheduling with CronTrigger
* CronTrigger 는 cron 표현식을 기반으로 작업을 예약하는 데 사용된다. 아래 예제를 보자.

```java
CronTrigger cronTrigger 
  = new CronTrigger("10 * * * * ?");
```

* 제공된 트리거를 사용하여 지정된 특정 주기 또는 일정에 따라 작업을 실행할 수 있습니다.

```java
taskScheduler.schedule(new RunnableTask("Cron Trigger"), cronTrigger);
```
* 이 경우 RunnableTask 는 매분 10초에 실행됩니다.

## 6. Scheduling with PeriodicTrigger


* 2000밀리초의 고정 지연 으로 작업을 예약하기 위해 PeriodicTrigger 를 사용하겠습니다 .

```java
PeriodicTrigger periodicTrigger 
  = new PeriodicTrigger(2000, TimeUnit.MICROSECONDS);
```

* 구성된 PeriodicTrigger 빈은 2000밀리초의 고정 지연 후에 작업을 실행하는 데 사용됩니다.
* 이제 PeriodicTrigger를 사용 하여 RunnableTask 를 예약해 보겠습니다 .


```java
taskScheduler.schedule(
  new RunnableTask("Periodic Trigger"), periodicTrigger);
```


* 또한 PeriodicTrigger 가 고정 지연이 아닌 고정 속도로 초기화되도록 구성 할 수 있으며 첫 번째 예약된 작업에 대해 주어진 밀리초 단위로 초기 지연을 설정할 수도 있습니다.

* 우리가 해야 할 일은 periodTrigger 빈 에서 return 문 앞에 두 줄의 코드를 추가하는 것입니다 .


```java
periodicTrigger.setFixedRate(true);
periodicTrigger.setInitialDelay(1000);
```


* setFixedRate 메서드를 사용하여 고정 지연이 아닌 고정 속도로 작업을 예약한 다음 setInitialDelay 메서드를 사용하여 실행할 첫 번째 실행 가능한 작업에 대해서만 초기 지연을 설정합니다.



## 7. 실전 예제

### 7.1 

```java
@Data
@Component
@ConfigurationProperties(value="scheduler-pool")
public class SchedulerPoolProperties {
	
	private int poolSize = 5;
	
	private boolean waitForTasksToCompleteOnShutdown = false;
	
	private int awaitTerminationSeconds = 0;
}
```

```java
@RequiredArgsConstructor
@Configuration
@EnableScheduling
public class SchedulerPoolConfiguration implements SchedulingConfigurer {

	private final SchedulerPoolProperties schedulerPoolProperties;
	
	@Bean
	public ThreadPoolTaskScheduler threadPoolTaskScheduler() {
		ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();
		threadPoolTaskScheduler.setPoolSize( schedulerPoolProperties.getPoolSize() );
		threadPoolTaskScheduler.setWaitForTasksToCompleteOnShutdown( true );
		threadPoolTaskScheduler.setAwaitTerminationSeconds( schedulerPoolProperties.getAwaitTerminationSeconds() );
		threadPoolTaskScheduler.setDaemon(true);
		threadPoolTaskScheduler.setThreadNamePrefix( "test-scheduler-" );
		
		return threadPoolTaskScheduler;
	}
	
	@Override
	public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
		taskRegistrar.setTaskScheduler(threadPoolTaskScheduler());
	}
}
```

### 7.2

```java
@Data
@Component
@ConfigurationProperties(value="async-pool")
public class AsyncPoolProperties {
	
	private int corePoolSize = 16;
	private int maxPoolSize = 200;
	private int keepAliveSeconds = 60;
	private int queueCapacity = 1000;
	private int awaitTerminationSeconds = 10;
}

```


```java

// http://www.baeldung.com/spring-async
@RequiredArgsConstructor
@Configuration
@EnableAsync
public class AsyncPoolConfiguration implements AsyncConfigurer {

	private final AsyncPoolProperties asyncPoolProperties;

	@Override
    public Executor getAsyncExecutor() {
	    
        return asyncThreadPoolTaskExecutor();
    }

    /**
     * core 개수가 차면, queue 에 넣고
     * queue 가 다 차면 maxQueue 사이즈만큼 늘린다.
     *
     * @return
     */
	@Bean
    public ThreadPoolTaskExecutor asyncThreadPoolTaskExecutor() {
	    ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize( asyncPoolProperties.getCorePoolSize() ); // 실행할 최소 Thread 수.
        taskExecutor.setMaxPoolSize( asyncPoolProperties.getMaxPoolSize() ); // 최대 Thread 지원수
        taskExecutor.setQueueCapacity( asyncPoolProperties.getQueueCapacity() );
        taskExecutor.setKeepAliveSeconds( asyncPoolProperties.getKeepAliveSeconds() ); // 현재 풀에 corePoolSize 의 수보다 많은 thread가 있는 경우, 초과한 만큼의 thread는, IDLE 상태가 되어 있는 기간이 keepAliveTime 를 넘으면(자) 종료
        taskExecutor.setWaitForTasksToCompleteOnShutdown(true);
        taskExecutor.setAwaitTerminationSeconds( asyncPoolProperties.getAwaitTerminationSeconds() );
        taskExecutor.setDaemon(true);
        taskExecutor.setThreadNamePrefix( "test-async-" );
        taskExecutor.initialize();
        
        return taskExecutor;
    }
    
    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
    	
        return new SimpleAsyncUncaughtExceptionHandler();
    }
}
```

## 참조
* https://www.baeldung.com/spring-task-scheduler
* https://www.baeldung.com/spring-async
* https://www.baeldung.com/spring-scheduled-tasks

