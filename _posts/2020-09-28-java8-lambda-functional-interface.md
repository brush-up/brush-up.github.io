---
title:  "[java]java8 lambda, functional interface 관련 정리 "
excerpt: "java8 lambda, functional interface 관련 정리"

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 2020-09-28
last_modified_at: 2020-09-28
---

## lambda
* 프로그래밍에서 익명함수를 나타냄.
* 인라인으로 정의된 함수유형이다
    * java에는 인라인함수가 없음. jvm에서 런타임 시점에 인라인으로 수행될수는 있음
    * java9에서 컴파일 단계에 최적화를 수행하는 Ahead of time라는게 있기는 함
        * [https://www.baeldung.com/ahead-of-time-compilation](https://www.baeldung.com/ahead-of-time-compilation)
* 이름이 없는 함수이다(익명함수)
* 언어별로 표현법은 다르지만, first-class citizen(일급객체)이다.
    * 일급객체
        * 함수의 인자로 넘겨받을수도있고, 함수의 결과값으로 리턴할수도있고, 변수에 값을 할당할수도있다
* 그러면 함수포인터랑 차이는 머지?
    * 함수포인터는 인라인은 아니니..
* 예제
    * C++

    ``` c++
    //ex1
    template<typename F>

    void Eval( const F& f ) {
            f();
    }
    void foo() {
            Eval( []{ printf("Hello, Lambdas\n"); } );
    }
    ...
    //ex2
    void bar() {
        auto f = []{ printf("Hello, Lambdas\n"); };
        f();
    }
    ```
    * java

    ``` java
    Predicate<Integer> pred = x -> x % 2 == 0; // Tests if the parameter is even.
    boolean result = pred.test(4); // true
    ```

* [https://en.wikipedia.org/wiki/Lambda\_calculus](https://en.wikipedia.org/wiki/Lambda_calculus)를 참고해보자

## java8 lambda

* 람다 표현식
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2020-09-28-java8-lambda-functional-interface-001.png)
    * [https://www.geeksforgeeks.org/lambda-expressions-java-8/](https://www.geeksforgeeks.org/lambda-expressions-java-8/)
* 기존에는 아래처럼 사용해서 구현을 했었다.

``` java
public interface MathInterface{
    void math(int a, int b);
}
.
.
MathInterface test = new MathInterface() {
    @Override
    public void math(int a, int b) {
        System.out.println("");
    }
};
test.math(1,2);
```

* 이것을 람다형식으로 변형하면

``` java
//lambda식 : 매개변수 O, 리턴 X
MathInterface test2 = (a, b) -> System.out.println("");
test2.math(2, 1);
```

* 아래처럼 생략할수도있다

``` java
public interface PrintInterface{
    void print(int a);
}
.
.
//아래처럼도 가능
PrintInterface test3 = (a) -> {System.out.println("");};
test3.print(3);

//하나의 매개변수만 존재하면 {} 를 생략할수 있음.
PrintInterface test4 = a -> System.out.println("");
test4.print(3);

//람다식이 하나의 실행문이라면 ()도 생략 가능하다.
PrintInterface test5 = (a) -> System.out.println("");
test5.print(3);
```

## functional interface
* java 에서 기본적으로 제공하는 functional interface 들은 아래와 같은것들이 있다.
```
Runnable
Supplier<T>
Consumer<T>
Function<T, R>
Predicate<T>
UnaryOperator<T>
BinaryOperator<T>
BiPredicate<T, U>
BiConsumer<T, U>
BiFunction<T, U, R>
Comparator<T>
```

* 참고
    * `Functional Interface 어노테이션`
        * 함수형 인터페이스에 Functional Interface 어노테이션이 필수는 아니지만, 이 어노테이션은 컴파일 단계에서 mehtod가 하나만 선언되어있는지 확인해준다.
### Runnable
* 매개변수 없고, 리턴값도 없다.

``` java
package java.lang;

@FunctionalInterface
public interface Runnable {
    void run();
}
```
* 람다 사용하지 않고 사용시

``` java
Runnable task01 = new Runnable() {
    @Override
    public void run() {
        System.out.println("================lambda================");
    }
};
Thread thread01 = new Thread(task01);
thread01.start();
```
* 람다 사용했을시

``` java
Runnable task02 = () ->  System.out.println("================lambda================");
Thread thread02 = new Thread(task02);
thread02.start();
```

### Supplier < T >
* 인자는 받지 않고 리턴 타입만 존재한다.

``` java
package java.util.function;

@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```
* 람다 사용 예시

``` java
Supplier<String> supplier01 = () -> "test";
System.out.println("================lambda================" + supplier01.get());
```
   
###  Consumer < T >
* T 타입의 인자는 받고, 리턴은 하지 않는다
* andThen이라는 디폴트 메서드를 제공한다.

``` java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T var1);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (t) -> {
            this.accept(t);
            after.accept(t);
        };
    }
}
```
* 람다 사용 예시

``` java
Consumer<String> consumer = (s) -> System.out.println("=====lambda================" + s);
consumer.accept("");
    ```
```

###  Function < T, R >
* 매개변수가 있고, 리턴 값도 있는 경우이다.

``` java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T var1);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (v) -> {
            return this.apply(before.apply(v));
        };
    }

    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (t) -> {
            return after.apply(this.apply(t));
        };
    }

    static <T> Function<T, T> identity() {
        return (t) -> {
            return t;
        };
    }
}
```
* 람다 예시
    * 매개변수로 전달받은 String을 대문자로 변환해주는 아주 간단한 기능의 함수를 작성해

``` java
    Function<String, String> function = (s)-> s.toUpperCase();
    System.out.println("================lambda================" + function.apply("abc"));
```

### Predicate < T >
* T 타입의 매개변수를 받고, boolean를 리턴한다. Function\<T, Boolean> 와 같은 기능을 한다고 생각해도 된다

``` java
    Predicate<String> predicate = (s)->s.startsWith("e");
    System.out.println("================lambda================" + predicate.test("abc"));
```

## Cumtom Functional Interface 사용하기

``` java
//JAVA 제네릭(Generic) 도 알아야 할듯?
@FunctionalInterface
public interface TestInterFace4<T, R> {
    R apply(T t);
}

//Cumtom Functional Interface 사용
public static Integer getTest1(String a, TestInterFace4<String, Integer> b){
    return b.apply(a);
}

//자바 제공 Functional Interface 사용
//TestInterFace4 를 Function 으로 대체.
public static Integer getTest2(String a, Function<String, Integer> b){
    return b.apply(a);
}
//https://nhnent.dooray.com/project/posts/2789822747730797200   규칙 18번 이후의 내용?  기본으로 제공하는 함수형 인터페이스를 사용하는 것을 권장한다
.
.
.
//Cumtom Functional Interface 사용
//custom 으로 하기 예제.
Integer a = getTest1("abc", new TestInterFace4<String, Integer>() {
    @Override
    public Integer apply(String s) {
        return s.length();
    }
});
//람다 형식으로
Integer b = getTest1("abc", s -> s.length());
//자바 제공 Functional Interface 사용 : custom functional interface사용법과 같음.
Integer c = getTest2("abc", s->s.length());
```

## ETC

* 디컴파일 하면?
    * 결과가 큰 의미는 없어보여 생략
* Java Generic 도 정리해야할듯...

## 참고

[https://en.wikipedia.org/wiki/Functional\_programming](https://en.wikipedia.org/wiki/Functional_programming)
[https://en.wikipedia.org/wiki/Lambda\_calculus](https://en.wikipedia.org/wiki/Lambda_calculus)
[https://www.geeksforgeeks.org/lambda-expressions-java-8/](https://www.geeksforgeeks.org/lambda-expressions-java-8/)
[https://brunch.co.kr/@springboot/276](https://brunch.co.kr/@springboot/276)


