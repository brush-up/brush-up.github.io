---
title:  "[iOS]	ios용 universal binary 만들기"
excerpt: "	ios용 universal binary 만들어보자"

categories:
  - ios
tags:
  - [iOS, Programming]

toc: true
toc_sticky: true
 
date: 2011-12-13
last_modified_at: 2011-12-13
---
# 회고
* 10년이 지난 지금 이게 무슨 필요가 있냐 싶지만.... 일단 글을 옮기는  목적에만 충실히 하기 위해...

# 내용
* iPhone3는 arm6, iPhone3GS/iPhone4는 arm7 아키텍처

1. otool 로 확인하기
```
$ otool -hv libXXX.a
```
* Universal Binary가 아닌 경우

```
$ otool -hv libXXX.a
Archive : libXXX.a (architecture armv6)
.
.
$
```

* Universal Binary인 경우

```
$ otool -hv libXXX.a
Archive : libXXX.a (architecture armv6)
.
.
Archive : libXXX.a (architecture armv7)
.
.
Archive : libXXX.a (architecture i386)
...
$
```


2. 라이브러리를 XCode로 빌드한 경우
* Build Settings->Architectures->Architectures에 armv6, armv7, i386을 모두 명시하면 됩니다.
* Build Settings->Architectures->Valid Architectures에도 armv6, armv7, i386을 모두 명시합니다.
* 라이브러리를 Makefile로 빌드하여, architecture별로 library파일이 따로 존재하는 경우
* lipo 명령어를 사용하여 library를 Universal Binary 하나로 합쳐줍니다

```
$ lipo libXXX.armv6.a libXXX.armv7.a libXXX.i386.a -create -output libXXX.a
$
```


