---
title:  "[작성중][java]CompletableFuture"
excerpt: "java CompletableFuture 참고용 정리"

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 2020-09-28
last_modified_at: 2020-09-28

published: true
---


# intro
* thread, Future, Promise, lamdba 등의 기본지식은 있어야 하는듯
* https://brush-up.github.io/java/java8-lambda-functional-interface/  를 먼저 선행해야함.
* future, promises 
    * https://en.wikipedia.org/wiki/Futures_and_promises 참고하자
    * Future 
        * a read-only reference(읽기 전용) to a yet-to-be-computed value
        * 이벤트 발생 시 사용할 수(읽을 수) 있는 어떤 값
    * Promise 
        * pretty much the same except that you can write to it(쓰기 가능)
        *  연산의 결과를 넣어두고(쓰고) 이 결과 값을 읽을 수 있는 Future를 얻을 수 있습니다. 즉, Promise가 끝나고 나면 성공, 혹은 실패에 연결되어 있는 Future를 통해 특정 동작을 유발시킬 수 있다.
    * 공통점은 yet-to-be-computed value 아직 계산되지 않은 값에 대한 레퍼런스 라는 점

* Future
    ```java
    package java.util.concurrent;

    public interface Future<V> {
        boolean cancel(boolean var1);
        boolean isCancelled();
        boolean isDone();
        V get() throws InterruptedException, ExecutionException;
        V get(long var1, TimeUnit var3) throws InterruptedException, ExecutionException, TimeoutException;
    }
    ```
* Future < V > 는 언젠가 사용 가능해지는 타입 V 의 값
    * read-only 하다
* Promise[ V ] 는 언젠가 얻게 될 타입 V의 값
    * writable 하다 인듯.
  
  
#  sync, async
### 동기방식
* 아래와 같이 시간이 많이 걸리는 method가 있을때 
```java
.
.
.
public static void main(String[] args){
    TestInterface test = new TestInterface() {
        @Override
        public String TEST_Sync(String name) {
            System.out.println(getCurrentTime() + " TEST_Sync: thread: " + Thread.currentThread().getName());
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return name;
        }
.
.
.
System.out.println(getCurrentTime() + "======== before TEST_Sync ========");
test.TEST_Sync("test");
System.out.println(getCurrentTime() + "======== after TEST_Sync ========");
```
결과값은 당연히 아래처럼 2초가 걸려 다음 스텝으로 넘어간다.
```
[10:38:25] ======== before TEST_Sync ========
[10:38:25]  TEST_Sync: thread: main
[10:38:27] ======== after TEST_Sync ========
```
### 비동기 방식
이러한 호출을 비동기로 하기 위해서 이전까지는 아래처럼 내부적으로 thread 하나를 생성해서 동작하도록 하면

```java
@Override
public CompletableFuture<String> TEST_ASync1(String name) {
    CompletableFuture<String> future1 = new CompletableFuture<>();
    new Thread(() -> {
        String output = TEST_Sync(name);
        future1.complete(output);
    }).start();;
    return future1;
}
.
.
.
System.out.println(getCurrentTime() + "======== before TEST_ASync1 ========");
CompletableFuture<String> future = (CompletableFuture<String>) test.TEST_ASync1("test2");
System.out.println(getCurrentTime() + "======== after1 TEST_ASync1 ========");
String aaa = future.join();//blocking
System.out.println(getCurrentTime() + "======== after2 TEST_ASync1 ========");
```

결과 값은 당연히 시간 지연 없이 바로 응답을 받게 된다. 물론 결과 최종 데이터를 조회하기 위해 CompletableFuture의 join이나 get 을 수행할때 blocking 이 되는데...

```
[10:38:27] ======== before TEST_ASync1 ========
[10:38:27] ======== after1 TEST_ASync1 ========
[10:38:27]  TEST_Sync: thread: Thread-0
[10:38:29] ======== after2 TEST_ASync2 ========
```
### 위의 비동기 method를 좀더 깔끔하게 해보자
CompletableFuture 에 제공하는 supplyAsync, runAync 라는 것이 있다.


supplyAsync 는 Supplier Functional Interface를 파라미터로 받아 
Supplie는 인자는 받지 않고 리턴 타입만 존재
https://brush-up.github.io/java/java8-lambda-functional-interface/

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```
```java
//supplyAsync
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
    return asyncSupplyStage(ASYNC_POOL, supplier);
}

public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor) {
    return asyncSupplyStage(screenExecutor(executor), supplier);
}
```

runAync 는 Runnable Functional Interface를 파라미터로 받아 
```java
@FunctionalInterface
public interface Runnable {
    void run();
}
```
```java
public static CompletableFuture<Void> runAsync(Runnable runnable) {
    return asyncRunStage(ASYNC_POOL, runnable);
}
//runAync
public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor) {
    return asyncRunStage(screenExecutor(executor), runnable);
}
```


이 기본지식을 가지고 아래처러 supplyAsync 를 이용해서 호출하면
```java
@Override
public CompletableFuture<String> TEST_ASync2(String name) {

    return CompletableFuture.supplyAsync(() -> {
        System.out.println(getCurrentTime() + " thread: " + Thread.currentThread().getName());
        return TEST_Sync(name);
    });
}
.
.
.
System.out.println(getCurrentTime() + "======== before TEST_ASync2 ========");
CompletableFuture<String> future2 = (CompletableFuture<String>) test.TEST_ASync2("test2");
System.out.println(getCurrentTime() + "======== after1 TEST_ASync2 ========");
String aaa2 = future2.join();//blocking
System.out.println(getCurrentTime() + "======== after2 TEST_ASync2 ========");
```
결과값은
```
[11:52:27] ======== before TEST_ASync2 ========
[11:52:27] ======== after1 TEST_ASync2 ========
[11:52:27]  thread: ForkJoinPool.commonPool-worker-3
[11:52:27]  TEST_Sync: thread: ForkJoinPool.commonPool-worker-3
[11:52:29] ======== after2 TEST_ASync2 ========
```
결과값을 보면 ForkJoinPool 를 이용하는데, supplyAsync를 실행할때 아래처럼 Executor를 파라미터로 주면 별도의 thread pool에서 동작하게 한다.

```java
static Executor executor1 = Executors.newFixedThreadPool(3);
.
.
.
//user? thread pool
@Override
public CompletableFuture<String> TEST_ASync3(String name) {

    return CompletableFuture.supplyAsync(() -> {
        System.out.println(getCurrentTime() + " thread: " + Thread.currentThread().getName());
        return TEST_Sync(name); },
            executor1
    );
}
.
.
.
System.out.println(getCurrentTime() + "======== before TEST_ASync3 ========");
CompletableFuture<String> future3 = (CompletableFuture<String>) test.TEST_ASync3("test2");
System.out.println(getCurrentTime() + "======== after1 TEST_ASync3 ========");
String aaa3 = future3.join();//blocking
System.out.println(getCurrentTime() + "======== after2 TEST_ASync3 ========");
```
결과 값을 보면. 별도의 thread pool를 이용하는것을 볼수있따.
```
[11:52:29] ======== before TEST_ASync3 ========
[11:52:29] ======== after1 TEST_ASync3 ========
[11:52:29]  thread: pool-1-thread-1
[11:52:29]  TEST_Sync: thread: pool-1-thread-1
[11:52:31] ======== after2 TEST_ASync3 ========
```

### 좀더 개선해 보자
CompletableFuture의 get, join 메소드를 사용하면 순간 blocking이 발생하는데 이를 개선해보자.
이제 blocking 현상을 제거하기 위해. CompletableFuture의 thenApply, thenAccept 를 이용해서 조금더 수정해보자.

thenAccept  는 CompletableFuture < U > 를 반환한다.(결과반환이 없다.) 
thenApply는 CompletableFuture < T > 데이터를 포함하는 Future를 반환한다.

```java
public <U> CompletableFuture<U> thenApply(Function<? super T, ? extends U> fn) {
    return this.uniApplyStage((Executor)null, fn);
}

public <U> CompletableFuture<U> thenApplyAsync(Function<? super T, ? extends U> fn) {
    return this.uniApplyStage(this.defaultExecutor(), fn);
}

public <U> CompletableFuture<U> thenApplyAsync(Function<? super T, ? extends U> fn, Executor executor) {
    return this.uniApplyStage(screenExecutor(executor), fn);
}

public CompletableFuture<Void> thenAccept(Consumer<? super T> action) {
    return this.uniAcceptStage((Executor)null, action);
}

public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action) {
    return this.uniAcceptStage(this.defaultExecutor(), action);
}

public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action, Executor executor) {
    return this.uniAcceptStage(screenExecutor(executor), action);
}
```


thenAccept method를 이용해서 코드 작성해보자.
```java
//use thenAccept
@Override
public void TEST_ASync4(String name) {

    System.out.println(getCurrentTime() + "======== TEST_ASync4 1========");
    CompletableFuture<Void> future = TEST_ASync3(name).thenAccept(p->
        {
            System.out.println(getCurrentTime() + " thread: " + Thread.currentThread().getName() + "TEST_ASync4 : call back1" + p);
        });
    System.out.println(getCurrentTime() + "======== TEST_ASync4 2========");
}

```
* 결과값
```
[09:26:42] ======== before TEST_ASync4 ========
[09:26:42] ======== TEST_ASync4 1========
[09:26:42] ======== TEST_ASync4 2========
[09:26:42] ======== after TEST_ASync4 ========
[09:26:42]  thread: pool-1-thread-1
[09:26:42]  TEST_Sync: thread: pool-1-thread-1
[09:26:44]  thread: pool-1-thread-1TEST_ASync4 : call back1test2
```



```java
//use thenApply
@Override
public void TEST_ASync5(String name) {

    System.out.println(getCurrentTime() + "======== TEST_ASync5 1========");
    CompletableFuture<Void> future = TEST_ASync3(name)
            .thenApply(p->{
                System.out.println(getCurrentTime() + " thread: " + Thread.currentThread().getName() + "TEST_ASync5 call back1: " + p); return p;
            })
            .thenAccept(p->{
                System.out.println(getCurrentTime() + " thread: " + Thread.currentThread().getName() + "TEST_ASync5 call back2: " + p);
            });

    System.out.println(getCurrentTime() + "======== TEST_ASync5 2========");
}

```

* 결과값
```
[09:28:24] ======== before TEST_ASync5 ========
[09:28:24] ======== TEST_ASync5 1========
[09:28:24] ======== TEST_ASync5 2========
[09:28:24] ======== after TEST_ASync5 ========
[09:28:24]  thread: pool-1-thread-1
[09:28:24]  TEST_Sync: thread: pool-1-thread-1
[09:28:26]  thread: pool-1-thread-1TEST_ASync5 call back1: test2
[09:28:26]  thread: pool-1-thread-1TEST_ASync5 call back2: test2
```

* 위와 마찬가지로 별로의 thread pool을 이용하고 싶으면 thenAcceptAsync, thenApplyAsync 를 이용하면 된다. 

* `추가로 확인해야 함`
    * https://dzone.com/articles/be-aware-of-forkjoinpoolcommonpool
    * https://c10106.tistory.com/5214
    * https://www.baeldung.com/java-fork-join
    * https://m.blog.naver.com/PostView.nhn?blogId=tmondev&logNo=220945933678&proxyReferer=https://www.google.com/
    * https://blog.naver.com/2feelus/220732310413
    * 
* 참고
    https://brunch.co.kr/@springboot/267
















.