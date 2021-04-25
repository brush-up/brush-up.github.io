---
title:  "[BOOK]C++ API 디자인 - CHAPTER 03 패턴 "
excerpt: "C++ API 디자인 - CHAPTER 03 패턴 "

categories:
  - Book
tags:
  - [Book, Programming]

toc: true
toc_sticky: true
 
date: 2019-09-27
last_modified_at: 2019-09-27

published: true
---

* 3.1 Pimpl 이디엄
	* 이 용어는 pointer to implementation(구현포인터)라는 말의 약자
	* 이 기법의 주요 목적은 헤더 파일에 private으로 선언한 구체적인 코드를 외부로 노출하지 않는 것
	* (엄밀히 말하면 pimpl은 디자인 패턴이 아니다.)
3.1 그림 넣기
	* pimpl 이디엄은 숨겨진 클래스를 내부 포인터로 가르키는 공개 클래스를 말한다
* 3.1.1 Pimpl 사용

```cpp
// autotimer.h
#ifdef WIN32
#include <windows.h>
#else
#include <sys/time.h>
#endif
#include <string>
class AutoTimer
{
public:
/// Create a new timer object with a human readable name
explicit AutoTimer(const std::string &name);
/// On destruction, the timer reports how long it was alive
~AutoTimer();
private:
// Return how long the object has been alive
double GetElapsed() const;
std::string mName;
#ifdef WIN32
DWORD mStartTime;
#else
struct timeval mStartTime;
#endif
};
```

* 를 아래처럼 바꿀수 있다.

```cpp
// autotimer.h
#include <string>
class AutoTimer
{
public:
explicit AutoTimer(const std::string &name);
~AutoTimer();
private:
class Impl;
Impl *mImpl;
};
.
.
.
// autotimer.cpp
#include "autotimer.h"
#include <iostream>
#if WIN32
#include <windows.h>
#else
#include <sys/time.h>
#endif
class AutoTimer::Impl
{
public:
double GetElapsed() const
{
#ifdef WIN32
return (GetTickCount() mStartTime) / 1e3;
#else
struct timeval end time;
gettimeofday(&end time, NULL);
double t1 mStartTime.tv usec / 1e6 þ mStartTime.tv sec;
double t2 end time.tv usec / 1e6 þ end time.tv sec;
return t2 t1;
#endif
}
std::string mName;
#ifdef WIN32
DWORD mStartTime;
#else
struct timeval mStartTime;
#endif
};
AutoTimer::AutoTimer(const std::string &name) :
mImpl(new AutoTimer::Impl())
{
mImpl >mName name;
#ifdef WIN32
mImpl >mStartTime GetTickCount();
#else
gettimeofday(&mImpl >mStartTime, NULL);
#endif
}
AutoTimer::~AutoTimer()
{
std::cout << mImpl >mName << ": took " << mImpl >GetElapsed()
<< " secs" << std::endl;
delete mImpl;
mImpl NULL;
}

```

* 3.1.2 의미론적 복사
	* c++에선 명시적으로 복사 생성자를 구현하지 않는 경우 자동으로 만든다. 이건 얕은 복사만 하기에 여러가지 방법이 필요하다.
	* 1. 클래스를 복사할 수 없게 만든다. - 명시적으로 박수 생성자를 선언하고 연산자를 할당한다. private로 선언하는 것이 좋은 방법이다.(이러면  객체 복사시 컴파일 오류 발생)
	* 2. 복사 기능을 명시적으로 하자.
	* 다음은 예제

```cpp
#include <string>
class AutoTimer
{
public:
explicit AutoTimer(const std::string &name);
~AutoTimer();
private:
// Make this object be non copyable
AutoTimer(const AutoTimer &);
const AutoTimer &operator (const AutoTimer &);
class Impl;
Impl *mImpl;
};
```
* 3.1.3 Pimpl과 스마트 포인터
	* pimpl 이 불편하면서 쉽게 오류가 나는건 메모리 할당 해제 문제이다 
	* 다음 예제처럼 해자

```cpp
#include <boost/scoped ptr.hpp>
#include <string>
class AutoTimer
{
public:
explicit AutoTimer(const std::string &name);
~AutoTimer();
private:
class Impl;
boost::scoped ptr<Impl> mImpl;
};
```
* 3.1.4 Pimpl의 장점
* 3.1.5 Pimpl의 단점
* 3.1.6 C의 Opaque 포인터

* 3.2 싱글톤
* 3.2.1 C++의 싱글톤 구현
	* 클라이언트가 또 다른 새 인스턴스를 생성할 수 없도록 기본 생성자를 private로 선언하자
	* 2개의 인스턴스가 생성될수 없도록 싱글톤 클래스가 복사할수 없게 만든다(복사 생성자와 할당 연산자를 private로 선언하자)
	* 클라이언트가 싱글톤 클래스를 제거할 수 없도록 소멸자를 private로 선언하자
	* getInstance()메서드는 싱글톤 클래스의 포인터나 참조 둘중 하나를 리턴하는데, 포인터를 리턴하면 클라이언트는 잠재적으로 객체를 제거할수있기에 참조를 리턴하자.
	* 다음은 C++에서 싱글톤 일반적인 형태

```cpp
class Singleton
{
public:
static Singleton &GetInstance();
private:
Singleton();
~Singleton();
Singleton(const Singleton &);
const Singleton &operator (const Singleton &);
};
```

* 그런 다음 사용자는 다은과 같이 싱글톤 인스턴스에 참조를 요청한다

```cpp
Singleton &obj Singleton::GetInstance();
```
* 3.2.2 스레드에 안전한 싱글톤 만들기
	* 위의 싱글톤은 thread safe하지 않다. 다음처럼 해결하자

```cpp
Singleton &Singleton::GetInstance()
{
Mutex mutex;
ScopedLock(&mutex); // unlocks mutex on function exit
static Singleton instance;
return instance;
}
```

* 다만 위의 방법은 비용이 비싸기때문에 (호출될때마다 잠근다.) 아래처럼 하는 것도 좋은 방법이다.

```cpp
Singleton &Singleton::GetInstance()
{
static Singleton *instance NULL;
if (! instance) // check #1
{
Mutex mutex;
ScopedLock(&mutex);
if (! instance) // check #2
{
instance new Singleton();
}
}
return *instance;
}
```
* 3.2.3 싱글톤 vs 의존성 삽입
* 3.2.4 싱글톤 vs 모노스테이트
* 3.2.5 싱글톤 vs 세션 상태
	* 싱글톤 패턴을 대체하는 방법으로 의존성 삽입이나 모노스테이트 패턴, 세션 문맥 사용 등의 방법이 있다.
* 3.3 팩토리 메서드
	* 팩토리 메서드는 생성 디자인 패턴의 한 종류로 C++의 특정한 타입을 명시하지 않고 객체를 생성한다. 
* 3.3.1 추상 기본 클래스
	* 추상 기본 클래스는 하나 이상의 순수 가상 함수로 구성된다. (아래는 얘제)

```cpp
// renderer.h
#include <string>
class IRenderer
{
public:
virtual ~IRenderer() {}
virtual bool LoadScene(const std::string &filename) 0;
virtual void SetViewportSize(int w, int h) 0;
virtual void SetCameraPosition(double x, double y, double z) 0;
virtual void SetLookAt(double x, double y, double z) 0;
virtual void Render() 0;
};
```
* 3.3.2 단순한 팩토리 예제

```cpp
// rendererfactory.h
#include "renderer.h"
#include <string>
class RendererFactory
{
public:
IRenderer *CreateRenderer(const std::string &type);
};
```
	* 팩토리 메서드의 선언은 위와 같이 객체 인스턴스를 리턴하는 일반적인 메서드면 충분. 그러나 이건 IRenderer 타입의 특정 인스턴스를 리턴하지 못한다.

	* 내부는 이렇게..

```cpp
// rendererfactory.cpp
#include "rendererfactory.h"
#include "openglrenderer.h"
#include "directxrenderer.h"
#include "mesarenderer.h"
IRenderer *RendererFactory::CreateRenderer(const std::string &type)
{
if (type "opengl")
return new OpenGLRenderer();
if (type "directx")
return new DirectXRenderer();
if (type "mesa")
return new MesaRenderer();
return NULL;
}
```
* 3.3.3 확장 가능한 팩토리 예제

* 3.4 API 래핑 패턴
* 3.4.1 프록시 패턴
	* 프록시 디자인 패턴은 일대일의 인터페이스, 클래스 전달방식을 제공한다.
	* 이 패턴은 주로 기존 클래스의 복사본을 프록시 클래스에 저장하거나 포인터를 저장하는 방식으로 구현한다.
	* 프록시 패턴은 인터페이스의 형태를 변경하지 않고 원형 클래스의 행동을 변경할때 유용하다.
* 3.4.2 어댑터 패턴
* 3.4.3 퍼사드 패턴
* 3.5 옵저버 패턴
* 3.5.1 모델-뷰-컨트롤러
* 3.5.2 옵저버 패턴 구현
* 3.5.4 푸시 옵저버 VS 풀 옵저버