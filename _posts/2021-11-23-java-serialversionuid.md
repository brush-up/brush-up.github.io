---
title:  "[java]Java serialVersionUID 간략 정리"
excerpt: "ava serialVersionUID 간략 정리를 해보자. "

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 2021-11-23
last_modified_at: 2021-11-23
---


## 들어가며
* 코드를 보면 아래와 같이 serialVersionUID 값이 지정되어있는 것을 볼수가 있다.
```java
private static final long serialVersionUID = -2010583614337318283L;
```
* 먼가 싶어 찾아봤다
    * 사실.. 아래 내용을 알고난뒤에도 꼭 필요한가 싶긴하다. jvm 위에서 돌아가는것이라 이런 스펙이 필요한가 싶음.

## serialVersionUID 사용처
* java 객체를 바이트 배열로 변환해서 파일이나 메모리등으로 Serialization나 Deserialization 할때 serialVersionUID 의 정보를 쓰고, 읽도록 해서 기존정보와 다르진 않는지 확인하는 용도로 쓰이는듯.
* serialVersionUID 지정을 하지 않으면 JVM마다 다르게 세팅이 될수도 있는듯.
    * Default serialVersionUID 생성로직
        * https://docs.oracle.com/javase/6/docs/platform/serialization/spec/class.html#4100
* 쓰고, 읽을때 serialVersionUID 정보가  다르다면 InvalidClassException 이 발생하게 된다고 함.

## 예시
* 아래 내용을 참고하자
    * https://mkyong.com/java-best-practices/understand-the-serialversionuid/
    * server/client 가 이기종간일 경우 다르게 읽힐수있고
    * serialVersionUID 가 바뀌는 경우에 대한 예시

## 자동생성 방법
* 이클립스에서 implements Serializable 선언하고 클래스명 위에 마우스 커서를 갖다대면 SerialversionUID 를 자동생성하는 메뉴가 뜰꺼야.
 * Intellij 에서는 Preference 메서 Inspection > Serialization issues 를 선택한 뒤 Serializable class without serialVersionUID 옵션을 선택한다,
 
## 조금더 상세히 알고 싶으면
* https://docs.oracle.com/javase/6/docs/platform/serialization/spec/version.html
* https://www.oracle.com/technical-resources/articles/java/serializationapi.html




