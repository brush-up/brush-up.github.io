---
title:  "[java] 어노테이션 정리1 (Lombok 관련)"
excerpt: "Java 어노테이션 정리1 (Lombok 관련)"

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 2022-03-10
last_modified_at: 2022-03-10
---

# intro
* 처음 제일 적응이 안되는 것이 annotation 이었다. 코드가 없는데, 실행이 되는데, 어떻게 되는지도 모르겠고 그러했던 기억들
* 문서 작성할때 한번에 해야지, 미루다 보면 더이상 적기 싫어진다.. 망..

# Lombok 관련 annotation
## @NonNull
* Lombok 에서 null check 로직을 자동으로 생성해주는 annotation
* 생성자, 메소드 특정 파라미터에 @NonNull이 달려있으면 Lombok 은 자동적으로 해당 파라미터에 대해 null check 코드를 생성함.
* 이경우에 null check 코드는 메소드나 생성자의 최상단에 위치
* 또, 필드에 @NonNull이 달려있으면 해당 필드에 값을 설정하는 메소드들에도 null check 코드를 생성
* @NonNull을 파라미터에 사용한 경우 lombok 사용한 코드와 순수 자바코드는 아래와 같음
* @NonNull을 파라미터에 사용했을 경우 예시
* Lombok annotated code:

```java
import lombok.NonNull;
public class NonNullExample extends Something {
    private String name;
    public NonNullExample(@NonNull Person person){
        super("Hello");
        this.name = person.GetName();
    }
}
```
* Equivalent Java source code:

```java
public class NonNullExample extends Something {
    private String name;
    public NonNullExample(@NonNull Person person){
        super("Hello");
        if(person == null){
            throw new NullPointerException("person is marked @NonNull but is null");
        }
        this.name = person.GetName();
    }
}
```


* @NonNull을 필드에 사용한 경우 예시.

```java
@Getter
@Setter
public class LombokTest{
    @NonNull
    private String test;
}
```
* Equivalent Java source code:

```java
public class LombokTest{
    @NonNull
    private String test;

    public LombokTest(){}

    @NonNull
    public String getTest(){
        return this.test;
    }

    public void setTest(@NonNull String test){
        if(test == null){
            throw new NullPointerException("test is makred non-null but is null");
        } else {
            this.test = test;
        }
    }
}
```

* 여기서 주의할점은 필드에 적용된 @NonNull 어노테이션은 lombok이 생성한 메소드나 생성자에만 효과가 있다 것이다.
* 만약 아래와 같이 lombok에게 메소드 생성을 맡기지 않고 직접 코드를 작성한다면

```java
public class LombokTest{
    @NonNull
    private String test;

    public LombokTest(){}
    public String getTest(){
        return this.test;
    }

    public void setTest(String test){
        this.test = test;
    }
}
```

* 아래와 같이 @NonNull 이 적용되지 않는다
```java
public class LombokTest{
    @NonNull
    private String test;

    public String getTest(){
        return test;
    }

    public void setTest(String test){
        this.test = test;
    }
}
```


* @NonNull 설정값
    * null값이 들어왔을 때 exception이 기본은 NullPointerException이 발생하지만
    * lombok.nonNull.exceptionType 설정값을 IllegalArgumentException으로 변경할 수 있습니다.

## @Getter, @Setter
* 필드에 @Getter나 @Setter를 붙인다면, lombok이 해당 필드에 대한 기본 getter/setter를 생성해준다.
* 기본적인 getter란 단순히 필드를 리턴하는 것을 말하며, 필드 이름이 foo라면 메소드 이름은 getFoo가 되고, 만약 필드 타입이 boolean이라면 isFoo가 된다.
* 기본적인 setter는 필드 이름이 foo라면 메소드 이름은 setFoo이며, void를 리턴타입으로 가지고, 필드와 같은 타입의 파라미터를 하나 가집니다. 단순히 필드에 값을 설정해주는 setter입니다.

```java
public class LombokTest {
    @Getter
    @Setter
    private String test1;
}
```

```java
public class LombokTest {
    
    private String test1;
    
    public String getTest1(){
        return test1;
    }
    public void setTest1(String test1){
        this.test1 = test1;
    }
}
```

* 만약 생성되는 getter/setter에 명시적으로 AccessLevel을 명시해주지 않으면, 접근 제한자는 public이 된다.
* 허용되는 access level들은 PUBLIC, PROTECTED, PACKAGE, PRIVATE가 있다.

```java
public class LombokTest {
    @Getter
    private String test1;
    
    @Getter(AccessLevel.PRIVATE)
    private String test2;    
}
```

```java
public class LombokTest {

    private String test1;
    private String test2;    
    
    public String getTest1(){
        return this.test1;
    }
    private String getTest2(){
        return this.test2;
    }    
}
```

* 만약 필드에 @Getter나 @Setter를 붙이지 않고 클래스에 붙인다면, static이 아닌 전체 필드에 @Getter와 @Setter 애노테이션이 적용된다.
* 그리고 만약 특정 필드에서 @Getter, @Setter의 생성을 막고 싶다면 AccessLevel.None을 사용하면 된다.  AccessLevel.None으로 값을 설정하면 해당 필드는 lombok이 메소드를 생성하지 않는다.

* Lombok annotated code:

```java
@Getter
public class LombokTest {
    
    private String test1;
    
    @Getter(AccessLevel.NONE)
    private String test2;
    
    public static String test3;
}
```

* Equivalent Java source code:

```java
public class LombokTest {

    private String test1;
    private String test2;    
    public static String test3; 
    
    public LombokTest(){
    }
    
    public String getTest1(){
        return this.test1;
    } 
}
```
* sataic 필드, None적용한 필드 모드 getter가 생성되지 않는다. 


## @NoArgsConstructor
* 생성자를 자동으로 생성해주는 애노테이션
* 이것은 파라미터가 없는 생성자를 생선한다

* Lombok annotated code:

```java
@NoArgsConstructor
public class LombokTest{
    private String test1;
    public LombokTest(String test1){
        this.test1 = test1;
    }
}
```

* Equivalent Java source code:

```java
public class LombokTest{
    private String test1;
    public LombokTest(String test1){
        this.test1 = test1;
    }
    public LombokTest(){
    }
}
```

* 필드들이 final로 생성되어 있는 경우에는 필드를 초기화 할 수 없기 때문에 생성자를 만들 수 없고 에러가 발생하게 됩니다.
이 때는 @NoArgsConstructor(force = true) 옵션을 이용해서 final 필드를 0, false, null 등으로 초기화를 강제로 시켜서 생성자를 만들 수 있습니다.

* @NonNull 같이 필드에 제약조건이 설정되어 있는 경우, 생성자내 null-check 로직이 생성되지 않습니다.
후에 초기화를 진행하기 전까지 null-check 로직이 발생하지 않는 점을 염두하고 코드를 개발해야 합니다.

## @RequiredArgsConstructor
* 생성자를 자동으로 생성해주는 어노테이션
* 이 애노테이션은 추가 작업을 필요로 하는 필드에 대한 생성자를 생성하는 애노테이션입니다.
* 초기화 되지 않은 모든 final 필드, @NonNull로 마크돼있는 모든 필드들에 대한 생성자를 자동으로 생성해줍니다.
* @NonNull로 마크돼있는 필드들은 null-check가 추가적으로 생성되며, @NonNull이 마크되어 있지만, 파라미터에서 null값이 들어온다면 생성자에서 NullPointerException이 발생합니다.
* 파라미터의 순서는 클래스에 있는 필드 순서에 맞춰서 생성됩니다.

* Lombok annotated code:

```java
@RequiredArgsConstructor
public class LombokTest{
    private String test1;
    private final String test2;
    @NonNull
    private String test3;
}
```

* Equivalent Java source code:

```java
public class LombokTest{
    private String test1;
    private final String test2;
    @NonNull
    private String test3;
    
    public LombokTest(String test2, @NonNull String test3){
        if(test3 == null){
            throw new NullPointerException("test3 is marked non-null but is null");
        }else{
            this.test2 = test2;
            this.test3 = test3;
        }
    }
}
```
* final 변수와 @NonNull 마크 변수에 해당하는 생성자가 생성된 것을 알수있다.
* 그리고 null-check 로직이 자동적으로 생성된 것도 볼 수 있다.


## @AllArgsConstructor
* 생성자를 자동으로 생성해주는 애노테이션
* 이 어노테이션은 클래스에 존재하는 모든 필드에 대한 생성자를 자동으로 생성해 준다.
* 만약 필드중에서 @NonNull 애노테이션이 마크되어 있다면 생성자 내에서 null-check 로직을 자동적으로 생성해 준다.

* Lombok annotated code:

```java
@AllArgsConstructor
public class LombokTest{
    private String test1;
    private String test2;
    @NonNull
    private String test3;
}
```

* Equivalent Java source code:
```java
public class LombokTest{
    private String test1;
    private String test2;
    @NonNull
    private String test3;
    
    public LombokTest(String test1, String test2, @NonNull String test3){
        if(test3 == null){
            throw new NullPointerException("test3 is marked non-null but is null");
        }else{
            this.test1 = test1;
            this.test2 = test2;
            this.test3 = test3;
        }
    }
}
```

* staticName 옵션을 이용한 static factory 메소드 생성
* 위의 세 생성자 관련 애노테이션은 static factory를 만들 수 있는 옵션이 있습니다.
* staticName이라는 옵션을 사용하여 생성자를 private으로 생성하고, private 생성자를 감싸고 있는 static factory 메소드를 추가할 수 있습니다.
* 아래와 같이 옵션을 사용할 수 있습니다.
    * ex) @RequiredArgsConstructor(staticName = “of”)

* Lombok annotated code:

```java
@RequiredArgsConstructor(staticName = "of")
public class LombokTest{
    private String test1;
    private final String test2;
    @NonNull
    private String test3;
}
```

* Equivalent Java source code:

```java
public class LombokTest{
    private String test1;
    private final String test2;
    @NonNull
    private String test3;
    
    private LombokTest(String test2, @NonNull String test3){
        if(test3 == null){
            throw new NullPointerException("test3 is marked non-null but is null");
        }else{
            this.test2 = test2;
            this.test3 = test3;
        }
    }
    public static LombokTest of(String test2, @NunNull String test3){
        return new LombokTest(test2, test3);
    }
}
```
* private 생성자가 생성되고, private 생성자를 감싸는 static factory 메소드가 생성된 것을 볼 수 있다.
* 생성자 관련 애노테이션을 사용할 때 주의사항
    1. static 필드들은 스킵됩니다.
    2. 파라미터의 순서는 클래스에 있는 필드 순서에 맞춰서 생성하기 때문에 매우 주의해야 한다.
        * 만약 클래스 필드의 순서를 바꾸었을 때 롬복이 자동적으로 생성자 파라미터 순서를 바꾸어 주기 때문에 주의하지 않으면 버그가 발생할 수 있다.
    3. AccessLevel을 꼭 설정해주어야 합니다.
        * 세 애노테이션 모두 접근 제한자를 AccessLevel로 설정할 수 있습니다.
    4. 기본값은 public이지만 필요로 따라서 접근제한자를 설정해주어야 합니다.
 
 
 ## @TOSTRING

* 이 annotation은  toString 메서드의 구현을 생성한다.
* 기본적으로 모든 non-static 필드는 이름-값 쌍의 메서드 출력에 포함됩니다. includeFieldNames 원하는 경우 주석 매개변수 를  로 false 로 설정하여 출력에 속성 이름이 포함되지 않도록 할 수 있다.
* 특정 필드는 매개변수에 필드 이름을 포함하여 생성된 메소드의 출력에서 ​​제외할 수 있습니다  exclude 또는  of 매개변수를 사용하여 출력에서 ​​원하는 필드만 나열할 수 있습니다.
* 매개변수를  로 설정하여 수퍼클래스 메소드 의 출력을  toString 포함할 수도 있습니다 (callSuper=true)
 
 
 
* Lombok annotated code:
```java
@ToString(callSuper=true,exclude="someExcludedField")
public class Foo extends Bar {
    private boolean someBoolean = true;
    private String someStringField;
    private float someExcludedField;
}
```

* Equivalent Java source code:
```java
public class Foo extends Bar {
    private boolean someBoolean = true;
    private String someStringField;
    private float someExcludedField;
 
    @java.lang.Override
    public java.lang.String toString() {
        return "Foo(super=" + super.toString() +
            ", someBoolean=" + someBoolean +
            ", someStringField=" + someStringField + ")";
    }
}
```
 
 
 
## @EQUALSANDHASHCODE
 
* equals와 hashcode를 자동으로 생성해주는 어노테이션 
* Java에서 equals와 hascode는 다음과 같은 의미입니다.
    * equals : 두 객체의 내용이 같은지, 동등성(equality)를 비교하는 연산자입니다.
    * hashcode : 두 객체가 같은 객체인지, 동일성(identity)를 비교하는 연산자입니다.
* callSuper = true로 설정하면 부모 클래스 필드 값들도 동일한지 체크하며, callSuper = false로 설정(기본값)하면 자신 클래스의 필드 값들만 고려합니다.

* 이 class레벨 annotation 인해 Lombok은  equals 및  hashCode 메서드가 둘 다 생성됩니다. 
    * 둘은 본질적으로  hashCode 계약에 의해 함께 연결되어 있기 때문.

* 기본적으로 정적 또는 일시적이 아닌 클래스의 모든 필드는 두 방법 모두에서 고려됩니다.
    * 생성된 논리 @ToString에  exclude 필드가 포함되지 않도록 매개변수가 제공됩니다. 
    * 매개변수를 사용하여  of 고려해야 하는 필드만 나열할 수도 있습니다.

* 또한  @ToString,  callSuper 이 주석에 대한 매개변수가 있습니다. true로 설정하면   현재 클래스의 필드를 고려하기 전에 수퍼클래스에서 equals 호출하여 동등성을 확인합니다  . 
    * equals메서드 의   경우 해시 계산에 hashCode 슈퍼클래스의 결과가 통합  됩니다.hashCode
* Lombok annotated code:
 
```java
@EqualsAndHashCode(callSuper=true,exclude={"address","city","state","zip"})
public class Person extends SentientBeing {
    enum Gender { Male, Female }
 
    @NonNull private String name;
    @NonNull private Gender gender;
 
    private String ssn;
    private String address;
    private String city;
    private String state;
    private String zip;
}
 
```

* Equivalent Java source code:

```java
public class Person extends SentientBeing {
 
    enum Gender {
        /*public static final*/ Male /* = new Gender() */,
        /*public static final*/ Female /* = new Gender() */;
    }
    @NonNull
    private String name;
    @NonNull
    private Gender gender;
    private String ssn;
    private String address;
    private String city;
    private String state;
    private String zip;
 
    @java.lang.Override
    public boolean equals(final java.lang.Object o) {
        if (o == this) return true;
        if (o == null) return false;
        if (o.getClass() != this.getClass()) return false;
        if (!super.equals(o)) return false;
        final Person other = (Person)o;
        if (this.name == null ? other.name != null : !this.name.equals(other.name)) return false;
        if (this.gender == null ? other.gender != null : !this.gender.equals(other.gender)) return false;
        if (this.ssn == null ? other.ssn != null : !this.ssn.equals(other.ssn)) return false;
        return true;
    }
 
    @java.lang.Override
    public int hashCode() {
        final int PRIME = 31;
        int result = 1;
        result = result * PRIME + super.hashCode();
        result = result * PRIME + (this.name == null ? 0 : this.name.hashCode());
        result = result * PRIME + (this.gender == null ? 0 : this.gender.hashCode());
        result = result * PRIME + (this.ssn == null ? 0 : this.ssn.hashCode());
        return result;
    }
}
```

## @DATA
The @Data annotation is likely the most frequently used annotation in the Project Lombok toolset. It combines the functionality of @ToString, @EqualsAndHashCode, @Getter, and @Setter.

Essentially, using @Data on a class is the same as annotating the class with a default @ToString and @EqualsAndHashCode, as well as annotating each field with both @Getter and @Setter. Annotating a class with @Data also triggers Lombok's constructor generation. This adds a public constructor that takes any @NonNull or final fields as parameters. This provides everything needed for a Plain Old Java Object (POJO).

While @Data is extremely useful, it does not provide the same granularity of control as the other Lombok annotations. In order to override the default method generation behaviors, annotate the class, field, or method with one of the other Lombok annotations and specify the necessary parameter values to achieve the desired effect.

@Data does provide a single parameter option that can be used to generate a static factory method. Setting the value of the staticConstructor parameter to the desired method name will cause Lombok to make the generated constructor private and expose a a static factory method of the given name.

@Data 주석은 아마도 Project Lombok 도구 세트에서 가장 자주 사용되는 주석일 것입니다 . @ToString,  @EqualsAndHashCode,  @Getter및  의 기능을 결합합니다  @Setter.

기본적으로 클래스에서 를 사용 @Data하는 것은 클래스에 기본 @ToString및 로 주석을 추가하는 것과 동일하며 @EqualsAndHashCode각 필드에 @Getter및 를 둘 다 사용하여 주석을 다는 것과 같습니다 @Setter. 클래스에 주석을  추가하면 @Data Lombok의 생성자 생성도 트리거됩니다. 이것은 임의의  @NonNull 또는  final 필드를 매개변수로 사용하는 공용 생성자를 추가합니다. 이것은 POJO(Plain Old Java Object)에 필요한 모든 것을 제공합니다.

매우 유용 하지만  @Data 다른 Lombok 주석과 같은 세부적인 제어를 제공하지 않습니다. 기본 메서드 생성 동작을 재정의하려면 클래스, 필드 또는 메서드에 다른 Lombok 주석 중 하나로 주석을 달고 필요한 매개변수 값을 지정하여 원하는 효과를 얻으십시오.

@Data 정적 팩토리 메소드를 생성하는 데 사용할 수 있는 단일 매개변수 옵션을 제공합니다. 매개변수 값을  staticConstructor 원하는 메소드 이름으로 설정하면 Lombok이 생성된 생성자를 비공개로 만들고 지정된 이름의 정적 팩토리 메소드를 노출합니다.


* Lombok annotated code:
```java
@Data(staticConstructor="of")
public class Company {
    private final Person founder;
    private String name;
    private List<Person> employees;
}
```

* Equivalent Java source code:
```java
public class Company {
    private final Person founder;
    private String name;
    private List<Person> employees;
 
    private Company(final Person founder) {
        this.founder = founder;
    }
 
    public static Company of(final Person founder) {
        return new Company(founder);
    }
 
    public Person getFounder() {
        return founder;
    }
 
    public String getName() {
        return name;
    }
 
    public void setName(final String name) {
        this.name = name;
    }
 
    public List<Person> getEmployees() {
        return employees;
    }
 
    public void setEmployees(final List<Person> employees) {
        this.employees = employees;
    }
 
    @java.lang.Override
    public boolean equals(final java.lang.Object o) {
        if (o == this) return true;
        if (o == null) return false;
        if (o.getClass() != this.getClass()) return false;
        final Company other = (Company)o;
        if (this.founder == null ? other.founder != null : !this.founder.equals(other.founder)) return false;
        if (this.name == null ? other.name != null : !this.name.equals(other.name)) return false;
        if (this.employees == null ? other.employees != null : !this.employees.equals(other.employees)) return false;
        return true;
    }
 
    @java.lang.Override
    public int hashCode() {
        final int PRIME = 31;
        int result = 1;
        result = result * PRIME + (this.founder == null ? 0 : this.founder.hashCode());
        result = result * PRIME + (this.name == null ? 0 : this.name.hashCode());
        result = result * PRIME + (this.employees == null ? 0 : this.employees.hashCode());
        return result;
    }
 
    @java.lang.Override
    public java.lang.String toString() {
        return "Company(founder=" + founder + ", name=" + name + ", employees=" + employees + ")";
    }
}
```


## @CLEANUP
* try-with-resource 구문과 비슷한 효과 로 구문이 종료될 때 AutoCloseable 인터페이스의 close()가 호출되는 try-with-resource와 달리 Scope가 종료될 때 close()가 호출된다
* @Cleanup 주석을 사용할 때 고려해야 할 주의 사항도 있습니다. 정리 메서드에서 예외가 throw되는 경우 메서드 본문에서 throw된 모든 예외를 선점합니다. 이로 인해 문제의 실제 원인이 묻힐 수 있으며 Project Lombok의 리소스 관리 사용을 선택할 때 고려해야 합니다. 또한 Java 7에서 자동 리소스 관리가 가능해지면서 이 특정 주석은 상대적으로 수명이 짧을 수 있습니다.



Lombok annotated code:
 
```java
public void testCleanUp() {
    try {
        @Cleanup ByteArrayOutputStream baos = new ByteArrayOutputStream();
        baos.write(new byte[] {'Y','e','s'});
        System.out.println(baos.toString());
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

Equivalent Java source code:
```java
public void testCleanUp() {
    try {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        try {
            baos.write(new byte[]{'Y', 'e', 's'});
            System.out.println(baos.toString());
        } finally {
            baos.close();
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```


## @SYNCHRONIZED
Using the synchronized keyword on a method can result in unfortunate effects, as any developer who has worked on multi-threaded software can attest.

The synchronized keyword will lock on the current object (this) in the case of an instance method or on the class object for a static method. This means that there is the potential for code outside of the control of the developer to lock on the same object, resulting in a deadlock.

It is generally advisable to instead lock explicitly on a separate object that is dedicated solely to that purpose and not exposed in such a way as to allow unsolicited locking. Project Lombok provides the @Synchronized annotation for that very purpose.

Annotating an instance method with @Synchronized will prompt Lombok to generate a private locking field named $lock on which the method will lock prior to executing. Similarly, annotating a static method in the same way will generate a private static object named $LOCK for the static method to use in an identical fashion. A different locking object can be specified by providing a field name to the annotation's value parameter. When a field name is provided, the developer must define the property because Lombok will not generate it.


메소드에 키워드를 사용하면  synchronized 다중 스레드 소프트웨어에 대해 작업한 개발자가 증명할 수 있듯이 불행한 결과가 발생할 수 있습니다.

synchronized 키워드는 this인스턴스 메서드의 경우 현재 개체( )를  잠그고 class 정적 메서드의 경우 개체를 잠급니다. 이는 개발자가 제어할 수 없는 코드가 동일한 개체를 잠글 가능성이 있어 교착 상태가 발생할 수 있음을 의미합니다.

일반적으로 해당 목적 전용이며 원치 않는 잠금을 허용하는 방식으로 노출되지 않는 별도의 개체에 대해 명시적으로 잠그는 것이 좋습니다. Project Lombok은  @Synchronized 바로 이러한 목적을 위해 주석을 제공합니다.

인스턴스 메서드에 주석을  추가하면 Lombok  에서 실행 전에 메서드가 잠길 @Synchronized 이름이 지정된 개인 잠금 필드를 생성하라는 메시지가 표시됩니다  . $lock유사하게, 동일한 방식으로 정적 메소드에 주석을 추가하면 동일한 방식  $LOCK 으로 사용할 정적 메소드에 대해 명명된 전용 정적 객체가 생성됩니다. 주석의  value 매개변수에 필드 이름을 제공하여 다른 잠금 객체를 지정할 수 있습니다. 필드 이름이 제공되면 Lombok에서 속성을 생성하지 않으므로 개발자가 속성을 정의해야 합니다.



Lombok annotated code:

```java
private DateFormat format = new SimpleDateFormat("MM-dd-YYYY");
 
@Synchronized
public String synchronizedFormat(Date date) {
    return format.format(date);
}
```

Equivalent Java source code:


```java
private final java.lang.Object $lock = new java.lang.Object[0];
private DateFormat format = new SimpleDateFormat("MM-dd-YYYY");
 
public String synchronizedFormat(Date date) {
    synchronized ($lock) {
        return format.format(date);
    }
}
```


## @SNEAKYTHROWS
@SneakyThrows is probably the Project Lombok annotation with the most detractors, since it is a direct assault on checked exceptions. There is a lot of disagreement with regards to the use of checked exceptions, with a large number of developers holding that they are a failed experiment. These developers will love @SneakyThrows. Those developers on the other side of the checked/unchecked exception fence will most likely view this as hiding potential problems.

Throwing IllegalAccessException would normally generate an "Unhandled exception" error if IllegalAccessException, or some parent class, is not listed in a throws clause:


@SneakyThrows 확인된 예외에 대한 직접적인 공격이기 때문에 아마도 가장 비방하는 사람들이 있는 Project Lombok 주석일 것입니다. 확인된 예외의 사용과 관련하여 많은 불일치가 있으며 많은 개발자가 실패한 실험이라고 주장합니다. 이 개발자들은 좋아할 것  @SneakyThrows입니다. 체크/비체크 예외 펜스의 반대편에 있는 개발자들은 이것을 잠재적인 문제를 숨기는 것으로 볼 가능성이 큽니다.

던지기  는 일반적으로 , 또는 일부 부모 클래스가 throws 절에 나열 IllegalAccessException 되지 않은 경우 "처리되지 않은 예외" 오류를 생성합니다  .IllegalAccessException

![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-03-10-java-annotation-01-01.PNG)


When annotated with @SneakyThrows, the error goes away.
로 주석을 달면  @SneakyThrows오류가 사라집니다.

Sneaky Throws
By default, @SneakyThrows will allow any checked exception to be thrown without declaring in the throws clause. This can be limited to a particular set of exceptions by providing an array of throwable classes ( Class) to the value parameter of the annotation.


기본적으로   절 @SneakyThrows 에서 선언하지 않고 검사된 예외가 throw되도록 허용합니다  .  이것은 주석 의 매개변수에 throw 가능한 클래스( )  throws의 배열을 제공하여 특정 예외 집합으로 제한될 수 있습니다  .Classvalue


Lombok annotated code:

 
```java
@SneakyThrows
public void testSneakyThrows() {
    throw new IllegalAccessException();
}
```

```java
public void testSneakyThrows() {
    try {
        throw new IllegalAccessException();
    } catch (java.lang.Throwable $ex) {
        throw lombok.Lombok.sneakyThrow($ex);
    }
```

A look at the above code and the signature of Lombok.sneakyThrow(Throwable) would lead most to believe that the exception is being wrapped in a RuntimeException and re-thrown; however this is not the case. The sneakyThrow method will never return normally and will instead throw the provided throwable completely unaltered.

위의 코드와 의 서명을  보면 예외가 로 래핑되어  다시 발생 Lombok.sneakyThrow(Throwable) 한다고 대부분 믿게 됩니다  . RuntimeException그러나 이것은 사실이 아닙니다. 메서드 는  sneakyThrow 정상적으로 반환되지 않으며 대신 제공된 throwable을 완전히 변경되지 않은 상태로 throw합니다.




## 참고
* https://projectlombok.org/
    * https://objectcomputing.com/resources/publications/sett/january-2010-reducing-boilerplate-code-with-project-lombok
* https://dingue.tistory.com/14



