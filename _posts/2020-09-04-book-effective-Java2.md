---
title:  "Effective Java 2"
excerpt: "Effective Java 2"

categories:
  - etc
tags:
  - [etc, Programming]

toc: true
toc_sticky: true
 
date: 2020-09-04
last_modified_at: 2020-09-04


published: false
---

## 1장 서론
## 2장 객체의 생성과 삭제
* 규칙 1 생성자 대신 정적 팩터리 메소드를 사용할 수 없는지 생각해 보라
    * 정적 팩터리 메소드
        * 객체를 생성자가 아닌, 정적 메서드를 이용해서 객체를 생성, 반환하는 것을 말한다.

        ```java
        //예제
        class Monitor {
            int size;
            public Monitor(int size) {
                this.size = size;
            }
            // 정적 팩토리 메소드
            public static Monitor new24inch() {
                return new Character(24);
            }
            // 정적 팩토리 메소드
            public static Monitor new27inch() {
                return new Character(27);
            }
        }
        .
        .
        //생성자로 할 경우
        Monitor monitor1 = new Monitor(24);
        Monitor monitor2 = new Monitor(27);

        //정적 팩토리 메소드로 할 경우
        Monitor monitor1 = Monitor.new24inch();
        Monitor monitor2 = Monitor.new27inch();
        ```
    * 장점
        * 생성자와는 다르게 자기 자신만의 이름을 가질 수 있다.
        * 생성자와는 다르게 매번 호출 할 때마다 객체를 생성하지 않아도 된다.
            * 미리 만들어 놓고 싱글톤처럼 사용가능하다
        * 자기 자신만을 인스턴스만 반환할 수 있는 생성자와는 다르게 자신이 어떠한 서브타입 객체도 반환할 수 있다.
            ```java
            //예제
            class UHD extends Monitor { }
            class QHD extends Monitor { }

            class MonitorUtil {

                //정적팩토리메소드 - 해상도(resolution) 로 생성할수있따.
                public static Monitor createMonitor(String resolution) throws Exception {
                    // UHD인가? QHD 인가?
                    if(isUHD(resolution)) {
                        return new UHD인가(1000); // Monitor 의 서브타입 객체
                    } else if(isQHD(resolution)) {
                        return new QHD(500); // Monitor 의 서브타입 객체
                    }
                }
            }


            ```

        * 형인자 자료형 객체를 만들 때 편리하다
    * 단점
        * public나 protected로 선언된 생성자가 없으므로 하위 클래스를 만들 수 없다
            * 보통 이것은 생성자 대신에 쓰겠다라는 의미이기 때문에 아래처럼 생성자를 막아준다.
            ```java
            class A {
                private A() { }
            }
            이렇게 하면 아래처럼 사용못한다.
            class B extends A {
                ...
            }  
            ```
    * 정적 팩터리 메서드가 다른 정적 메서드와 확연히 구분되지 않는다
        * 정적 팩터리 메서드를 쓰면 사용법을  파악하기 힘들다.
* 보통 정적 팩터리 메서드의 이름은 다음과 같이 쓰인다.
    * valueOf : 인자로 주어진 값을 갖는 객체를 반환. 형변환 (type-conversion) 메소드
    * of : valueOf를 간단하게 쓴 것
    * getInstance : 인자에 기술된 객체를 반환 하지만, 인자와 같은 값을 갖지 않을 수도 있다.
    * newInstance : getInstance와 같지만 호출할 때 마다 다를 객체를 반환한다.
    * getType : getInstance와 같지만, 반환될 객체의 클래스와 다른 클래스의 팩토리 메소드가 있을 때 사용한다.
    * newType : newInstance와 같지만, 반환될 객체의 클래스가 다른 클래스의 다른 팩토리 메소드에 있을 때 사용한다.

----

* 규칙 2 생성자 인자가 많을 때는 Builder 패턴 적용을 고려하라.
    * 인자가 많은 클래스의 경우 전진적인 생성자를 이용해서 생성할수있지만.
        ```java
        class A{
          A(){}
          A(int a){};
          A(int a, int b){};
          A(int a, int b, int c){}
        ...
        }
        ```
    * 아래처럼 builder 패턴을 이용하자
        ```
        class A {
            private final String a;
            private final String b;
            private final String c;

            public static class Builder {
                private String a = "";
                private String b = "";
                private String c = "";

                public Builder(String a, String b) {this.a = a;this.b = b;}
                public Builder setC(String c) { this.c = c; return this;}
                public A build() {
                    return new A(this);
                }
            }

            public A(Builder builder) {
                this.a = builder.a;
                this.b = builder.b;
                this.sex = builder.sex;
            }
        }
        .
        .
        .
        A a = new A.Builder("a", "b").setC("C").build();
        ```
    * 하지만 객체를 생성하려면 우선 빌더 객체를 생성해야 하는 단점은 있다.


* 규칙 3 private 생성자나 enum 자료형은 싱글턴 패턴을 따르도록 설계하라
    ```java
    // public final 필드를 이용한 싱글턴
    public class Elvis {
      public static final Elvis INSTANCE = new Elvis();
      private Elvis() { ... }
      public void leaveTheBuilding( ) { ...}
    }
    ```
    ```java
    // 정적 펙터리를 이용한 싱글턴
    public class Elvis {
      private static final Elvis INSTANCE = new Elvis();
      private Elvis() { ... }
      public static Elvis getInstance() { return INSTANCE; }

      public void leaveTheBuilding( ) { . . . }
    }
    ```

    ```java
    // Enum 싱글턴 - 이렇게 하는 쪽이 더 낫다
    public enum Elvis {
      INSTANCE;

      public void leaveTheBuilding() { . . . }
    }
    ```
* 규칙 4 객체 생성을 막을 때는 private 생성자를 사용하라
    * 생성자를 생략하면 컴파일러가 자동으로 인자없는 public 기본 생성자를 만든다. 유틸성이나, 정적 메소드나 필드만 모은 클래스를 만들어야할땐 abstract로 해봤자 하위 클래스를 정의하는 순간 객체 생성이 가능해지기때문에 private 생성자를 클래스에 넣어서 객체 생성을 방지하는것이 좋다. 
    ```java
    // 객쳬룰 만둘 수 없는 유틸리티 클래스
    public class UtilityClass {
      // 기본 생성자가 자동 생성되지 못하도록 하여 객체 생성 방지
      private UtilityClass() {
        throw new AssertionError();
      }
      ...
    }
    ```
    * 명시적으로 정의된 생성자가 private이르모 클래스 외부에서 사용X, 클래스 안에서 실수로 생성자를 호출하면 바로 알수있도록 AssertionError 를 넣음. 

* 규칙 5 불필요한 객체는 만들지 말라
    * 생성자 대신 정적 팩터리 메서드를 이용하면 불필요한 객체 생성을 피할수 있을때가 많다. 
    * 아래는 정적 초기화 블록(static initalizer) 를 통해서 개선할수 있는 예제이다.
        ```java
        public class Person{
            private final Date birhDate;
            //이렇게 하면 안된다. 
            public  boolean isBabyBoomer(){
                //생성비용이 높은 객체를 쓸데없이 생성한다. 
                Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
                gmtCal.set(1946, ...);
                Date boomStart = gmtCal.getTime();
                gmtCal.set(1965, ...);
                Date boomEnd = gmtCal.getTime();
                return birthDate.compareTo(boomStart) >= 0 && birthDate.compare(boomEnd) < 0;
            }
        }
        ```
        * 이건 isBabyBoomer 메서드 호출될때마다 Calendar 객체하나, TimeZone 객체하나, Date객체2개를 쓸데없이 만든다. 
        ```java
        public class Person{
            private final Date birthDate;
            private static final Date BOOM_START;    
            private static final Date BOOM_END;

            //
            static{
                Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
                gmtCal.set(1946, ...);
                BOOM_START = gmtCal.getTime();
                gmtCal.set(1965, ...);
                BOOM_END = gmtCal.getTime();
            }

            public boolean isBabyBoomer(){
                return birthDate.compareTo(BOOM_START) >= 0 && birthDate.compare(BOOM_END) < 0;
            }
        }
        ```
        * 이렇게 하면 클래스 초기화 할때 한번만 만드니 효율적.
    * 직접 관리하는 객체 풀(Object pool)을 만들어 객체 생성을 피하는 기법은 객체 생성 비용이 극단적으로 높지 않다면 사용하지 않는 것이 좋다.(db connection pool같은것만 이렇게 하고 웬만하면 코드가 복잡해지니 사용하지 말라.)
* 규칙 6 유효기간이 지난 객체 참조는 폐기하라
    * 쓸 일이 없는 객체참조는 무조건 null로 하자.

* 규칙 7 종료자 사용을 피하라
    * 종료자(finalizer)는 C++ 소멸자와는 다르다.
        * finalizer는 즉시 실행되리라는 보장이 전혀 없다.(긴급한 작업은 여기서 하면 안된다.)
        * finalizer 실행시점은 gc 알고리즘에 좌우된다. 
        * java 명세에 즉시 실행되야한다고 없고, 반드시 실행되야한다는 문구도 없다.
    * 프로그램 성능이 심각하게 떨어진다. 
    * 명시적 종료 메서드는 try-finally 문과 함께 쓰자

    ```java
    Foo foo = new foo
    try{
        //foo로 해야하는 작업
        ...
    } finally{
        foo.terminate();//명시적 종료 메소드 호출
    }
    ```
    * 자원 반환에 대한 최종적 안전장치를 구현하거나, 그다지 중요하지 않는 네이티브 자원을 종료시키려는 것이 아니라면 종료자는 사용하지 말자. 

## 3장 모든 객체의 공통 메서드
* Object는 객체 생성이 가능한 클래스이지만 기본적으로 계승해서 사용사도록 설계된 클래스이다. Object에 정의된 비-final 메서드(equals, hashCode, toString, clone, finalize)에는 override사도록 설계된 메서들이기때문에 명시적인 일반규약이 있다. 그 규약을 알아보자.

* 규칙 8 equals를 재정의할 때는 일반 규약을 따르라
    * 각각의 객체가 고유하다
    * 클래스에 논리적 동일성(logical equality), 검사 방법이 있건 없건 상관없다.
    * 상위 클래스에서 재정의한 equals가 하위 클래스에서 사용히기에도 적당하다.
    * 클래스가 private 또는 package-private로 선언되었고, equals 메서드를 호출할 일이 없다.
    * 반사성(Reflexivity): null이 아닌 참조 x가 있을때 x.equals(x)는 true를 반환한다. 
    * 대칭성(Symmetry): null 아닌 참조 x,y가 있을때 x.equals(y)는 y.equals(x)가 true일때만 true를 반환한다.
    * 추이성(transitive): null 아닌 참조, x,y,z가 있을때 x.equals(y)가 true이고 y.equals(x)가 true이면 x.equals(z)도 true이다. 
    * 일관성(Consistency): null아닌 참조 x,y가 있을때 equals를 통해 비교되는 정보에 아무 변화가 없다면 x.equals(y)호출 결과는 호출 횟수에 상관없이 항상 같아야 한다.
    * null아닌 참조 x에 대해서 x.equals(null)은 항상 false이다.
    * equals 메서드를 구현하기 위해 따라야 할 지침!
        * == 연산자를 사용하여 equals의 인지가 자기 자신인지 검사하라.
        * instanceOf 연산지를 사용하여 인자의 자료형이 정확한지 검사하라.
        * equals의 인자를 정확한 자료형으로 변환하라.
        * ‘중요’ 필드 각각이 인자로 주어진 객체의 해당 필드와 일치하는지 검사한다.
        * equals 메서드 구현을 끝냈다면, 대칭성, 추이성, 일관성의 세 속성이 만족되는지 검토하라.
        * equals를 구현할때는 hasCode도 재정의하라
        * equals메서드의 안지형을 Object에서 다른것으로 바꾸지 마라.

* 규칙 9 equals를 재정의할 때는 반드시 hashCode도 재정의하라
    * equals 메서드를 재정의하는 클래스는 반드시 hashCode 메서드도 재정의 해야 한다
        * HashMap, HashSet, HashTable 같은 해시(hash) 기반 컬렉션과 함께 시용하면 오동작
    * hashCode를 재정의하지 않으면 위반되는 핵심 규약은 두번째다. 같은 객체는 같은 혜시 코드 값을 가져야 한다는 규약이 위반되는 것이다.

* 규칙 10 toString은 항상 재정의하라
    * 가능하다면 toString 메서드는 객체 내에 중요 정보를 전부 담아 반환해야 한다.
    * toString이 반환하는 문자열의 형식을 명시하건 그렇지 않건 간에, 어떤 의도인지는 문서에 분명하게 남겨야 한다.
    * toString이 반환하는 문자열에 포함되는 정보들은 전부 프로그래밍을 통해서 가져올 수 있도록(programmatic access) 하라.

* 규칙 11 clone을 재정의할 때는 신중하라
    * 비-final 클래스에 clone을 재정의할 때는 반드시 super.clone을 호출해 얻은 객체를 반환해야 한다.
    * 실질적으로 cloneable 인터페이스를 구현하는 클래스는 제대로 동작하는 public clone 메서드를 제공해야 한다.
    * 라이브러리가 할 수 있는 일을 클라이언트에게 미루지 말라
    * 라이브러리가 할 수 있는 일을 클라이언트에게 미루지 말라는 것이다.
    * 사실상, clone 메서드는 또 다른 형태의 생성자다. 원래 객체를 손상시키는 일이 없도록 해야 하고, 복사본의 불변식(invariant)도 제대로 만족시켜야 한다.
    * clone의 아키텍처는 변경 가능한 객체를 참조하는 final 필드의 일반적 용법과 호환되지 않는다.
    * 객체를 복사할 대안을 제공하거나, 아예 복제 기능을 제공하지 않는 것이 낫다. 
    * 객체 복제를 지원하는 좋은 방법은, 복사 생성자나 복사 팩터리를 제공는 것이다. 
* 규칙 12 Comparable 구현을 고려하라
    * compareTo를 구현할 때는 모든 x와 y에 대해 sgn(x.compareTo(y)) == -sgn(y,compareTo(x))가 만족되도록 해야 한다. (y.compareTo(x)가 예외를 발생시킨다면 x.compareTo(y)도 그래야 하고, 그 역도 성립해야 한다.)
    * compareTo를 구현할 때는 추이성(transitivity)이 만족되도록 해야 한다. 즉, (x.compareTo(y)) 0 && y.compareTo(z)) 0)이면 x.compareTo(z)) 0이어야 한다.
    * 마지막으로, x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))의 관계가 모든 z에 대해 성립하도록 해야 한다.
    * 강력히 추천하지만 절대적으로 요구되는 것은 아닌 조건 하나는 (x.compareTo(y) == 0) == (x.equals(y))이다.

## 4장 클래스와 인터페이스
* 규칙 13 클래스와 멤버의 접근 권한은 최소화하라
    * 각 클래스와 멤버는 기능한 한 접근 불가능하도록 만들어라.
    * private - 이렇게 선언된 멤버는 선언된 최상위 레벨 클래스 내부에서만 접근 가능하다.
    * package-private - 이렇게 선언된 멤버는 같은 패키지 내의 아무 클래스나 사용할 수 있다. 기본 접근 권한(default access)으로 알려져 있는데, 멤버를 선언할 때 아무런 접근 권한 수정자(access modifier)도 붙이지 않으면, 이 권한이 주어지기 때문.
    * protected - 이렇게 선언된 멤버는 선언된 클래스 및 그 하위 클래스만 사용할 수 있다(몇 가지 제약 사항에 대해서는 [JLS, 6.6.2] 참고). 선언된 클래스와 같은 패키지에 있는 클래스에서도 시용이 기능하다. (사용을 자제하자)
    * public - 이렇게 선언된 멤버는 어디서도 사용이 가능하다.
    * 객체 필드(instance field)는 절대로 public으로 선언하면 안 된다.
    * 변경 가능 public 필드를 가진 클래스는 다중 스레드에 안전하지않다.
    * public static final 필드를 제외한 어떤 필드도 public 필드로 선언하지 마라.
    * public static final 필드가 참조하는 객체는 변경 불가능 객체로 만들라.

* 규칙 14 public 클래스 안에는 public 필드를 두지 말고 접근자 메서드를 사용하라
    * 선언된 패키지 밖에서도 사용 가능한 클래스에는 접근자 메서드를 제공하라.
        ```java
        //이딴식으로 사용하지 말라
        class Point {
          public double x;
          public double y;
        }
        
        // 접근자 메서드와 수정자룰 이용해 이렇게 사용해라
        class Point {
          private double x;
          private double y;

          public Point(double x, double y) {
            this.x = x;
            this.y = y;
          }

          public double getX() { return x; }
          public double getY() { return y; }
          public void setX(double x) { this.x = x; }
          public void setY(double y) { this.y = y; }
        }
        ```
* 규칙 15 변경 가능성을 최소화하라
    * 변경 불가능 클래스 규칙 5가지를 지켜라
        * 객체 상태를 변경하는 메서드(수정자 메서드 등)를 제공하지 않는다.
        * 계승할 수 없도록 한다.(보통 클래스를 final로 선언하면 된다.)
        * 모든 필드를 final로 선언한다.
        * 모든 필드를 private로 선언한다.
        * 변경 기능 컴포넌트에 대한 독점적 접근권을 보장한다.
        ```java
        public final class Complex{
            private final double re;
            private final double im;
            public Complex(double re, double, im){
                ...
            }
            public double readPart(){return re;}
            public double imaginaryPart(){return im;}
            ...
        }
        ```
    * 변경 불가능 객체는 단순하다.
    * 변경 불가능 객체는 스레드에 안전, 어떤 동기화도 필요 없음
    * 변경 불가능한 객체는 자유롭게 공유할 수 있다.
    * 변경 불기능한 객체는 그 내부도 공유할 수 있다.
    * 변경 불기능 객체의 유일한 단점은 값마다 별도의 객체를 만들어야 한다는 점이다.
    * 변경 불가능한 클래스로 만들수 없다면, 변경 가능성을 최대한 제한해라
    * 변경 가능한 클래스로 만들 타당한 이유가 없으면, 반드시 변경 불가능 클래스로 만들어라.
    * 특별한 이유가 없다면 모든 필드는 final로 선언해라

* 규칙 16 계승하는 대신 구성하라
    * 메서드 호출과 달리, 계승은 캡슐화(encapsulation) 원칙을 위반한다.
        * 하위 클래스는 상위 클래스의 변화에 발맞춰 진화해야한다. 
    * 계승대신 wrapper 클래스를 만드는 것도 좋은 방법중 하나이다.

* 규칙 17 계승을 위한 설계와 문서를 갖추거나, 그럴 수 없다면 계승을 금지하라
    * 재정의 가능 메서드를 내부적으로 어떻게 사용하는지(self-use) 반드시 문서에 남기라는 것이다.
    * 클래스 내부 동작에 개입할 수 있는 훅(hooks)을 신중하게 고른 protected 메서드 형태로 제공해야 한다.
    * 계승을 위해 설계한 클래스를 테스트할 유일한 방법은 하위 클래스를 직접 만들어 보는 것이다.
    * 계승에 맞도록 설계하고 문서화하지 않는 클래스에 대한 하위 클래스는 만들지 않는 것이다. 

* 규칙 18 추상 클래스 대신 인터페이스를 사용하라
    * 이미 있는 클래스를 개조해서 새로운 인터페이스를 구현하도록 하는 것은 간단하다.
    * 인터페이스는 믹스인(mixin)을 정의하는 데 이상적이다.
    * 인터페이스는 비 계층적인(nonhierarchical) 자료형 프레임워크(type framework)를 만들 수 있도록 한다.
    * 인터페이스를 사용하면 포장 클래스 숙어(wrapper class idiom)을 통해 안전하면서도 강력한 기능 개선이 기능하다.
    * 추상 골격 구현(abstract skeletal implementation) 클래스를 중요 인터페이스마다 두면, 인터페이스의 점과 추상 클래스의 장
    * 인터페이스보다는 추상 클래스가 발전시키기 쉽다
    * 인터페이스가 공개되고 널리 구현된 다음에는, 인터페이스 수정이 거의 불기능

* 규칙 19 인터페이스는 자료형을 정의할 때만 사용하라
    * 상수 인터페이스 패턴은 인터페이스를 잘못 사용한 것이다.

* 규칙 20 태그 달린 클래스 대신 클래스 계층을 활용하라
    * 하나의 클래스에 여러 구현체를 구현할수 있는 클래스를 만들지 말고 계층구조를 이용해라

* 규칙 21 전략을 표현하고 싶을 때는 함수 객체를 사용하라
    * * 객체 참조를 통해 C의 함수포인터와 비슷한 효과를 낼 수 있다.

* 규칙 22 멤버 클래스는 가능하면 static으로 선언하라
    * 중첩 클래스(nested class)
        * 다른 클래스 안에 정의된 클래스
        * 해당 클래스가 속한 클래스 안에서만 사용
        * 정적 멤버 클래스 (static member class)
            * ..
        * 비-정적 멤버 클래스(nonstatic member class)
            * ..
        * 익명 클래스(anonymous class)
            * ..
        * 지역 클래스(local class)
            * ..

## 5장 제네릭
* 규칙 23 새 코드에는 무인자 제네릭 자료형을 사용하지 마라
    * 선언부에 형인자(type parameter)가 포함된 클래스나 인터페이스는 제네릭(generic) 클래스나 인터페이스라고 한다.
        * 가령, List인터페이스에 리스트에 보관될 원소의 자료형을 나타내는 형인자 E가 도입됨.
    * 기존에 이렇게 하면 넣을땐 문제가 없지만, 뺄때 ClassCastException이 발생
    ```java
    private final Collection stampls = ...;
    //실수로 Coin 객체를 넣음
    stamps.add(new Coin(...));
    (Stamp)stamps.iterator.next();//exception
    ```
    * 형인자 컬렉션 자료형-형 안정성 확보
    ```java
    private final Collection<Stamp> stamps = ...;
    ```
    * 무인자 자료형을 쓰면 형 안전성이 사라지고, 제네릭의 장점 중 하나인 표현력(expressiveness) 측면에서 손해를 보게 된다.
        * ex. List < E > 의 무인자 자료형은 List 이다.
    * List와 같은 무인자 자료형을 사용하면 형 안전성을 잃게 되지만, List<Object> 와 같은 형인자 자료형을 쓰면 그렇지 않다.
        * List 와 List < Object > 차이는 List는 형 검사 절차를 완전히 생략한것이고, 후자는 아무 객체나 넣을 수 있다는 것을 컴파일러에게 알리것이다. 
    * 비한정적 와일드카드 자료형도 있다. 
        * 제네릭자료형을 쓰고싶으나 실제 형 인자가 무엇인지 모르고 쓰거나, 신경쓰지 않고싶을때 형 인자로 ? 를 쓰면 된다. 
            * ex. Set < E > 에 대한 비한정적 와이륻카드 자료형은 Set < ? >  이다. 
            * Collection<?>에는 null 이의의 어떤 원소도 넣을 수 없다.
            * Collection<?>에는 어떤 자료형의 객체를 꺼낼 수 있는지도 알 수 없다.

* 규칙 24 무점검 경고(unchecked warning)를 제거하라
    * 모든 무점검 경고는, 가능하다면 없애야 한다.
    * 제거할 수 없는 경고 메시지는 형 안전성이 확실할 때만 @SuppressWarnings(“unchecked”) 어노테이션(annotation)을 사용해
    * 하지만 @SuppressWarnings 어노테이션은 가능한 한 작은 범위에 적용하라.
    * @SuppressWarnings(“unchecked”) 어노테이션을 사용할 때마다, 왜 형 안전성을 위반하지 않는지 밝히는 주석을 반드시 붙이라


* 규칙 25 배열 대신 리스트를 써라
    * 배열은 공변 자료형(covariant)
        * Sub가 Super의 하위 자료형(subtype)이라면 Sub[]도 Super[]의 하위 자료형이라는 것이다
        * 제네릭은 불변 자료형(invariant)
            * Type1과 Type2가 있을 때, List<Type1>은 List<Type2>의 상위 지료형이나 하위 자료형이 될 수 없다.
    * 배열은 실체화(reification) 되는 자료형
        * 배열의 각 원소는 실행시간에 결정됨.
        * 제네릭은 삭제(erasure) 과정을 통해 구현된다.
    * new List<E>[], new List<String>[], new E[]는 전부 컴파일되지 않는 코드다. 컴파일하려고 하면 제네릭 배열 생성(generic array creation)이라는 오류가 발생
        * E, List<E>, List<String>와 같은 자료형은 실체화 불기능(non-refiable) 자료형


* 규칙 26 가능하면 제네릭 자료형으로 만들 것
    * 클래스를 제네릭화하는 첫 번째 단계는 선언부에 형인자(type parameter)를 추가, 관습적으로 E라고 붙이자.
    * 두번째 단계는 Object를 자료형으로 사용하는 부분을 전부 형인자 E로 대체 후 컴파일 해보자.
        * E와 같이 실제화 불가능 자료형은 배열을 생성할수 없다. 
            * Object배열을 만들어서 제네릭 배열 자료형으로 형변환 하던지
            * E[]에서 Object[]로 바꾸던지 하자.

* 규칙 27 가능하면 제네릭 메서드로 만들 것
    * 형인자를 선언하는 형인자 목록(type parameter list)은 메서드의 수정자(modifier)와 반환값 자료형 사이에 두자.
    ```java
    //권할수 없는 방법
    public static Set union(Set s1, Set s2){
        Set result = new HashSet(s1);
        result.addAll(s2);
        return result
    }
    //제네릭 메서드
    public static <E> Set<E> union(Set<E> sq, Set<E> s2){
        Set<E> result = new HashSet<E>(s1);
        result.addAll(s2);
        return result
    }
    ```
   
* 규칙 28 한정적 와일드카드를 써서 API 유연성을 높여라
    * 규칙 25에서 처럼 형인자 자료형(parameterized type)은 불변(incariant) 자료형이다. Type1,Type2가 List< Type1>, List< Type2>사이에 어떠한 상위-하위 자료형 관계도 성립할수 없다. 
    ```java
    //와일드 카드 자료형을 사용하지 않는 - 문제가 있다.
    public void pushAll(Iterable<E> src){
        for(E e: src)
            push(e);
    }
    //E 객체 생산자 역할을 하는 인자에 대한 와일드카드 자료형
    public void pushAll(Iterable<? extends E> src){
        for(E e: src)
            push(e);
    }
    ```
    * 모든 자료형은 자기 자신의 하위 자료형이다.
    ```java
    //와일드 카드 자료형을 없이 구현한 - 문제가 있다.
    public void popAll(Collection<E> dst){
        while(!isEmpty())
            dst.add(pop());
    }
    //E 소비자 구실을 하는 인자에 대한 와일드카드 자료형
    public void popAll(Collection<? super E> dst){
        while(!isEmpty())
            dst.add(pop());
    }
    ```
    * 모든 자료형E는 자기 자신의 상위 자료형이다.
    * PECS (Produce - Extends, Consumer - Super)
    * 반환값에는 와일드카드 자료형을 쓰면 안 된다.
    * 클래스 사용자가 와일드카드 자료형에 대해 고민하게 된다면, 그것은 아마도 클래스 API가 잘못 설계된 탓일 것이다.
    * Comparable< T> 대신 항상 Comparable<? super T>를 사용해야 한다.
    * Comparator< T> 대신 항상 Comparator<? super T>를 사용해야 한다.
    * 형인자가 메서드 선언에 단 한군데 나타난다면 해당 인자를 와일드카드로 바꾸라는 것이다.

* 규칙 29 형 안전 다형성 컨테이너를 쓰면 어떨지 따져보라
    * ...

## 6장 열거형(enum)과 어노테이션
* 규칙 30 int 상수 대신 enum을 사용하라
    * 자바에 enum 이란 자료형이 도입되기 이전에 아래처럼 사용해서 열거자료형을 흉내냈었다.
    ```java
    //int를 이용한 euum 패턴
    public static final int APPLE_FUJI = 0;
    public static final int APPLE_PIPPIN = 1;
    //아래처럼 하자
    public enum Apple {FUIJ, PIPPIN}
    ```
    * C의enum과 비슷하지만 좀 다르다.
        * 자바의 enum은 완전한 기능을 갖춘 클래스다. (다른 언어는 결국 int값이다.)
        * 열거 상수 (enumeration constant)별로 하나의 객체를 public static final 필드 형태로 제공하는 것이다.
        * 임의의 메서드나 필드도 추가할수 있다?

    ```java
    //데이터와 연산을 구비한 enum 자료형
    public enum Planet {
        MERCURY(3.302e+23, 2.439e6),
        EARTH(5.975e+24, 6.378e6);
        
        private final double mass;//킬로그램 단위
        private final double radius;//미터단위
        //생성자
        Planet(double mass, double radius){
            this.mass = mass;
        }
        public double mass() {returm mass};
    }
    ```
    * enum 상수에 데이터를 넣으려면 객체 필드(instance field)를 선언하고 생성자를 통해 받은 데이터를 그 필드에 저장하면 된다.
    * 요약하면 enum 자료형은 int 상수에 비해 가독성도 높고, 안전하고, 더 강력하다. 

* 규칙 31 ordinal 대신 객체 필드를 사용하라

    ```java
    // ordinal을 남용한 사례 - 따라하면 곤란
    public enum Ensemble {
        SOLO, DUET, TRIO, QUARTET, QUINTET, SEXTET, SEPTET, OCTET, NONET, DECTET;
        public int numberOfMusicians() {
            return ordinal() + 1;
        }
    }
    //위에처럼 하지 말고 아래처럼
    public enum Ensemble {
        SOLO(1), DUET(2), 
        ... 
        DECTET(10). TPIPLE_QUARTET(12);
        private final int numberOfMusicians;
        Ensemble(int size) {this.numberOfMusicians = size;}
        public int numberOfMusicians() {return numberOfMusicians; }
    }
    ```
    * enum 상수에 연계되는 값을 ordinal을 사용해 표현하지 말라는 것이다. 그런 값이 필요하다면 그 대신 객체 필드(instance field)에 저장해야 한다.

* 규칙 32 비트 필드(bit field) 대신 EnumSet을 사용하라
    ```java
    //비트필드 열거형 상수 - 이제는 피하자
    public class Text {
        public static final int STYLE_BOLD = 1 << 0;//1
        public static final int STYLE_ITALIC = 1 << 1;//2
        public static final int STYLE_UNDERLINE = 1 << 2;//4
        //
        public void applyStyles(int styles){...}
    }
    ```
    * 집합을 비트 필드로 나타내면 비트 단위 산술 연산(bitwise arithmetic)을 통해 합집합이나 교집합 등의 집합 연산도 효율적으로 실행할 수 있다. 
        * OR 연산에 좋다
            * text.applyStyles(STYLE_BOLD | STYLE_ITALIC))
    ```java
    //Enumset 이용한 방법
    public class Text {
        public enum Style {BOLD, ITALIC, UNDRLINE}
        //
        public void applyStyles(Set<Style> styles){...}
    }
    ...
    text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
    ```
    * 열거 자료형을 집합에 사용해야 한다고 해서 비트 필드로 표현하면 곤란하다

* 규칙 33 ordinal을 배열 첨자로 사용하는 대신 EnumMap을 이용하라
    * ordinal 값을 배열 첨자로 사용하는 것은 적절치 않다. 대신 EnumMap을 써라.

* 규칙 34 확장 가능한 enum을 만들어야 한다면 인터페이스를 이용하라
    * 계승 가능 enum 자료형은 만들 수 없지만, 인터페이스를 만들고 그 인터페이스를 구현하는 기본 enum 자료형을 만들면 계승 가능 enum 자료형을 흉내 낼 수 있다. 

* 규칙 35 작명 패턴 대신 어노테이션을 사용하라
    * 어노테이션을 써라.

* 규칙 36 Override 어노테이션은 일관되게 사용하라
    * 상위 클래스에 선언된 메서드를 재정의할 때는 반드시 선언부에 Override 어노테이션을 붇어야 한다
    * 상위 자료형에 선언된 메서드를 재정의하는 모든 메서드에 Override 어노테이션을 붙이면 많은 오류를 미연에 방지할수 있따. 

* 규칙 37 자료형을 정의할 때 표식 인터페이스를 사용하라
    * ?


## 7장 메서드
* 규칙 38 인자의 유효성을 검사하라
    * ok
* 규칙 39 필요하다면 방어적 복사본을 만들라
    * ok
* 규칙 40 메서드 시그너처는 신중하게 설계하라
    * ok

* 규칙 41 오버로딩할 때는 주의하라
    * 오버로딩된 메서드 가운데 어떤 것이 호출될지는 컴파일 시점에 결정
    * 오버로딩된 메서드는 정적 (static)으로 선택되지만, 재정의된(overridden) 메서드는 동적(dynamic)으로 선택
        * 상위 클래스와 같은 시그니처를 갖는 하위 클래스
    * 혼란을 피하는 안전하고 보수적인 전략은, 같은 수의 인자를 갖는 두 개의 오버로딩 메서드를 API에 포함시키지 않는 것

* 규칙 42 varargs는 신중히 사용하라
    * 배열이 자동생성되는 구조.
    * 가변인자 사용예
    ```java
    static int sum(int... args){
        int sum = 0;
        for(int arg : args)
            sum += arg;
        return sum;
    }
    ```
    * 마지막 인자가 배열이라고 해서 무조건 뜯어고칠 생각은 버려라
    * 성능이 중요하면 varargs메서드 호출할때마다 배열이 만들어지고 초기화하기 때문에 조심해서 사용해야함.
        * 오버로딩으로 미리 인자가 적은 함수는 구현해두면 좋음.

* 규칙 43 null 대신 빈 배열이나 컬렉션을 반환하라
    * null 대신에 빈 배열이나 빈 컬렉션을 반환하라는 것이다.
        * C언어의 관습으로 c에서는 배열의 길이가 배열과 따로 반환, c에서는 길이 0인 배열을 할당해서 반환하더라도 아무 이득이 없다.

* 규칙 44 모든 API 요소에 문서화 주석을 달라
    * ok

## 8장 일반적인 프로그래밍 원칙들
* 규칙 45 지역 변수의 유효범위를 최소화하라
    * ok

* 규칙 46 for 문보다는 for-each 문을 사용하라
    * ok

* 규칙 47 어떤 라이브러리가 있는지 파악하고, 적절히 활용하라
    * 바퀴를 다시 발명하지 말라.

* 규칙 48 정확한 답이 필요하다면 float와 double은 피하라
    * 부동소수점 연산 정밀도 떨어짐.
    * 돈 계산을 할 때는 BigDecimal, int 또는 long을 사용한다는 원칙

* 규칙 49 객체화된 기본 자료형 대신 기본 자료형을 이용하라
    * ok

* 규칙 50 다른 자료형이 적절하다면 문자열 사용은 피하라
    * ok

* 규칙 51 문자열 연결 시 성능에 주의하라
    * 만족스런 성능을 얻으려면 String 대신 StringBuilder를 써
        * n개의 문자열에 연결 연산자를 반복적용하는것은 n제곱에 비례

* 규칙 52 객체를 참조할 때는 그 인터페이스를 사용하라
    * ?

* 규칙 53 리플렉션 대신 인터페이스를 이용하라

* 규칙 54 네이티브 메서드는 신중하게 사용하라

* 규칙 55 신중하게 최적화하라
    * 빠른 프로그램이 아닌 좋은 프로그램으로

* 규칙 56 일반적으로 통용되는 작명 관습을 따르라
    * ok

## 9장 예외 
* 9장부터 번역이 더 이상해졌는지, 내가 용어를 모르는 것인지..


* 규칙 57 예외는 예외적 상황에만 사용하라
    * 예외는 예외적인 상황에만 사용해야 한다. 평상시 제어 흐름(ordinary control flow)에 이용해서는 안 된다.

* 규칙 58 복구 가능 상태에는 점검지정 예외를 사용하고, 프로그래밍 오류에는 실행시점 예외를 이용하라
    * 자바는 세 가지 종류의 ‘throwable’을 제공한다. 점검지정 예외(checked exception), 실행시점 예외(runtime exception), 그리고 오류(error)다
    * ?

* 규칙 59 불필요한 점검지정 예외 사용은 피하라
    * ?

* 규칙 60 표준 예외를 사용하라
    * ?

* 규칙 61 추상화 수준에 맞는 예외를 던져라
    * 위 계층에서는 하위 계층에서 발생하는 예외를 반드시 받아서 상위 계층 추상화 수준에 맞는 예외로 바꿔서 던져야 한다.
        * 예외 변환 이라 부른다.(exception translation)
    ```java
    //예외변환
    try{
        //낮은 수준의 추상화 계층이용
        ...
    }catch(LowerLevelException e){
        throw new HigherLevelException(...);
    }
    ```

* 규칙 62 메서드에서 던져지는 모든 예외에 대해 문서를 남겨라
    * ok

* 규칙 63 어떤 오류인지를 드러내는 정보를 상세한 메시지에 담으라
    * ok

* 규칙 64 실패 원자성 달성을 위해 노력하라
    * ?

* 규칙 65 예외를 무시하지 마라
    * 빈 catch 블록은 예의를 선언한 목적, 그러니까 예외적 상황을 반드시 처리하도록 강제한다는 목적에 배치된다.
        ```java
        //catch 블록을 비워 놓으면 예외는 무시된다 - 이러지 말아라.
        try{
            ...
        }catch (SomeException e){
        }
        ```
    * 적어도 catch 블록 안에는 예의를 무시해도 괜찮은 이유라도 주석으로 남겨 두어야 한다.

## 10장 병행성
* 규칙 66 변경 가능 공유 데이터에 대한 접근은 동기화하라
    * synchronized 키워드는 특정 메서드나 코드 블록을 한 번에 한 스레드만 시용하도록 보장한다

* 규칙 67 과도한 동기화는 피하라
    * ok

* 규칙 68 스레드보다는 실행자와 태스크를 이용하라
    * 작은 프로그램이거나 부하가 크지 않은 서버를 만들 때는 보통 Executors.newCachedThreadPool이 좋다.
        * 작업은 큐에 들어가는 것이 아니라, 실행을 담당하는 스레드에 바로 넘겨진다. 가용한 스레드가 없는 경우 새 스레드가 만들어진다. 
    * 부하가 심한 환경에 들어갈 서버를 만들 때는 Executors.newFixedThreadPool을 이용해서 스레드 개수가 고정된 풀을 만들거나
    * 최대한 많은 부분을 직접 제어하기 위해 ThreadPoolExecutor 클래스를 사용하는 것이 좋다.

* 규칙 69 wait나 notify 대신 병행성 유틸리티를 이용하라
    * 가장 널리 쓰이는 동기자로는 CountDownLatch가 있다.
    * Collections.synchronizedMap이나 HashTable 대신 ConcurrentHashMap을 사용하도록 하자
    * 특정 구간의 실행시간을 쟬 때는 System.currentTimeMillis 대신 System.nanoTime을 사용해야 한다


* 규칙 70 스레드 안전성에 대해 문서로 남겨라
    * ok

* 규칙 71 초기화 지연은 신중하게 하라
    * 성능 문제 때문에 객체 필드 초기화를 지연시키고 싶다면 이중 검사(double-check) 숙어를 사용하라.
        * 한번은 락 없이 검사하고, 이후 락 건뒤 검사한다. volative로 선언해야 한다.
    ```java
    private volatile FieldType field;
    FieldType getField(){
        FieldType result = field;
        if(result == null){//락없이 첫번재 검사
            synchronized(this){
                result = field;
                if(result == null){//락 건뒤 두번째 검사.
                    field = result = computerFieldValule();
                }
            }
        }
        return result;
    }
    ```

* 규칙 72 스레드 스케줄러에 의존하지 마라
    * Thread.yield를 호출해서 문제를 해결하려고는 하지 마라.

* 규칙 73 스레드 그룹은 피하라
    * 스레드 그룹은 이제 폐기된 추상화 단위다.


## 11장 직렬화
* 직렬화
    * https://woowabros.github.io/experience/2017/10/17/java-serialize.html
    * https://woowabros.github.io/experience/2017/10/17/java-serialize2.html

* 규칙 74 Serializable 인터페이스를 구현할 때는 신중하라
    * Serializable 구현과 관련된 가장 큰 문제는 일단 클래스를 릴리스하고 나면 클래스 구현을 유연하게 바꾸기 어려워진다는 것이다
    * 두 번째 문제는, 버그나 보안 취약점(security hole)이 발생할 기능성이 높아진다는 것이다.
    * 세 번째 문제는, 새 버전 클래스를 내놓기 위한 데스트 부담이 늘어난다는 것이다.
    * 계승을 염두에 두고 설계하는 클래스는(규칙 17) Serializable을 구현하지 않는 것이 바람직하다. 또한 인터페이스는 가급적 Serializable을 계승하지 말아야 한다.
    * 내부 클래스(inner class)는(규칙 22) Serializable을 구현하면 안 된다

* 규칙 75 사용자 지정 직렬화 형식을 사용하면 좋을지 따져 보라
    * 어떤 직렬화 형식을 이용하건, 직렬화 가능 클래스를 구현할 때는 직렬 버전 UID(serial version UID)를 명시적으로 선언해야 한다.
    * ?

* 규칙 76 readObject 메서드는 방어적으로 구현하라
    * ?

* 규칙 77 개체 통제가 필요하다면 readResolve 대신 enum 자료형을 이용하라
    * ?

* 규칙 78 직렬화된 객체 대신 직렬화 프락시를 고려해 보라
    * ?




